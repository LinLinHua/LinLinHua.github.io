---
layout: post # Do not change
category: [ml] # Project categories
title: "Adaptive Weight Packing Profiler for Edge LLMs" # Project title
date:   2025-12-02 21:30:00 -0800 # Updated date
author: LilyHua # Your author nick
prevPart: _posts/2025-11-14-personal-portfolio-ci-cd.md  
# nextPart: (This is your latest project)
og_description: "Replicating MLSys '25 research to optimize Edge LLM inference memory bandwidth by ~40% using adaptive weight packing techniques."
---

Deploying Large Language Models (LLMs) like Llama-3 or OPT on edge devices (mobile phones, IoT) faces a critical bottleneck: the **"Memory Wall."** While compute power has grown, memory bandwidth hasn't kept pace.

This project is a Python-based profiling tool designed to address this bottleneck. Based on the **MLSys '25 paper "MEADOW,"** I engineered a solution that compresses model weights to minimize data movement, achieving a theoretical bandwidth reduction of **~40%**.

### 1. The Challenge: Bandwidth-Bound Inference

On edge devices with strict power constraints (<10W), fetching large weight matrices from off-chip DRAM consumes significant time and energy. Standard execution (GEMM) is inefficient because it repeatedly fetches redundant data.

**The Goal:**
To characterize the redundancy in LLM weights and implement a compression strategy that reduces off-chip memory access without needing to retrain the model.

### 2. Core Implementation: Dictionary-Based Packing

I engineered a custom compression algorithm using **PyTorch** and **Hugging Face Transformers** that treats weight matrices like a dictionary.

**How it works:**
* **Chunking:** The $$N \times M$$ weight matrix is sliced into small blocks (e.g., $$4 \times 1$$).
* **Unique Extraction:** The algorithm identifies unique patterns across these blocks.
* **Packing:** Instead of storing raw values, we store a small "Unique Matrix" (Dictionary) and a lightweight "Index Map."

By running this on the **OPT-125M** model with simulated **INT8 quantization**, I confirmed that Attention layers exhibit massive redundancy, making them perfect candidates for this optimization.

### 3. Engineering Extension: Sensitivity Analysis

Going beyond simple replication, I conducted a **sensitivity analysis** to answer an engineering trade-off question: *"What is the optimal chunk size?"*

I extended the profiler to evaluate different configurations ($$C=2, 4, 8$$):
* **Small Chunks ($$C=2$$):** Higher compression ratio (easier to find matches), but heavier index metadata overhead.
* **Large Chunks ($$C=8$$):** Lower metadata overhead, but significantly lower match rates.

**Result:** The analysis identified a "sweet spot" at $$C=4$$ for Attention layers, maximizing bandwidth savings.



### 4. Impact & Results

This tool provides a concrete architectural insight for designing next-generation edge accelerators.

* **~40% Bandwidth Reduction:** Achieved on Attention layers compared to standard INT8 quantization.
* **Hardware-Aware:** The profiling results help determine the optimal on-chip buffer sizes needed for future hardware designs.

### Technologies Used
* **Python & PyTorch** (Core Logic)
* **Hugging Face Transformers** (Model Loading)
* **NumPy** (Vectorized Operations)
* **Matplotlib** (Data Visualization)
* **System Modeling** (Bandwidth Estimation)

---

### Project Links
{% assign github_url = 'https://github.com/LinLinHua/Meadow-Weight-Packing-Profiler' %}
<div class='sx-button'>
  <a href='{{ github_url }}' class='sx-button__content red' target='_blank' rel='noopener noreferrer'>
    <img src='/assets/img/icons/icons8-github-500.svg'/> View Source Code
  </a>
</div>