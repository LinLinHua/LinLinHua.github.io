---
layout: post # Do not change
category: [ml, nlp, python, pytorch] # Project categories
title: "Intent Classification for ATIS Airline Queries (EE 541)" # Project title
date:   2025-11-13 10:00:00 -0800 # Date from proposal timeline
author: LinLinHua # Your author nick

#
nextPart: _posts/2025-11-13-receipt-inbox.md # Links to your next project
prevPart: _posts/2021-09-01-gans-regression-paper.md # Links to your previous project
og_description: "A systematic approach to multi-class text classification using LSTMs and PyTorch, comparing learned vs. pre-trained GloVe embeddings on the ATIS dataset."
---

This project for my EE 541 Deep Learning course was a systematic investigation into multi-class text classification. The goal was to build a system that could understand a user's natural language query and classify it into one of 21 distinct intents for the **ATIS (Airline Travel Information System)** dataset [cite: 12-13].

This is a classic but challenging NLP problem, as the model must distinguish between semantically similar queries, such as "show me flights" (`flight` intent) versus "what time is my flight" (`flight_time` intent).

### My Role: PyTorch Pipeline and Core Model

[cite_start]As a co-author of the project, my primary responsibilities were focused on the core data and modeling pipeline [cite: 36-54]:
1.  **PyTorch Data Pipeline:** I implemented the complete `Dataset` and `DataLoader` classes, which handled tokenizing, numericalizing, and batching the raw text queries for the model.
2.  **Core Model Implementation:** I implemented and trained the primary **LSTM (Long Short-Term Memory)** model from scratch, including the embedding, LSTM, and final linear classification layers.
3.  **Final Analysis:** I was responsible for writing the analysis and conclusion for the final report and completing the Model Card.

### The Experimental Approach

The project's goal was not just to achieve high accuracy, but to follow a systematic, hypothesis-driven approach.

**1. The Baseline:** We first established a non-deep-learning baseline using **Logistic Regression** and **TF-IDF** features. This gave us a clear metric to beat.

**2. The Core LSTM Model:** We then implemented our core deep learning model in PyTorch, an RNN using an **LSTM** architecture, which is designed to "remember" sequential information in the text.

**3. The Central Experiment:** The core of our project was a direct comparison between two different embedding strategies:
* **(a) Learned Embeddings:** Training the embedding vectors from scratch, allowing them to learn the specific vocabulary of the ATIS dataset (e.g., "flight", "airfare", "BOS").
* **(b) Pre-trained Embeddings:** Initializing the embedding layer with **GloVe** vectors, leveraging a model pre-trained on billions of general-purpose web documents.

### Evaluation & Challenges

This problem was defined by two key challenges:
* **Class Imbalance:** Some intents (like `flight`) appear far more often than others. To account for this, we used the **Macro F1-Score** as our primary success metric, as it measures performance on rare classes fairly.
* **Semantic Ambiguity:** We relied on **Confusion Matrices** to perform a qualitative analysis and pinpoint exactly *where* our model was failing (e.g., was it confusing `flight` with `airfare`?).

This project was a deep dive into the entire experimental pipeline of an NLP project, from data ingestion to model implementation, comparison, and analysis.

---

### Project Links

<div class='sx-button'>
  <a href='https://github.com/kanglinde/ee541-final-proj.git' class='sx-button__content theme' target='_blank' rel='noopener noreferrer'>
    <img src='/assets/img/icons/c.svg'/> View Project on GitHub
  </a>
</div>

<div class='sx-button'>
  <a href='/assets/pdf/EE541_Intent_Classification_Proposal.pdf' class='sx-button__content blue' target='_blank' rel='noopener noreferrer'>
    <img src='/assets/img/icons/info.svg'/> Read the Full Project Proposal (PDF)
  </a>
</div>

<div class='sx-button'>
  <a href='https://ee541.usc-ece.com/project/topics/intent-classification.html' class='sx-button__content orange' target='_blank' rel='noopener noreferrer'>
    <img src='/assets/img/icons/info.svg'/> View Official Topic Description
  </a>
</div>

<div class='sx-button'>
  <a href='https://github.com/howl-anderson/ATIS_dataset' class='sx-button__content green' target='_blank' rel='noopener noreferrer'>
    <img src='/assets/img/icons/paint_brush.svg'/> View Data Source (ATIS)
  </a>
</div>