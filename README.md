# XSS-Detection-using-Apache-Spark-MLlib

This repository contains a distributed machine learning pipeline implemented in PySpark to detect Cross-Site Scripting (XSS) attacks within large-scale network traffic data. The project addresses severe class imbalance using Cost-Sensitive Learning and evaluates parallel scalability across different Spark worker configurations.

## Dataset Overview
The project utilizes the CICIDS2017 dataset. After applying the data cleaning pipeline to remove nulls, missing fields, and infinite values, the final dataset contains 2,827,876 valid network flow records.
* **Total Records:** 2,827,876
* **Normal Traffic Records:** 2,827,224
* **XSS Attack Records:** 652
* **Class Ratio:** 0.0231%

To reduce computational overhead and data serialization latency between worker processes, the feature space was reduced to 5 critical numerical features:
1. **Destination Port:** Provides HTTP/HTTPS service context.
2. **Flow Duration:** Captures temporal traffic patterns.
3. **Total Fwd Packets:** Tracks forward traffic volume.
4. **Fwd Packet Length Max:** Captures payload characteristics.
5. **Packet Length Mean:** Reflects anomalous traffic behavior associated with XSS.

## Class Imbalance Mitigation
Instead of synthetic oversampling techniques like SMOTE, which introduce memory overhead and disk spilling in distributed environments, a Cost-Sensitive Learning approach was implemented. 

Class weights were incorporated into the Spark MLlib pipeline via the `weightCol` parameter, calculated inversely proportional to class frequencies:
* **Majority Class (Normal Traffic) Weight:** 0.5001
* **Minority Class (XSS Traffic) Weight:** 2168.61

## Distributed Environment Setup
* **Framework:** PySpark with Apache Spark 3.4.1 MLlib
* **Cluster Environment:** Pseudo-distributed Spark Standalone cluster running on independent Java Virtual Machine (JVM) processes to emulate distribution coordination overhead.
* **Hardware Configuration:** Workstation with 32 GB RAM.
* **Executor Configuration:** 3 GB RAM per worker node.
* **Data Partitioning:** 80% Training data, 20% Testing data partitioned using a fixed random seed to ensure reproducibility.

## Experimental Results

### 1. Model Performance Comparison
The models were evaluated under a 4-worker cluster configuration:

| Model | Training Time | AUC | Recall (XSS) | Precision (XSS) |
| :--- | :--- | :--- | :--- | :--- |
| Logistic Regression | 65.16 s | 0.9502 | 98.23% | 0.043% |
| Random Forest (50 Trees) | 121.63 s | 0.9956 | 97.34% | 1.65% |

### 2. Distributed Scalability and Efficiency
The scalability of the Random Forest model was evaluated across different Spark worker configurations:

| Number of Spark Workers | Training Time | Speedup ($S_P$) | Efficiency ($E_P$) |
| :--- | :--- | :--- | :--- |
| 1 Worker | 414.41 s | 1.00x | 100.0% |
| 2 Workers | 302.11 s | 1.37x | 68.5% |
| 4 Workers | 121.63 s | 3.41x | 85.2% |

The non-linear efficiency increase from 2 workers to 4 workers indicates a memory threshold effect, where distributing the workload across more executors reduced memory utilization pressure and mitigated disk I/O bottlenecks.

## Installation

Install the required Python packages using:

```bash
pip install -r requirements.txt
```

## Run the Experiments

Launch Jupyter Notebook:

```bash
jupyter notebook
```

Then open one of the following notebooks depending on the desired Spark worker configuration:

- `xss_detection-1worker.ipynb`
- `xss_detection-2workers.ipynb`
- `xss_detection-4workers.ipynb`

---

## Dataset Source

The CICIDS2017 dataset is publicly available from the Canadian Institute for Cybersecurity (CIC):

https://www.unb.ca/cic/datasets/ids-2017.html




