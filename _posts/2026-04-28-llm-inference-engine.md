---
layout: post
title: "LLM Inference Engine: FlashAttention-2, Paged KV Cache, and Tensor Core MMA"
date: 2026-04-28
category: "CUDA · LLM Inference · GPU Systems"
summary: "Building an LLM inference engine from scratch: a FlashAttention-2 prefill kernel with Tensor Core MMA and dynamic shared memory (numerically verified against PyTorch SDPA, max_diff < 0.002), and a vLLM-compatible paged KV cache with block_table indexing across all 16 transformer layers — integrated with a HuggingFace-patched ModelRunner achieving 51.84 tok/s on Llama-3.2-1B on A100."
description: "FlashAttention-2 CUDA kernel with wmma Tensor Core API, online softmax, GQA support, and dynamic shared memory. Paged KV cache with block_table indexing. HuggingFace LlamaAttention forward pass patch. All running on Llama-3.2-1B / Llama-3.1-8B on A100."
tags: [CUDA C++, Triton, FlashAttention, KV Cache, LLM Inference, Tensor Core, A100 GPU, HuggingFace, vLLM]
links:
  - label: GitHub
    url: https://github.com/LinLinHua/llm-inference-engine
    external: true
---

This project builds an LLM inference engine from scratch, targeting the decode path of transformer models on NVIDIA A100. The engine implements three components: a FlashAttention-2 prefill kernel in CUDA using the Tensor Core MMA API, a vLLM-compatible paged KV cache, and a HuggingFace attention patch that dispatches to custom kernels while maintaining compatibility with standard generation APIs. All components run on Llama-3.2-1B and Llama-3.1-8B.

The FA-2 kernel achieves **numerical correctness verified against PyTorch SDPA (max_diff < 0.002, fp16)**. The patched ModelRunner achieves **51.84 tok/s on Llama-3.2-1B on A100**, a 1.03× improvement over the unpatched HuggingFace baseline.

---

## Background: Why Standard Attention is Slow

Standard multi-head attention materializes the full N×N score matrix to HBM three times during each forward pass:

```
Step 1: S = Q @ Kᵀ / √d    → write N×N to HBM
Step 2: P = softmax(S)      → read + write N×N to HBM
Step 3: O = P @ V           → read N×N from HBM
```

For a sequence of N=2048 tokens with head_dim=128 in fp16, this is roughly 2048² × 2 × 3 ≈ 25 MB of intermediate HBM traffic **per head per layer**. At 32 heads and 32 layers, that totals ~25 GB per forward pass — at 2 TB/s HBM bandwidth, that alone takes ~12 ms before any compute happens.

The secondary problem is the KV cache. During autoregressive decode, each new token needs the K and V tensors of all previous tokens. Naive implementations pre-allocate a contiguous buffer of size `max_seq_len × num_layers × num_heads × head_dim` per request at initialization, regardless of how long the sequence actually turns out to be. At max_seq_len=4096, that is ~3.2 GB for 100 concurrent requests — with typical sequences being far shorter, 60–80% of that allocation is wasted.

---

## Part 1: FlashAttention-2 Prefill Kernel

### Algorithm: Online Softmax

FlashAttention [1] eliminates intermediate HBM traffic by fusing all three steps into a single kernel using **online softmax**. The key insight is that softmax can be computed in a single streaming pass without ever materializing the full row. For each KV tile, maintain two running statistics per query row:

```
m_new = max(m_old, tile_max)              ← running maximum
alpha = exp(m_old - m_new)                ← rescale factor for previous accumulator
l     = l * alpha + Σ exp(s_j - m_new)   ← running normalization sum
O     = O * alpha + P_tile @ V_tile       ← rescale and accumulate output
```

The correction factor `alpha` adjusts all previous contributions whenever the running max increases, so the final output is mathematically identical to standard softmax — no approximation.

FlashAttention-2 [2] improves on the original by changing the loop order: the outer loop iterates over **Q blocks**, the inner loop over **KV blocks**. This means each thread block owns one Q tile and holds its entire output accumulator `O_acc` in registers across the full KV sweep, writing to HBM exactly once at the end. In FA-1, multiple thread blocks updated the same output tile, requiring intermediate HBM writes.

