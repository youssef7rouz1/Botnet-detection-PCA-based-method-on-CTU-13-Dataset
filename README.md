# Botnet Detection Using PCA-Based Methods on the CTU-13 Dataset

This repository implements a PCA-based method for botnet detection using two complementary approaches:
- **Lakhina Entropy-Based PCA** – a simplified, pedagogical baseline to understand PCA for anomaly detection.
- **FLAGS Detection Approach** – our final target that uses TCP flag histograms combined with PCA for robust botnet detection.

---

## Table of Contents

- [Introduction and Motivation](#introduction-and-motivation)
- [Dataset Description and Statistical Analysis](#dataset-description-and-statistical-analysis)
- [Lakhina Entropy-Based PCA](#lakhina-entropy-based-pca)
- [FLAGS Detection Approach](#flags-detection-approach)
- [Implementation Details](#implementation-details)
  - [Preprocessing and Feature Extraction](#preprocessing-and-feature-extraction)
  - [PCA and Anomaly Scoring](#pca-and-anomaly-scoring)
  - [Hybrid Model (PCA + Logistic Regression)](#hybrid-model-pca--logistic-regression)
- [Results and Evaluation](#results-and-evaluation)
- [Usage](#usage)
- [References](#references)

---

## Introduction and Motivation

In today’s cybersecurity landscape, the detection of network anomalies and botnet activities is crucial. This project addresses the problem of intrusion detection by:
- Implementing and evaluating anomaly detectors based on network traffic analysis.
- Detecting botnet activity using the CTU-13 dataset, which contains realistic network attack scenarios.

Our approach is twofold:
1. **Lakhina Entropy-Based PCA:** We use this as a simplified baseline to understand how PCA can be applied to network anomaly detection. It serves as the foundation for our final FLAGS-based method.
2. **FLAGS Detection:** This method leverages TCP flag histograms as features for PCA, aiming to overcome the limitations of volume-based detectors by focusing on behavioral patterns in TCP flags.

---

## Dataset Description and Statistical Analysis

The **CTU-13 dataset** is a collection of 13 network capture scenarios from CTU University, Czech Republic (2011), containing mixed botnet and normal traffic. Key columns include:
- **StartTime:** Timestamp of the flow’s start.
- **Dur:** Duration of the flow.
- **Proto:** Protocol (focus on TCP).
- **SrcAddr / DstAddr:** Source and destination IP addresses.
- **Sport / Dport:** Source and destination ports.
- **State:** Contains TCP flag information (SYN, ACK, FIN, RST, PSH, URG).
- **sTos/dTos, TotPkts, TotBytes, SrcBytes:** Various traffic metrics.
- **Label:** Indicates the nature of the flow. *Note:* Flows with labels starting with “From-Botnet” are malicious; “To-Botnet” flows are not considered malicious; “From-Normal” indicates legitimate traffic.

Preprocessing includes converting timestamps, removing the first 25 minutes of flows to discard non-representative data, and filtering for TCP flows.

---

## Lakhina Entropy-Based PCA

### Why Entropy and PCA?
- **Entropy:** Measures the uncertainty in the distribution of network features (e.g., ports and IPs). Under normal conditions, these distributions are stable; anomalies (e.g., botnet traffic) cause significant changes.
- **PCA (Principal Component Analysis):** Reduces the dimensionality of the feature space while retaining most of the variance, thus simplifying the detection of outliers.

### Process Overview:
1. **Feature Extraction:** Calculate Shannon entropy for fields like source port, destination port, destination IP, and TCP flags.
2. **PCA Application:** Project entropy feature vectors into a lower-dimensional space.
3. **Anomaly Scoring:** Identify anomalous IPs based on their projections in the PCA subspaces.

---

## FLAGS Detection Approach

FLAGS builds on the principles of Lakhina’s method but uses TCP flag histograms instead of entropy values. It involves:
- **Extracting Flag Histograms:** Compute the histogram of TCP flags for each IP.
- **PCA Projection:** Apply PCA to these histograms to capture both dominant (normal) and residual (anomalous) behavior.
- **Anomaly Detection:** Flag an IP as anomalous if its projection in either the major or minor subspace exceeds a predefined threshold.

This method is designed to detect various types of attacks, such as TCP SYN Floods, TCP Reset Attacks, Port Scanning, and TCP Replay Attacks.

---

## Implementation Details

### Preprocessing and Feature Extraction
- **Utility Functions:**  
  - `compute_entropy()`: Computes Shannon entropy.  
  - `get_flow_label()`: Identifies malicious flows by checking for “From-Botnet”.  
  - `parse_state_flags()`: Extracts TCP flag indicators from the 'State' field.  
  - `add_flag_columns()`: Adds TCP flag columns to the DataFrame.  
  - `aggregate_features()`: Groups flows by source IP, calculates flag frequencies, entropies, and assigns an aggregated label.

- **Data Flow:**  
  1. Load the .binetflow file for a given scenario (e.g., scenario 12).  
  2. Filter for TCP flows and process the "State" column to extract flags.  
  3. Aggregate features by IP and save the processed data as a CSV.

### PCA and Anomaly Scoring
- **PCA Application:**  
  The `perform_pca()` function standardizes the selected features (e.g., entropies), applies PCA, and selects significant components based on an eigenvalue threshold.
  
- **Anomaly Scoring:**  
  The `calculate_anomaly_scores()` function projects the standardized data onto two subspaces (major and minor) and computes anomaly scores by normalizing each projection by its corresponding eigenvalue. A high score indicates significant deviation from normal behavior.

### Hybrid Model (PCA + Logistic Regression)
- **Model Training:**  
  A hybrid model is trained by combining PCA for dimensionality reduction with logistic regression for classification. This model is optimized to handle imbalanced data and provides probabilistic predictions.
  
- **Threshold Optimization:**  
  The system includes threshold optimization to convert continuous probability scores into binary decisions (normal vs. anomalous), maximizing the F1-score.

---

## Results and Evaluation

The project evaluates the detection system on the CTU-13 dataset using performance metrics such as:
- **True Positives (TP):** Correctly identified malicious IPs.
- **True Negatives (TN):** Correctly identified normal IPs.
- **False Positives (FP):** Normal IPs incorrectly flagged as malicious.
- **False Negatives (FN):** Malicious IPs that were missed.
- **Precision, Recall (TPR), Accuracy, F1-Score, and FPR.**

### Key Findings:
- **High Recall:** All malicious IPs were detected.
- **Low Precision:** Many normal IPs were flagged as malicious, reducing precision.
- **Impact of Class Imbalance:** With very few malicious IPs, overall accuracy is high but precision is a more reliable indicator.
- **No Time-Based Evaluation:** The evaluation was conducted on the entire aggregated dataset, without splitting data into time windows as suggested in the literature.

A heatmap of F1-scores was generated to optimize detection thresholds, helping to balance sensitivity and specificity.

---

## Usage

1. **Clone the Repository:**
   ```bash
   git clone https://github.com/yourusername/Botnet-detection-PCA-based-method-on-CTU-13-Dataset.git
   cd Botnet-detection-PCA-based-method-on-CTU-13-Dataset
