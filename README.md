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
- Final feature set reduced to 206 features

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

### 2. Correlation Filtering
Highly correlated features removed using Pearson correlation thresholding.

### 3. Final Feature Space
- 206 network-flow features
- Includes:
  - packet statistics
  - inter-arrival times
  - byte counts
  - flag features
  - flow duration metrics

---

## Train/Test Split

| Split | Samples |
|---|---|
| Training | 432,395 |
| Testing | 108,099 |

Training distribution:
- Benign: 279,342
- Attack: 153,053

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

---

## Results Summary

| Model | ACC (%) | FAR (%) | UND (%) | Time (s) |
|---|---|---|---|---|
| Logistic Regression | 90.45 | 4.67 | 18.46 | 6.32 |
| Random Forest | 96.61 | 2.83 | 4.41 | 24.72 |
| Naive Bayes | 41.30 | 90.34 | 0.94 | 2.03 |
| KNN | 96.37 | 2.54 | 5.61 | 2735.65 |
| Decision Tree | 96.39 | 3.88 | 3.12 | 19.24 |
| Label Propagation | 96.39 | 1.72 | 7.07 | 602.67 |
| GNN (GAT) | 93.19 | 1.13 | 17.17 | 1231.40 |

---

## Key Findings

### Random Forest
- Best overall accuracy
- Strong balance between FAR and UND
- Best traditional ML model

### Decision Tree
- Lowest UND among supervised ML models
- Best at minimizing missed attacks

### Label Propagation
- Strongest semi-supervised approach
- Used only 40% labeled data
- Excellent balance between FAR and UND

### GNN (Graph Attention Network)
- Lowest FAR overall
- Best false-alarm suppression
- Precision-focused deployment profile


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


