# Network Traffic Anomaly Detection using Unsupervised Learning

This project builds a machine learning pipeline for detecting anomalous network traffic using unsupervised learning methods. The main goal is to model normal network behavior from benign traffic and identify attack traffic as deviations from the learned baseline.

The project is designed around a realistic security-monitoring scenario where normal traffic is available, but labeled attack data may be limited or unavailable during training. Instead of training a supervised classifier on known attacks, the project follows a normal-baseline anomaly detection approach.

---

## Project Objective

The main research question is:

> Can an anomaly detection model trained only on benign network traffic identify DDoS traffic as anomalous?

To answer this, the project uses flow-level network traffic data and compares multiple unsupervised anomaly detection approaches:

* Isolation Forest
* Local Outlier Factor
* DBSCAN
* Autoencoder-based reconstruction error

---

## Dataset

This project uses the CICIDS2017 network intrusion detection dataset.

CICIDS2017 contains network traffic captured in a controlled enterprise-like environment. The dataset includes benign traffic and several attack types, including:

* DDoS
* DoS
* PortScan
* Bot
* Web Attacks
* Infiltration
* FTP-Patator
* SSH-Patator
* Heartbleed

The dataset is provided as flow-level CSV files. Each row represents a network flow, and each column describes statistical properties of that flow.

Example feature groups include:

* Flow duration features
* Packet count features
* Packet length statistics
* Flow byte and packet rates
* TCP flag counts
* Active and idle time statistics

The `Label` column indicates whether a flow is `BENIGN` or belongs to a specific attack category.

---

## Dataset Files Used

For the first experiment, two files were selected:

| Role              | File                                               | Description                      |
| ----------------- | -------------------------------------------------- | -------------------------------- |
| Training baseline | `Monday-WorkingHours.pcap_ISCX.csv`                | Contains only BENIGN traffic     |
| Evaluation data   | `Friday-WorkingHours-Afternoon-DDos.pcap_ISCX.csv` | Contains BENIGN and DDoS traffic |

The Monday file is used to learn normal network behavior. The Friday afternoon DDoS file is used to evaluate whether the trained models can identify DDoS flows as anomalies.

---

## Experiment Design

The experiment follows a normal-only anomaly detection setup:

1. Train models using only BENIGN traffic from Monday.
2. Evaluate the trained models on mixed BENIGN and DDoS traffic from Friday.
3. Use labels only for evaluation, not for training.
4. Compare models using anomaly scores and threshold-based detection metrics.

This setup is closer to a practical security monitoring scenario where a system first learns normal behavior and then flags unusual traffic patterns.

