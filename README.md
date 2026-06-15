# Network Traffic Anomaly Detection using Unsupervised Learning

This project builds a machine learning pipeline for detecting anomalous network traffic using unsupervised learning methods. The goal is to model normal network behavior from benign traffic and identify attack traffic as deviations from this learned baseline.

The project is inspired by real-world security monitoring scenarios where labeled attack data may be limited, but normal network traffic is available. Instead of training a supervised classifier on known attacks, the first experiment follows a normal-baseline anomaly detection setup.

## Project Objective

The main objective is to answer the following question:

> Can an anomaly detection model trained only on benign network traffic identify DDoS traffic as anomalous?

To investigate this, the project uses flow-level network traffic data and compares several unsupervised anomaly detection approaches:

- Isolation Forest
- Local Outlier Factor
- DBSCAN
- Autoencoder-based reconstruction error

## Dataset

The project uses the CICIDS2017 network intrusion detection dataset.

CICIDS2017 contains network traffic captured in a controlled enterprise-like environment. The dataset includes both benign traffic and several attack types such as DDoS, DoS, PortScan, Bot, Web Attacks, Infiltration, FTP-Patator, and SSH-Patator.

The dataset is provided as flow-level CSV files. Each row represents a network flow, and each column describes statistical properties of that flow, such as packet counts, byte rates, flow duration, packet length statistics, and TCP flag counts.

Example feature groups include:

- Flow duration features
- Packet count features
- Packet length statistics
- Flow byte and packet rates
- TCP flag counts
- Active and idle time statistics

The `Label` column indicates whether a flow is `BENIGN` or belongs to a specific attack category.

## Dataset Files Used

For the first experiment, two files were selected:

| Role | File | Description |
|---|---|---|
| Training baseline | `Monday-WorkingHours.pcap_ISCX.csv` | Contains only BENIGN traffic |
| Evaluation data | `Friday-WorkingHours-Afternoon-DDos.pcap_ISCX.csv` | Contains BENIGN and DDoS traffic |

The Monday file is used to learn normal network behavior. The Friday afternoon DDoS file is used to evaluate whether the trained models can identify DDoS flows as anomalies.

## Experiment Design

The experiment follows a normal-only anomaly detection setup:

1. Train models using only BENIGN traffic from Monday.
2. Evaluate on mixed BENIGN and DDoS traffic from Friday.
3. Use labels only for evaluation, not for training.
4. Compare anomaly scores and threshold-based detection performance.

This setup is closer to a realistic monitoring scenario where a system first learns normal behavior and then flags unusual traffic patterns.

## Project Structure

```text
network-traffic-anomaly-detection/
│
├── README.md
├── requirements.txt
├── .gitignore
│
├── notebooks/
│   ├── 01_data_understanding.ipynb
│   ├── 02_eda_preprocessing.ipynb
│   └── 03_baseline_anomaly_detection.ipynb
│
├── data/
│   ├── raw/
│   └── processed/
│
└── reports/
    └── figures/