```
FA-1 (outer=KV, inner=Q):                FA-2 (outer=Q, inner=KV):
  for each KV tile:                         for each Q tile (one thread block):
    for each Q tile:                          load Q tile → SRAM, stay there
      load Q, K, V → compute                  for each KV tile:
      write partial O → HBM ← slow              load K, V → SRAM
                                                compute S = Q @ Kᵀ (Tensor Core)
                                                online softmax update → registers
                                              write O → HBM once ← fast
```

### Tensor Core MMA

The Q×Kᵀ matrix multiply uses the CUDA `wmma` (Warp Matrix Multiply-Accumulate) API, which dispatches to Tensor Core instructions. Tensor Cores execute a 16×16×16 matrix multiply-accumulate in a single instruction at dramatically higher throughput than scalar FP32 FMAs. On A100, Tensor Core FP16 throughput is 312 TFLOPS versus 19.5 TFLOPS for scalar FP32.

```cuda
// Each warp computes a 16×16 tile of S = Q_smem @ K_smem^T
wmma::fragment<wmma::accumulator, 16, 16, 16, float> acc;
wmma::fill_fragment(acc, 0.f);

for (int dk = 0; dk < HD; dk += 16) {
    wmma::fragment<wmma::matrix_a, 16, 16, 16, __half, wmma::row_major> q_frag;
    wmma::fragment<wmma::matrix_b, 16, 16, 16, __half, wmma::col_major> k_frag;

    wmma::load_matrix_sync(q_frag, &Q_smem[warp_row * HD + dk], HD);
    wmma::load_matrix_sync(k_frag, &K_smem[nc * 16 * HD + dk], BC);
    wmma::mma_sync(acc, q_frag, k_frag, acc);
}
wmma::store_matrix_sync(&S_smem[warp_row * BC + nc * 16], acc, BC, wmma::mem_row_major);
```

The accumulator fragment type is `float` (fp32), which is the standard pattern: fp16 inputs feed the Tensor Core, fp32 accumulates the dot product to avoid overflow.

### Dynamic Shared Memory

The initial static shared memory allocation with BR=BC=64 exceeded A100's default 48KB limit per thread block:

```
Q_smem[64][128] fp16 = 16 KB
K_smem[64][128] fp16 = 16 KB
V_smem[64][128] fp16 = 16 KB
S_smem[64][64]  fp32 = 16 KB
total           = 64 KB  >  48 KB limit
```

The fix uses **dynamic shared memory**, which allows up to 164 KB on A100 via `cudaFuncSetAttribute`:

```cuda
// In the kernel: single extern __shared__ buffer, manually partitioned
extern __shared__ char smem_buf[];
__half* Q_smem = (__half*)smem_buf;
__half* K_smem = Q_smem + BR * HD;
__half* V_smem = K_smem + BC * HD;
float*  S_smem = (float*)(V_smem + BC * HD);

// In the wrapper: set attribute and pass smem_size to kernel launch
size_t smem_size = (BR + BC + BC) * HD * sizeof(__half) + BR * BC * sizeof(float);
cudaFuncSetAttribute(fa2_prefill_kernel,
    cudaFuncAttributeMaxDynamicSharedMemorySize, smem_size);
fa2_prefill_kernel<<<grid, block, smem_size>>>(...);
```

### GQA Support

Llama-3 uses Grouped Query Attention (GQA): 32 query heads share 8 KV heads. The kernel handles this via a simple index mapping:

```cuda
int kv_h = h / (Hq / Hkv);  // e.g., heads 0-3 all map to kv_head 0
```

No K/V expansion in HBM is needed — the kernel reads the correct KV head directly.

### Correctness Verification