---

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
```

The `data/` directory is not tracked in Git because the raw and processed dataset files are large.

---

## Notebook Overview

### 1. Data Understanding

The first notebook explores the dataset structure and label distribution across the available CSV files.

Main steps:

* List all dataset files
* Inspect rows and columns
* Check available labels
* Identify benign-only and attack-containing files
* Select the initial baseline/evaluation setup

The key finding is that `Monday-WorkingHours.pcap_ISCX.csv` contains only BENIGN traffic, making it suitable as a normal baseline.

---

### 2. EDA and Preprocessing

The second notebook prepares the selected training and evaluation files for modeling.

Main steps:

* Load Monday BENIGN and Friday DDoS files
* Inspect data types
* Check missing values
* Check infinite values
* Check duplicate rows
* Clean invalid rows
* Remove duplicate rows
* Separate features and labels
* Convert labels to binary format
* Scale features using `StandardScaler`

The cleaning process handles invalid values in rate-based features such as:

* `Flow Bytes/s`
* `Flow Packets/s`

These features can contain missing or infinite values when flow duration is zero or extremely small.

---

### 3. Baseline Anomaly Detection

The third notebook trains and evaluates unsupervised anomaly detection models.

Models evaluated:

* Isolation Forest
* Local Outlier Factor
* DBSCAN
* Autoencoder

The models are trained only on benign traffic and evaluated on mixed benign + DDoS traffic.

---

## Preprocessing Summary

Initial data quality checks showed missing values, infinite values, and duplicate rows.

| Dataset       | Missing Values | Infinite Values | Duplicate Rows |
| ------------- | -------------: | --------------: | -------------: |
| Training data |             64 |             810 |         26,935 |
| Test data     |              4 |              64 |          2,633 |

After cleaning:

| Dataset       | Original Shape |  Cleaned Shape |
| ------------- | -------------: | -------------: |
| Training data | `(529918, 79)` | `(502650, 79)` |
| Test data     | `(225745, 79)` | `(223082, 79)` |

After preprocessing, the cleaned datasets contained:

* 0 missing values
* 0 infinite values
* 0 duplicate rows

The test labels were converted into binary format:

* `0` = BENIGN
* `1` = DDoS

---

## Models

### Isolation Forest

Isolation Forest is a tree-based anomaly detection method. It isolates observations by randomly selecting features and split values. Anomalies are expected to be isolated more easily than normal samples.

In this project, Isolation Forest is trained only on benign traffic. Higher anomaly scores indicate more suspicious flows.

---

### Local Outlier Factor

Local Outlier Factor compares the local density of a sample with the density of its neighbors. Samples that exist in lower-density regions compared to nearby points are considered more anomalous.

LOF was evaluated in novelty detection mode, allowing the model to score unseen test samples.

---

### DBSCAN

DBSCAN is a density-based clustering algorithm. It groups dense regions into clusters and marks low-density points as noise.

In this project, DBSCAN is used as an exploratory clustering baseline. Since scikit-learn's DBSCAN does not provide direct novelty scoring for unseen test samples, it is not used as the main test-set anomaly detector.

DBSCAN helped analyze whether benign network traffic forms dense clusters and how sensitive the clustering is to the `eps` parameter.

---

### Autoencoder

The autoencoder is a neural reconstruction-based anomaly detector. It is trained to reconstruct benign traffic features.

During evaluation, reconstruction error is used as the anomaly score:

* Low reconstruction error = normal-like traffic
* High reconstruction error = anomalous traffic

Since the autoencoder is trained only on benign traffic, DDoS flows are expected to produce higher reconstruction errors.

---

## Results

The final baseline comparison is shown below.

| Model                | Selected Threshold Percentile | ROC-AUC | PR-AUC | Precision | Recall | F1-score | False Positive Rate | Notes                                |
| -------------------- | ----------------------------: | ------: | -----: | --------: | -----: | -------: | ------------------: | ------------------------------------ |
| Isolation Forest     |                            90 |  0.6809 | 0.6495 |    0.7706 | 0.6361 |   0.6969 |              0.2550 | Tree-based baseline                  |
| Local Outlier Factor |                            95 |  0.8027 | 0.8116 |    0.7891 | 0.5737 |   0.6644 |              0.2065 | Best ranking performance             |
| Autoencoder          |                            97 |  0.7533 | 0.7912 |    0.7546 | 0.6278 |   0.6854 |              0.2749 | Neural reconstruction-based detector |
| DBSCAN               |                             — |       — |      — |         — |      — |        — |                   — | Exploratory clustering baseline      |

---

## Result Interpretation

Local Outlier Factor achieved the strongest ranking performance, with the highest ROC-AUC and PR-AUC. This means LOF was best at assigning higher anomaly scores to DDoS traffic compared to benign traffic.

The autoencoder also performed well and produced a meaningful reconstruction-error signal. DDoS flows had higher reconstruction errors than benign flows, showing that the model learned normal traffic patterns and struggled more to reconstruct attack traffic.

Isolation Forest provided a useful initial baseline, but its ranking performance was weaker than LOF and the autoencoder.

DBSCAN showed that benign traffic forms multiple dense clusters. However, because DBSCAN does not directly support novelty scoring for unseen test data in scikit-learn, it was used only as an exploratory clustering method.

Overall, LOF and the autoencoder were the strongest models in this first experiment.

---

## Key Findings

* Normal network traffic can be modeled using benign-only training data.
* DDoS traffic receives higher anomaly scores than benign traffic across multiple models.
* LOF achieved the best ranking performance.
* Autoencoder reconstruction error provides an interpretable anomaly signal.
* Threshold selection strongly affects the trade-off between detection rate and false positive rate.
* DBSCAN is useful for exploring density structure but less practical for test-time anomaly scoring in this setup.

---

## Limitations

This project is an initial benchmark experiment and has some limitations:

* The first experiment evaluates only DDoS traffic.
* The dataset is a benchmark enterprise-style IDS dataset, not a pure OT/ICS dataset.
* The models were trained on one benign day and tested on one attack scenario.
* DBSCAN was evaluated only as an exploratory method due to scalability and novelty-scoring limitations.
* Thresholds were selected experimentally and may not generalize to all attack types.
* No feature selection or dimensionality reduction was applied yet.

---

## Future Work

Possible improvements include:

* Evaluate on additional attack types such as PortScan, Bot, DoS, FTP-Patator, SSH-Patator, and Web Attacks.
* Add feature selection to reduce noisy or redundant features.
* Apply PCA or UMAP for visualization and dimensionality reduction.
* Tune model hyperparameters more systematically.
* Train autoencoders with validation-based early stopping.
* Compare dense autoencoders with deeper or regularized architectures.
* Evaluate false positive rate under different alerting thresholds.
* Extend the pipeline to OT/ICS datasets with protocol-specific features such as Modbus or OPC-UA communication patterns.

---

## How to Run

### 1. Clone the repository

```bash
git clone https://github.com/manuviswakarmave/network-intrusion-detection.git
cd network-traffic-anomaly-detection
```

### 2. Create a virtual environment

```bash
python -m venv .venv
```

Activate it on Windows:

```bash
.venv\Scripts\activate
```

Activate it on Linux/macOS:

```bash
source .venv/bin/activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Download the dataset

