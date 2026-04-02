---
layout: post
title: "FlashAttention from Scratch: Eliminating O(N²) HBM Traffic in Transformer Inference"
date: 2026-03-03
category: "CUDA · Attention Kernels · LLM Infrastructure"
summary: "From-scratch CUDA implementation of FlashAttention with online softmax and SRAM tiling, achieving 2×+ throughput over PyTorch naive attention on RTX 4090 at sequence length 4096."
description: "From-scratch CUDA C++ implementation of FlashAttention — online softmax + SRAM tiling eliminates O(N²) HBM traffic, achieving 2×+ throughput on RTX 4090 at sequence length 4096."
tags: [CUDA C++, FlashAttention, Online Softmax, SRAM Tiling, RTX 4090, LLM Inference, Nsight Compute, Transformers]
links:
  - label: GitHub
    url: https://github.com/LinLinHua
    external: true
---

This project implemented a fused FlashAttention kernel from scratch in CUDA C++. The goal was to reproduce the core contribution of Dao et al. [1] — eliminating the O(N²) HBM memory traffic that makes standard attention memory-bound — and benchmark the result against PyTorch's naive attention baseline on an RTX 4090 across sequence lengths up to 4096.

## Background: Why Attention is Memory-Bound

Standard scaled dot-product attention computes:

$$\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V$$

The naive implementation materializes the full $N \times N$ score matrix $S = QK^T / \sqrt{d_k}$ to HBM (high bandwidth memory), computes $P = \text{softmax}(S)$ by reading $S$ back from HBM and writing $P$ to HBM, then computes $O = PV$ by reading $P$ from HBM. Three full passes over an $N \times N$ matrix.

At sequence length $N = 4096$ with FP16, the attention score matrix alone is $4096 \times 4096 \times 2 = 32$ MB per head. A model with 32 heads and 32 layers generates 32 GB of HBM traffic *per forward pass* just for attention intermediates. Since the A100 and RTX 4090 both have peak HBM bandwidth of ~2 TB/s, and the arithmetic work scales as $O(N^2 d)$, standard attention is firmly memory-bound for long sequences [2].

**FlashAttention** [1], introduced by Dao et al. in 2022, avoids materializing the full attention matrix by computing attention in tiles that fit within SRAM. The key algorithmic ingredient is the **online softmax** algorithm of Milakov and Gimelshein [3], which allows softmax to be computed incrementally over a sequence of blocks without requiring the full row to be in memory simultaneously.

The online softmax maintains a running maximum $m$ and running sum $\ell$ for each query row. When processing a new block with local maximum $m_{new}$, the accumulated sum from previous blocks is rescaled:

$$\ell_{new} = e^{m_{old} - m_{new}} \cdot \ell_{old} + \sum_j e^{s_j - m_{new}}$$

This rescaling is numerically exact and requires only $O(1)$ extra state per query row, making a single-pass fused attention kernel possible [1].

**IO complexity.** Standard attention reads $O(N^2)$ bytes from HBM (the score matrix, twice). FlashAttention reads only $O(N \cdot d)$ bytes from HBM — the Q, K, V matrices — because attention scores are computed and immediately consumed within SRAM without being written back to HBM. For long sequences, this reduction in IO dominates the runtime [1].

## What We Did

We implemented a fused attention CUDA kernel combining online softmax with SRAM tiling, targeting the RTX 4090's SM architecture (Ada Lovelace, 128 KB shared memory per SM).

The kernel launches with one thread block per query tile. Each block loads its tile of $Q$ into shared memory, then iterates over all $K$ and $V$ tiles. For each $(Q\text{-tile}, K\text{-tile})$ pair, threads compute partial dot products entirely within registers, update the running max and sum accumulators using the online softmax recurrence, and apply the rescaling correction to the accumulated output $O$. After iterating over all $K/V$ tiles, the final output is written once to HBM. The full $N \times N$ attention score matrix is never materialized.

Tile size selection involved profiling across multiple configurations. Larger tiles improve arithmetic intensity (fewer HBM loads per FLOP) but increase shared memory pressure and risk register spilling to local memory. We used Nsight Compute to measure occupancy, shared memory utilization, and arithmetic intensity at each tile configuration, settling on a tile size that maximized FLOP throughput without triggering register spilling.

For **causal masking** (decoder-only models), we added per-tile masking in the inner loop: for tiles entirely above the diagonal (future tokens), the tile is skipped; for tiles on the diagonal, a per-thread conditional zeros out future-token contributions. This adds minimal overhead — a branch prediction that is consistently taken or not taken depending on tile position.

We benchmarked against PyTorch's `torch.nn.functional.scaled_dot_product_attention` baseline across sequence lengths 512, 1024, 2048, and 4096 on an RTX 4090. The throughput advantage exceeded 2× at sequence length 4096 and widened with sequence length, consistent with the O(N²) vs. O(N) HBM traffic scaling.

Nsight Compute confirmed the expected shift: compared to naive attention, HBM read bandwidth dropped substantially, SM utilization increased, and the kernel's roofline position moved from deep in the memory-bound region toward the compute-bound side — matching the theoretical prediction from IO complexity analysis.

## References

[1] T. Dao, D. Fu, S. Ermon, A. Rudra, and C. Ré, "FlashAttention: Fast and Memory-Efficient Exact Attention with IO-Awareness," in *NeurIPS 2022*. https://arxiv.org/abs/2205.14135

[2] S. Williams, A. Waterman, and D. Patterson, "Roofline: An Insightful Visual Performance Model for Multicore Architectures," *Communications of the ACM*, vol. 52, no. 4, pp. 65–76, 2009.

[3] M. Milakov and N. Gimelshein, "Online normalizer calculation for softmax," *arXiv:1805.02867*, 2018.

[4] NVIDIA Corporation, *CUDA C++ Programming Guide*, version 12.x, 2024. https://docs.nvidia.com/cuda/cuda-c-programming-guide/

[5] T. Dao, "FlashAttention-2: Faster Attention with Better Parallelism and Work Partitioning," in *ICLR 2024*. https://arxiv.org/abs/2307.08691
