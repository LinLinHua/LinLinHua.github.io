---
layout: post
title: "Intent Classification for ATIS Airline Queries"
date: 2025-11-12
category: "NLP · Deep Learning · PyTorch"
summary: "Building a 21-class intent classifier on the ATIS dataset — systematic progression from logistic regression baseline through LSTM with GloVe transfer learning, with PyTorch pipeline implementation."
description: "Building a 21-class intent classifier on the ATIS dataset via PyTorch LSTM with GloVe transfer learning. Systematic hypothesis-driven experimental progression."
tags: [Python, PyTorch, GloVe 100d, LSTM, TF-IDF, Transfer Learning, NLP, ATIS]
links:
  - label: GitHub
    url: https://github.com/kanglinde/ee541-final-proj
    external: true
  - label: Project Proposal (PDF)
    url: /assets/pdf/EE541_Proposal.pdf
    external: false
---

This project was completed for EE 541 (A Computational Introduction to Deep Learning) at USC, in collaboration with Harry Kang. The task was multi-class intent classification on the ATIS (Airline Travel Information System) benchmark dataset. My primary contributions were the PyTorch data pipeline, the LSTM model implementation, and the GloVe transfer learning integration.

## Background: Intent Classification and the ATIS Benchmark

Intent classification is the task of mapping a natural language utterance to one of a set of predefined intent categories — a core component of task-oriented dialogue systems and virtual assistants. The challenge is not just vocabulary coverage; it is disambiguating semantically similar queries. "Show me flights from Boston" and "What time does my flight leave" share nearly every content word, but map to distinct intents (`flight` vs. `flight_time`) that require completely different system responses.

The ATIS dataset, originally collected by DARPA for speech understanding research, consists of approximately 5,800 training utterances and 900 test utterances distributed across 21 airline travel intent classes [1]. It has been a standard benchmark for NLP intent classification since the early work of Tur et al. [2] and remains widely used for evaluating sequence models.

A known challenge in ATIS is class imbalance: the `flight` intent accounts for over 70% of training examples. This makes accuracy a misleading evaluation metric — a model predicting `flight` for everything would score 70% accuracy while being completely useless. **Macro F1-Score**, which averages performance across all 21 classes equally, is the appropriate metric [3].

The evolution of methods on ATIS follows the broader NLP trajectory: early approaches used hand-crafted features with SVMs or logistic regression [2]; deep learning methods introduced RNNs and LSTMs that capture sequential structure [4]; state-of-the-art results now come from fine-tuned transformers such as BERT [5]. Our project systematically reproduced this progression, from TF-IDF baseline through LSTM with transfer learning.

GloVe (Global Vectors for Word Representation) [6] pre-trains word vectors on large text corpora by factorizing a word co-occurrence matrix. The resulting 100-dimensional vectors encode semantic relationships — words like "aircraft," "plane," and "jet" are nearby neighbors in the embedding space — giving a model pre-trained semantic knowledge it would otherwise need to learn from scratch on the small ATIS training set.

## What We Did

**Experiment 1 — Logistic Regression baseline.** We implemented a Logistic Regression classifier with TF-IDF features using scikit-learn. This intentional baseline was meant to fail: TF-IDF treats each document as a bag of words with no sequential structure, which is the wrong inductive bias for a task where word order and context determine intent. As expected, the model achieved Macro F1 ≈ 0.30 and exhibited strong bias toward majority classes. This confirmed the need for a sequence-aware deep learning approach.

**Data pipeline.** I built the full PyTorch data pipeline from scratch: a vocabulary builder that tokenizes the training set and assigns integer IDs, a custom `ATISDataset` class implementing `__getitem__` and `__len__`, and a `DataLoader` with a custom collate function handling variable-length sequences through padding. Sequences are padded to the maximum length in each batch, not to a global average — a distinction that turned out to matter significantly.

**Experiment 3a — LSTM with learned embeddings.** I implemented the LSTM classifier in PyTorch: an `nn.Embedding` layer initialized randomly, one or more `nn.LSTM` layers, and a final `nn.Linear` classifier operating on the hidden state at the final timestep. Training from scratch on ~5,800 examples produced modest results, as expected — the model has limited data to learn both word semantics and intent-discriminative patterns simultaneously.

**Debugging — the "3.2 Loss" stall.** During initial training, loss stuck at approximately 3.20 with near-zero accuracy across all epochs. log(21) ≈ 3.04, which is the expected cross-entropy for a randomly initialized model predicting uniform probabilities. A loss of 3.20 with no decrease meant the model was not learning at all despite gradients flowing. The root cause was a padding bug: the collate function was computing the average sequence length and then *destructively* truncating all sequences to that length, silently corrupting sequences longer than average. Replacing this with non-destructive padding to the maximum batch length restored convergence immediately.