This project uses the CICIDS2017 CSV dataset. The dataset files are not included in the repository because they are large.

One option is to download the dataset using KaggleHub:

```python
import kagglehub

path = kagglehub.dataset_download("chethuhn/network-intrusion-dataset")
print("Dataset downloaded to:", path)
```

Then copy the downloaded files into:

```text
data/raw/network-intrusion-dataset/versions/1/
```

Expected files include:

```text
Monday-WorkingHours.pcap_ISCX.csv
Friday-WorkingHours-Afternoon-DDos.pcap_ISCX.csv
Friday-WorkingHours-Afternoon-PortScan.pcap_ISCX.csv
Friday-WorkingHours-Morning.pcap_ISCX.csv
Tuesday-WorkingHours.pcap_ISCX.csv
Wednesday-workingHours.pcap_ISCX.csv
Thursday-WorkingHours-Morning-WebAttacks.pcap_ISCX.csv
Thursday-WorkingHours-Afternoon-Infilteration.pcap_ISCX.csv
```

### 5. Run the notebooks

Run the notebooks in order:

```text
01_data_understanding.ipynb
02_eda_preprocessing.ipynb
03_baseline_anomaly_detection.ipynb
```

---

## Technologies Used

* Python
* pandas
* NumPy
* scikit-learn
* PyTorch
* Jupyter Notebook
* KaggleHub

---

## Project Status

The first baseline experiment is complete.

Completed:

* Data understanding
* EDA and preprocessing
* Isolation Forest baseline
* LOF baseline
* DBSCAN exploratory clustering
* Autoencoder anomaly detection
* Final baseline comparison

Next planned step:

* Extend evaluation to more attack types and improve model selection.

---

## Git Ignore Recommendation

The dataset and processed files should not be committed to Git.

Recommended `.gitignore`:

```gitignore
data/raw/
data/processed/
.idea/
.ipynb_checkpoints/
__pycache__/
*.pyc
models/
```