```python
B, Hq, Hkv, Sq, D = 1, 32, 8, 64, 128

out_fa2 = m.fa2_prefill(Q, K, V, causal=True)

g = Hq // Hkv
out_ref = F.scaled_dot_product_attention(
    Q, K.repeat_interleave(g, dim=1), V.repeat_interleave(g, dim=1), is_causal=True
)

max_diff  = (out_fa2 - out_ref).abs().max().item()   # 0.00195
mean_diff = (out_fa2 - out_ref).abs().mean().item()  # 0.00003
```

| Metric | Value |
|---|---|
| max_diff (fp16) | 0.00195 |
| mean_diff (fp16) | 0.00003 |
| Correctness | **PASS** (threshold 0.05) |

The error is consistent with fp16 rounding — the kernel computes exactly the same attention as SDPA.

### Performance and Remaining Bottleneck

The P×V GEMM (softmax output multiplied by V) is currently implemented as a scalar loop rather than a second Tensor Core GEMM:

```cuda
// Scalar loop — not using Tensor Cores
for (int di = 0; di < 4; di++) {
    int d = lane_id + di * 32;
    float v_acc = 0.f;
    for (int jj = 0; jj < BC; jj++)
        v_acc += S_smem[(warp_row + r) * BC + jj] * __half2float(V_smem[jj * HD + d]);
    O_acc[r][di] = rescale * O_acc[r][di] + v_acc;
}
```

This loop runs BC=64 iterations per output element, with column-wise reads from `V_smem` that are not coalesced. Production kernels (e.g., Dao-AI FlashAttention [3]) fuse both GEMMs into a Tensor Core pipeline, achieving full SM utilization. Implementing the second Tensor Core GEMM requires restructuring the softmax output into a wmma fragment, which is left as a future extension.

---

## Part 2: Paged KV Cache

### The Memory Fragmentation Problem

Standard KV cache allocates a contiguous buffer for each request at startup:

```
Request A: [K₀|K₁|K₂|K₃| — | — | — | — ]  (8 slots, 3 used, 5 wasted)
Request B: [K₀|K₁|K₂|K₃|K₄|K₅| — | — ]  (8 slots, 6 used, 2 wasted)
Request C: [K₀|K₁| — | — | — | — | — | — ]  (8 slots, 2 used, 6 wasted)
```

Three failure modes follow directly from the contiguity requirement:

**Internal fragmentation** — because decode length is unknown at request start, the system must pre-allocate for the maximum possible sequence length. Typical utilization is 20–40% of allocated memory [4].

**Reservation waste** — even optimistic pre-allocation must hold reserved slots for future tokens during active generation, since the next token must land at a contiguous address.

**External fragmentation** — after short requests complete and free their memory, the gaps are too small for new long requests. A 512-token gap cannot hold a new 1024-token request even if total free memory is sufficient.

### Block Table Indexing

PagedAttention [4] applies the operating system's virtual memory paging concept to GPU KV cache. The physical memory is divided into fixed-size blocks. Each request maintains a **block table** — an indirection layer recording which physical blocks hold its KV data and in what order. New tokens consume one block at a time from a free list, with no pre-allocation and no contiguity requirement.

```
Physical HBM: [blk0][blk1][blk2][blk3][blk4][blk5]...

Request A: block_table = [blk2, blk5]     ← token 0-1 in blk2, token 2-3 in blk5
Request B: block_table = [blk0, blk3, blk4]
Request C: block_table = [blk1]
```

When Request A completes, `blk2` and `blk5` are returned to the free list in O(1) and immediately available for any new request, regardless of that request's sequence length.

### Implementation

The physical layout follows vLLM's convention [4]:

```python
# Layout: [num_blocks, num_kv_heads, head_dim, block_size]
# Transposed head_dim/block_size vs standard for coalesced access during gather
key_cache   = torch.zeros(num_blocks, num_kv_heads, head_dim, block_size)
value_cache = torch.zeros(num_blocks, num_kv_heads, head_dim, block_size)
```

The block allocator is a simple free list:

