---
layout: post # Do not change
category: [ml] # Project categories
title: "Intent Classification for ATIS Airline Queries" # Project title
date:   2025-11-13 10:00:00 -0800 # Updated date to reflect current progress
author: LilyHua # Your author nick

#
nextPart: _posts/2025-11-13-receipt-inbox.md # Links to your next project
prevPart: _posts/2021-09-01-gans-regression-paper.md # Links to your previous project
og_description: "A systematic approach to multi-class text classification using LSTMs and PyTorch, implementing Transfer Learning with GloVe embeddings on the ATIS dataset."
---

This project is a systematic investigation into multi-class text classification. The goal is to build a system that understands a user's natural language query and classifies it into one of 21 distinct intents for the **ATIS (Airline Travel Information System)** dataset.

This is a classic but challenging NLP problem, as the model must distinguish between semantically similar queries, such as "show me flights" (`flight` intent) versus "what time is my flight" (`flight_time` intent).

### My Role: PyTorch Pipeline and Core Model Implementation

As a co-author, I led the technical implementation of the deep learning pipeline. My contributions have evolved from the proposal stage to concrete execution:

1.  **PyTorch Data Pipeline:** I engineered the custom `ATISDataset` and `DataLoader` classes. This involved handling raw text tokenization, vocabulary building, and converting text to numeric tensors for model ingestion.
2.  **Core Model Architecture:** I implemented the primary **LSTM (Long Short-Term Memory)** architecture from scratch in PyTorch, designing a modular class that supports variable layers, dropout, and different embedding strategies.
3.  **Transfer Learning Integration:** I successfully integrated pre-trained **GloVe (Global Vectors for Word Representation)** embeddings (100d), allowing the model to leverage semantic knowledge from billions of web tokens rather than learning from scratch.

### The Experimental Progress

We adopted a hypothesis-driven approach, moving from simple baselines to complex deep learning architectures.

**1. Establishing the Baseline (Complete)**
We first implemented a **Logistic Regression** model using **TF-IDF** features.
* **Result:** As expected, the linear model underfitted the complex data, achieving a low Macro F1-Score (approx 0.30) and exhibiting a clear bias towards majority classes. This confirmed the need for a Deep Learning approach.

**2. Experiment 3a: Learned Embeddings (Complete)**
We trained an LSTM model initialized with random weights. This required the model to learn the entire vocabulary from scratch using only the limited ATIS training set (approx. 5,800 samples).

**3. Experiment 3b: Pre-trained GloVe Embeddings (Complete)**
This was the central experiment. We initialized the embedding layer with **GloVe 100d vectors**. This step allows the model to handle domain-specific vocabulary and synonyms (e.g., "plane" vs. "aircraft") much more effectively by utilizing transfer learning.

### Technical Challenges & "War Stories"

The implementation process wasn't without hurdles. I diagnosed and resolved several critical issues:

* **The "3.2 Loss" Mystery:** During the initial LSTM training, the loss stuck at ~3.20 and accuracy was near 0%. I diagnosed this as a data preprocessing issue where using "average length" for padding was corrupting valid inputs. I fixed this by implementing a robust padding strategy and switching to non-destructive list operations in Python, immediately restoring model convergence.
* **Dimension Mismatch:** Integrated 100-dimensional GloVe vectors into a model architecture originally designed for 128 dimensions, requiring careful refactoring of the model initialization logic.
* **Large File Handling:** The GloVe dataset exceeded GitHub's 100MB file limit. I refactored our Git workflow to manage large artifacts via `.gitignore` and history rewriting to ensure a clean repository state.

### Current Status: Optimization (Experiment 4)

With the single-layer LSTM + GloVe model successfully running, I am currently working on **Experiment 4: Systematic Optimization**. I am upgrading the architecture to a **Multi-layer LSTM (2 layers)** and tuning hyperparameters (Learning Rate and Dropout) to maximize the F1-Score and mitigate overfitting.

---

### Project Links
{% assign url3 = 'https://github.com/LinLinHua/Intent-Classification-for-ATIS-Airline-Queries.git' %}
<div class='sx-button'>
  <a href='{{ url3 }}' class='sx-button__content red' target='_blank' rel='noopener noreferrer'>
    <img src='/assets/img/icons/icons8-github-500.svg'/> View Project on GitHub
  </a>
</div>

<div class='sx-button'>
  <a href='/assets/pdf/EE541_Intent_Classification_Proposal.pdf' class='sx-button__content blue' target='_blank' rel='noopener noreferrer'>
    <img src='{{ site.baseurl }}/assets/img/icons/resume.svg' alt="Info Icon"/> Read the Full Project Proposal (PDF)
  </a>
</div>
{% assign url2 = 'https://ee541.usc-ece.com/project/topics/intent-classification.html' %}
<div class='sx-button'>
  <a href='{{ url2 }}' class='sx-button__content orange' target='_blank' rel='noopener noreferrer'>
    <img src='/assets/img/icons/info.svg'/> View Official Topic Description
  </a>
</div>
{% assign github_url1 = 'https://github.com/howl-anderson/ATIS_dataset' %}
<div class='sx-button'>
  <a href='{{ github_url1 }}' class='sx-button__content green' target='_blank' rel='noopener noreferrer'>
    <img src='/assets/img/icons/info.svg'/> View Data Source (ATIS)
  </a>
</div>