**Experiment 3b — GloVe 100d transfer learning.** I loaded the GloVe 100-dimensional pre-trained vectors, built a mapping from vocabulary tokens to their GloVe representations, and initialized the `nn.Embedding` weight matrix with the pre-built vectors rather than random initialization. Words absent from GloVe's vocabulary (airline codes, abbreviations) fell back to random initialization and were learned from data. The improvement on semantically similar intent pairs was measurable — the model could leverage GloVe's semantic structure to distinguish "show me flights" from "what time is my flight" where the randomly initialized model conflated them.

**Experiment 4 — Multi-layer LSTM with hyperparameter optimization.** I extended the architecture to two stacked LSTM layers and conducted systematic search over learning rate and dropout. Stacking layers creates a hierarchical representation: the first layer models local sequential dependencies, the second layer models higher-level compositional structure. Dropout between layers consistently improved validation Macro F1 compared to architectures without it, as expected given the small training set size.

One operational challenge: the GloVe 6B.100d file is approximately 330 MB, exceeding GitHub's 100MB file limit. I managed this by excluding the file from the repository via `.gitignore` and documenting the download procedure in the README, with Git history rewriting to remove any accidentally committed large files.

## Results

The table below summarises the four experiments. Accuracy is a misleading metric here given class imbalance — Macro F1 is the number that matters.

<figure style="margin: 2rem 0;">
  <img src="/assets/img/posts/intent_perf_table.png" alt="Performance summary table across all four experiments" style="width:100%; max-width:400px; display:block;">
  <figcaption style="font-size:13px; color:#8a8a84; margin-top:0.6rem; line-height:1.6;">Table 1: Performance summary. Exp. 1 (Logistic Regression + TF-IDF) establishes the baseline. Multi-layer LSTM with GloVe initialisation and further hyperparameter optimisation achieves the best result across both metrics.</figcaption>
</figure>

The key takeaways: GloVe initialisation (Exp. 2a → 2b) adds ~8 points of Macro F1 with no architectural change — evidence that pre-trained semantics directly benefit the small-data regime. The jump from Exp. 1 to Exp. 2a is larger still (~26 points), confirming that sequential structure matters far more than richer features for this task.

The confusion matrix for the best model (Exp. 3, Multi-layer LSTM + GloVe Optimised) breaks down where errors concentrate:

<figure style="margin: 2rem 0;">
  <img src="/assets/img/posts/intent_confusion_matrix.png" alt="Confusion matrix for the best-performing LSTM+GloVe optimised model" style="width:100%; max-width:400px; display:block;">
  <figcaption style="font-size:13px; color:#8a8a84; margin-top:0.6rem; line-height:1.6;">Figure 5: Confusion matrix, LSTM + GloVe Optimised (Exp. 3). The strong diagonal confirms high per-class precision on dominant classes. Most off-diagonal mass sits in composite intent rows (e.g. `flight+airfare`, `meal`) — compound intents that share surface-level vocabulary with their constituent single-intent classes.</figcaption>
</figure>

The dominant `flight` class (626 test examples, top-left) is handled near-perfectly. Errors concentrate in low-frequency composite intents such as `flight+airfare` and `meal`, where limited training examples make it hard to distinguish combined from individual intents. This is consistent with the known difficulty of ATIS at the long tail [3].

## References

[1] C. T. Hemphill, J. J. Godfrey, and G. R. Doddington, "The ATIS spoken language systems pilot corpus," in *Proceedings of the DARPA Speech and Natural Language Workshop*, 1990.

[2] G. Tur, D. Hakkani-Tür, and L. Heck, "Zero-shot learning for semantic utterance classification," *arXiv:1401.0509*, 2014.

[3] C. D. Manning, P. Raghavan, and H. Schütze, *Introduction to Information Retrieval*. Cambridge University Press, 2008.

[4] D. Hakkani-Tür et al., "Multi-domain joint semantic frame parsing using bi-directional RNN-LSTM," in *SLT 2016*.

[5] Q. Chen, Z. Zhu, and Z. Ling, "BERT-based approach for intent classification on the ATIS dataset," in *Proceedings of the 4th Workshop on Representation Learning for NLP*, 2019.

[6] J. Pennington, R. Socher, and C. D. Manning, "GloVe: Global Vectors for Word Representation," in *EMNLP 2014*.