```python
def write_kv(self, request_id, layer_id, token_pos, k, v):
    self._ensure_blocks(request_id, token_pos + 1)
    block_idx = token_pos // self.block_size
    block_off = token_pos %  self.block_size
    phys      = self.block_tables[request_id][block_idx]
    self.key_cache[layer_id][phys, :, :, block_off] = k
    self.value_cache[layer_id][phys, :, :, block_off] = v

def _ensure_blocks(self, request_id, seq_len):
    needed  = (seq_len + self.block_size - 1) // self.block_size
    current = len(self.block_tables[request_id])
    for _ in range(needed - current):
        if not self.free_blocks:
            raise RuntimeError("KV cache full")
        self.block_tables[request_id].append(self.free_blocks.pop())
```

The cache covers all 16 transformer layers simultaneously, indexed by `layer_id`. Memory is allocated lazily — no blocks are assigned until the first `write_kv` call for a request.

### Memory Efficiency

| Config | Memory Allocated | Notes |
|---|---|---|
| Naive (max_seq_len=4096, 100 req) | ~3.2 GB | 60–80% wasted |
| Paged (512 blocks × block_size=16) | 0.25 GB | Near-zero fragmentation |

With 512 blocks of 16 tokens each, the cache holds 8192 tokens across all requests. Blocks are allocated per-token as generation proceeds, so a request that generates only 8 tokens consumes exactly ½ of one block rather than 4096 slots.

---

## Part 3: ModelRunner — Patching HuggingFace Attention

### The Integration Problem

HuggingFace Transformers v4.46+ changed the `LlamaAttention.forward` signature in a major refactor [5]: `rotary_emb` moved from the attention module to the model level, and the interface now passes pre-computed `position_embeddings = (cos, sin)` rather than `position_ids`. Patching the forward pass requires matching this new signature exactly.

### Official Forward Signature

Inspecting the installed version directly:

```python
def forward(
    self,
    hidden_states,
    position_embeddings,   # (cos, sin) — passed in from LlamaModel
    attention_mask=None,
    past_key_values=None,
    cache_position=None,
    **kwargs,
) -> tuple[torch.Tensor, torch.Tensor]:
```

Key changes vs older versions: `position_ids` is gone, `position_embeddings` is now a required positional argument, and `past_key_values` is a `DynamicCache` object (not a tuple).

### Patched Forward

The `_patched_forward` function mirrors the official signature and dispatches to the custom kernel:

```python
def _patched_forward(self, hidden_states, position_embeddings=None,
                     attention_mask=None, past_key_values=None,
                     cache_position=None, **kwargs):

    # QKV projection
    input_shape = hidden_states.shape[:-1]
    hidden_shape = (*input_shape, -1, self.head_dim)
    Q = self.q_proj(hidden_states).view(hidden_shape).transpose(1, 2)
    K = self.k_proj(hidden_states).view(hidden_shape).transpose(1, 2)
    V = self.v_proj(hidden_states).view(hidden_shape).transpose(1, 2)

    # RoPE — using HuggingFace's official apply_rotary_pos_emb
    cos, sin = position_embeddings
    Q, K = apply_rotary_pos_emb(Q, K, cos, sin)

    # KV cache update — DynamicCache API
    if past_key_values is not None:
        cache_kwargs = {"sin": sin, "cos": cos, "cache_position": cache_position}
        K, V = past_key_values.update(K, V, self._layer_id, cache_kwargs)

    # Write to PagedKVCache (alongside HF DynamicCache)
    paged_kv = getattr(self, "_paged_kv_cache", None)
    if paged_kv is not None:
        seq_pos = K.shape[2] - 1
        paged_kv.write_kv(request_id=self._request_id,
                          layer_id=self._layer_id, token_pos=seq_pos,
                          k=K[0, :, seq_pos, :], v=V[0, :, seq_pos, :])

    # Kernel dispatch
    is_decode = (Q.shape[2] == 1)
    if not is_decode and self._cuda_ext is not None:
        attn_output = self._cuda_ext.fa2_prefill(Q, K, V, True)
    else:
        g = self._Hq // self._Hkv
        attn_output = F.scaled_dot_product_attention(
            Q, K.repeat_interleave(g, dim=1), V.repeat_interleave(g, dim=1),
            is_causal=(Q.shape[2] > 1))

    attn_output = attn_output.transpose(1, 2).reshape(*input_shape, -1).contiguous()
    return self.o_proj(attn_output), None
```

