# 🏎️ Formula 1 Predictive Analytics Architecture in Microsoft Fabric

[![Microsoft Fabric](https://img.shields.io/badge/Microsoft%20Fabric-Data%20Engineering-blue)](https://azure.microsoft.com/en-us/solutions/microsoft-fabric/)
[![PySpark](https://img.shields.io/badge/PySpark-Data%20Processing-orange)](https://spark.apache.org/docs/latest/api/python/)
[![XGBoost](https://img.shields.io/badge/XGBoost-Machine%20Learning-green)](https://xgboost.readthedocs.io/)
[![Python](https://img.shields.io/badge/Python-3.10-blue)](https://www.python.org/)

An end-to-end, fully automated Data Engineering and Machine Learning pipeline built natively within **Microsoft Fabric**. 

This project ingests live Formula 1 telemetry, processes it through a highly scalable Medallion Lakehouse architecture (Bronze, Silver, Gold) using PySpark, and deploys an XGBoost Classification engine. The pipeline automatically evaluates Friday Race Pace and Saturday Qualifying track position to predict Sunday race outcomes, emailing the results autonomously.

## 🏗️ Enterprise Architecture Overview

<img width="2752" height="1536" alt="unnamed" src="https://github.com/user-attachments/assets/fdf63f55-6a5f-4297-88ef-1768855ec254" />

<img width="847" height="307" alt="architecture" src="https://github.com/user-attachments/assets/9a4220f8-5f9e-45ed-8952-3b3e6cd6ef94" />

### The Tech Stack
* **Compute Engine:** Apache Spark (Microsoft Fabric Synapse Data Engineering)
* **Storage:** Delta Lake / Parquet Files (Fabric OneLake)
* **Orchestration:** Fabric Data Factory (DAG Pipelines & Schedule Triggers)
* **Data Sources:** Live F1 Telemetry & Weather Data (`fastf1` API)
* **Machine Learning:** `xgboost`, `scikit-learn`, `pandas`
* **Alerting:** Office 365 Outlook Integration

---

## ⚙️ The Medallion Lakehouse Pipeline

This project adheres strictly to the Medallion Data Architecture paradigm, ensuring data quality, lineage, and scalability.

### 🥉 Bronze Layer (Raw Ingestion)
The ingestion layer connects to the `fastf1` Python API to hoard massive JSON telemetry files. 
* **Dynamic Calendar Handling:** Code features self-healing `try/except` blocks to account for real-world anomalies like F1 Sprint Weekends or cancelled sessions.
* **Local Caching:** Implements temporary cluster caching to prevent API rate-limiting and ensure idempotent pipeline runs.

### 🥈 Silver Layer (Cleansing & Conforming)
Raw F1 data is notoriously messy. The PySpark Silver pipeline standardizes the data lake:
* **Anomaly Scrubbing:** Drops invalid laps, 15-minute red flag pit stops, and experimental Pirelli tire tests (`IsAccurate == True`).
* **Regex Time Parsing:** Converts string-based F1 time-deltas (e.g., `"0 days 00:01:30.505"`) into mathematically aggregatable numeric seconds.
* **Weather Integration:** Joins thousands of granular track temperature and weather logs to specific driver stints.

### 🥇 Gold Layer (Feature Engineering)
The business logic layer. Using advanced PySpark `Window` functions, the code engineers strictly **Regulation-Proof Features** that remain accurate regardless of yearly FIA car changes:
* `Qualifying_Delta_To_Pole`: The absolute time difference between a driver's best Q1/Q2/Q3 lap and the Pole Position lap.
* `Pace_Delta_To_Leader`: The delta between a driver's mathematical average heavy-fuel FP2 race simulation and the fastest car on track.

---

## 🧠 Machine Learning Engine (The Saturday Night Predictor)

<img width="602" height="311" alt="Prediction Output" src="https://github.com/user-attachments/assets/682a0b62-01d4-494b-974f-ea1a8fd41033" />

Instead of attempting to predict highly volatile exact lap times, the AI is modeled as a **Classification Engine**.

* **The Target Variable:** `Is_Podium_Contender` (Predicting if a driver possesses both top-5 track position and top-5 race pace).
* **Time-Series Holdout Validation:** The XGBoost model was trained strictly on 2024–2025 historical data and validated against a blind holdout of a 2026 race.
* **Inference Logic:** Uses `.predict_proba()` to output exact percentage confidence scores rather than rigid binary predictions, accurately modeling the "Chaos Factor" of live sports.

---

## 🔄 Automation & Orchestration (Fabric Data Factory)

<img width="1242" height="197" alt="Fabric DAG" src="https://github.com/user-attachments/assets/0881c736-8541-4e74-99ac-ce85526213fb" />
<img width="563" height="549" alt="F1 Data Analysis Workspace" src="https://github.com/user-attachments/assets/9f05b615-1724-4eed-8008-d6f523ba7795" />

The entire architecture is fully autonomous. 
* **DAG Implementation:** Notebooks are chained dependently. Silver only runs if Bronze succeeds; ML only runs if Gold succeeds.
* **Schedule Trigger:** The pipeline awakens weekly on Saturday night after real-world Qualifying concludes.
* **Dynamic Alerting:** Utilizing the `mssparkutils.notebook.exit()` command, the PySpark cluster passes the final AI string predictions back to the Fabric Orchestrator, triggering an automated Office 365 Email detailing the Top 3 predicted drivers directly to the stakeholders' inbox.

---

## 🚀 How to Run this Repository

To replicate this environment in your own Microsoft Fabric or Databricks workspace:

1. Clone this repository to your local machine.
2. Import the `1_Master_Ingestion_Pipeline.ipynb` into your Spark workspace. Ensure your cluster has internet access to install the `fastf1` library via `%pip install`.
3. Create a unified Lakehouse in your workspace.
4. Import the remaining Silver, Gold, and ML notebooks. 
5. Build a Data Factory Pipeline chaining the 4 notebooks together.
6. (Optional) Append an Office 365 or Webhook activity to the end of the pipeline to capture the `mssparkutils` exit value for notifications.

---

Further Updates:
1. Bring visualisations for different analyses.
