# Network Intrusion Detection on Cloud Traffic
## Detecting DDoS Attacks with Machine Learning and Graph-Based Methods

This project presents a comparative evaluation of traditional Machine Learning and Graph-Based approaches for detecting Distributed Denial of Service (DDoS) attacks in cloud-network environments using the **BCCC-cPacket-Cloud-DDoS-2024** dataset.

The project was developed as part of the **Network Science** subject project.

---

## Overview

Modern cloud infrastructures are increasingly vulnerable to DDoS attacks that overwhelm services with massive amounts of malicious traffic. Traditional rule-based Intrusion Detection Systems (IDS) struggle to adapt to evolving attack patterns, making Machine Learning (ML) and Graph Neural Networks (GNNs) promising alternatives.

This project compares:

### Traditional ML Models
- Logistic Regression
- Random Forest
- Naive Bayes
- K-Nearest Neighbors (KNN)
- Decision Tree

### Graph-Based Models
- Label Propagation (Semi-Supervised Learning)
- Graph Attention Network (GAT / GNN)

The goal is to evaluate how different algorithms behave in terms of:

- Accuracy (ACC)
- False Alarm Rate (FAR)
- Un-Detection Rate (UND)
- Computational performance

---

## Dataset

Dataset used:

**BCCC-cPacket-Cloud-DDoS-2024**

Source:  
https://www.yorku.ca/research/bccc/ucs-technical/cybersecurity-datasets-cds/cloud-ddos-attacks-bccc-cpacket-cloud-ddos-2024/

### Dataset Characteristics

- 540,494 network flow samples
- 319 original features
- Final feature space: **48 PCA components** retaining 95.05% of variance
  (variance filter first reduces 315 → 312, PCA then reduces 312 → 48)

Labels:
- Benign
- Attack
- Suspicious

For binary classification:
- Attack + Suspicious → merged into `Attack`

---

## Feature Engineering

The preprocessing pipeline includes:

### 1. Variance Filtering
Removal of near-zero variance features.

### 2. PCA (Principal Component Analysis)
PCA replaces the previous Pearson correlation filter. PCA produces a compact
orthogonal basis that captures the directions of greatest variance across the
full feature set simultaneously, rather than only removing pairs that are
linearly correlated. The number of components is chosen to retain 95% of the
explained variance.

### 3. Final Feature Space
- **48 principal components** retaining 95.05% of variance
- Derived from the variance-filtered NTLFlowLyzer features:
  - packet statistics
  - inter-arrival times
  - byte counts
  - flag features
  - flow duration metrics

---

## Train / Validation / Test Split

The data is split into three disjoint sets so that hyperparameter selection
and threshold tuning use a held-out validation set, leaving the test set
completely untouched until final evaluation.

| Split | Samples | Fraction of total | Purpose |
|---|---|---|---|
| Training | 345,916 | 64% | Model fitting |
| Validation | 86,479 | 16% | Hyperparameter / threshold tuning, early stopping |
| Test | 108,099 | 20% | Final reported metrics |

The split is performed in two stages:
1. **80 / 20** stratified split into `train_full` and `test`.
2. **80 / 20** stratified split of `train_full` into `train` and `validation`.

All splits are stratified on the binary `Benign / Attack` label.

Training-set class distribution: **223,474 benign / 122,442 attack** (imbalance ratio 1.83×).

---

## Evaluation Metrics

This project focuses on security-oriented IDS metrics rather than accuracy alone.

### Accuracy (ACC)
Overall correctly classified flows.

### False Alarm Rate (FAR)
Percentage of benign traffic incorrectly classified as attacks.

High FAR leads to:
- alert fatigue
- wasted SOC resources

### Un-Detection Rate (UND)
Percentage of attacks missed by the IDS.

UND is critical because:
- undetected attacks can cause service outages

### F1 Score
Harmonic mean of precision and recall on the `Attack` class. F1 is reported
for every model alongside accuracy / FAR / UND because in an imbalanced
binary classification setting it summarises detection quality more
faithfully than accuracy alone.

---

## Results Summary

Test-set metrics from the re-executed notebook (PCA features, 64/16/20 split):

| Model | ACC (%) | FAR (%) | UND (%) | F1 | Precision | Recall | Time (s) |
|---|---|---|---|---|---|---|---|
| Logistic Regression | 90.20 | 4.76 | 18.98 | 0.8541 | 0.9031 | 0.8102 | 1.25 |
| Random Forest | 96.25 | 3.22 | 4.71 | 0.9474 | 0.9419 | 0.9529 | 29.78 |
| Naive Bayes | 40.53 | 90.44 | 2.93 | 0.5361 | 0.3703 | 0.9707 | 0.49 |
| KNN | **96.33** | 2.61 | 5.61 | **0.9479** | 0.9519 | 0.9439 | 105.55 |
| Decision Tree | 95.90 | 4.47 | **3.43** | 0.9434 | 0.9222 | **0.9657** | 23.33 |
| Label Propagation | 96.29 | **1.80** | 7.19 | 0.9466 | **0.9658** | 0.9281 | 119.65 |
| GNN (GAT) | 93.98 | 2.61 | 12.24 | 0.9117 | 0.9486 | 0.8776 | 849.00 |

**Best Accuracy:** KNN · **Lowest FAR:** Label Propagation · **Lowest UND (excluding Naive Bayes artifact):** Decision Tree · **Best F1:** KNN

Note: Naive Bayes's apparent low UND (2.93%) is an artifact of it predicting "Attack" almost universally — its FAR of 90.44% makes it unusable.

---

## Key Findings

### KNN
- Highest overall accuracy (96.33%) and best F1 score (0.9479)
- Competitive FAR (2.61%) but the slowest non-graph classifier (~106 s)

### Random Forest
- Second-best F1 (0.9474), strongest balance of FAR (3.22%) and UND (4.71%)
- Trains in ~30 s — best speed/accuracy tradeoff among traditional ML

### Decision Tree
- Lowest UND of all supervised models at 3.43%
- Highest recall (0.9657) — best at not missing attacks

### Label Propagation
- **Lowest FAR overall (1.80%)** and highest precision (0.9658)
- Used only 40% labeled data
- Threshold tuned on the held-out validation set (0.577, max-F1)

### GNN (Graph Attention Network)
- Solid precision (0.9486) but higher UND (12.24%) than the other graph method
- Trained the longest (~14 min) — the new precision/recall balance shifts compared to the previous pipeline because the PCA feature space changes neighbour structure


---

## Technologies Used

### Programming
- Python

### ML / Data Science
- Scikit-learn
- Pandas
- NumPy

### Graph Learning
- PyTorch
- PyTorch Geometric
- NetworkX

### Visualization
- Matplotlib
- Seaborn

---

## Graph-Based Learning

One of the key focuses of this project is the application of graph-based methods to intrusion detection.

### Why Graphs?

Network traffic naturally forms relational structures:
- similar attack flows cluster together
- neighboring flows often share behavioral patterns

Graph-based learning exploits these relationships instead of treating every flow independently.

### Graph Attention Network (GAT)

The GNN implementation:
- constructs a k-NN graph
- aggregates neighbor information
- uses attention mechanisms to weight important neighbors


## Running the Project

Clone repository:

```bash
git clone https://github.com/cokovskak/ids-cloud-network-science.git
```


Run notebook:

```bash
jupyter notebook notebook.ipynb
```

---