### CUDA Extension Build

Per official PyTorch documentation [6], `torch/extension.h` must not be parsed by `nvcc`. The binding is separated into two files:

- `flash_attn_prefill.cu` — kernel + C++ wrapper, `#include <ATen/ATen.h>`, uses `at::Tensor`
- `flash_attn_prefill_bind.cpp` — pybind11 module, `#include <torch/extension.h>`, forward declaration only

```python
cpp_ext.load(
    name="flash_attn_prefill",
    sources=["flash_attn_prefill_bind.cpp", "flash_attn_prefill.cu"],
    extra_cuda_cflags=["-O2", "--use_fast_math", "-arch=sm_80"],
)
```

### End-to-End Benchmark

| Config | tok/s | Speedup | Notes |
|---|---|---|---|
| Baseline (HF SDPA) | 50.22 | 1.00× | Unpatched HuggingFace, A100 |
| Patched SDPA | 51.84 | 1.03× | Our forward, SDPA fallback |
| + FA-2 CUDA kernel | pending | — | Pending P×V Tensor Core fusion |

The 1.03× improvement from the patched forward reflects reduced Python overhead in the dispatch path. The FA-2 kernel correctness is verified but its performance is bottlenecked by the scalar P×V loop; the full speedup requires the second Tensor Core GEMM.

---

## What's Next

**P×V Tensor Core GEMM.** The scalar softmax-output × V loop is the primary performance gap. The fix is to store the softmax probabilities as a wmma fragment after the online softmax update, then call `wmma::mma_sync` for the P×V product. This requires restructuring the softmax normalization to operate on fragment elements rather than shared memory scalars.

**Triton FlashDecoding kernel.** Autoregressive decode generates one token at a time — the query is a single vector, making the prefill kernel's tiling unnecessary. FlashDecoding [7] splits the KV sequence across multiple thread blocks and uses a parallel reduction to combine partial outputs, recovering SM occupancy for the single-token decode case.

**Speculative decoding.** A draft model (Llama-3.2-1B) generates candidate token sequences for verification by the target model (Llama-3.1-8B). Accepted tokens provide superlinear throughput gains when the acceptance rate is high on repetitive or predictable text.

---

## References

[1] T. Dao, D. Fu, S. Ermon, A. Rudra, and C. Ré, "FlashAttention: Fast and Memory-Efficient Exact Attention with IO-Awareness," *NeurIPS 2022*. https://arxiv.org/abs/2205.14135

[2] T. Dao, "FlashAttention-2: Faster Attention with Better Parallelism and Work Partitioning," *ICLR 2024*. https://arxiv.org/abs/2307.08691

[3] Dao-AI Lab, *flash-attention* (GitHub repository). https://github.com/Dao-AILab/flash-attention

[4] W. Kwon et al., "Efficient Memory Management for Large Language Model Serving with PagedAttention," *SOSP 2023*. https://arxiv.org/abs/2309.06180

[5] D. Sultania, "Understanding KV Cache and Paged Attention in LLMs," *Medium*, Oct. 2024. https://medium.com/my-musings-with-llms/understanding-kv-cache-and-paged-attention-in-llms-a-deep-dive-into-efficient-inference-62fa372432ce

[6] HuggingFace, *transformers* (GitHub), PR #35235 "All attention refactor." https://github.com/huggingface/transformers/pull/35235

[7] PyTorch, *torch.utils.cpp_extension* documentation. https://pytorch.org/docs/stable/cpp_extension.html

[8] T. Dao et al., "FlashDecoding++," 2023. https://arxiv.org/abs/2311.01282

[9] NVIDIA Corporation, *CUDA C++ Programming Guide*, version 12.x, 2024. https://docs.nvidia.com/cuda/cuda-c-programming-guide/

[10] PyTorch, *extension-cpp* (GitHub repository). https://github.com/pytorch/extension-cpp
