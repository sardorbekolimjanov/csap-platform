# CSAP — Project Specification

## 1. Project Overview

**Project name:** CSAP

**Full name:** Computer System Anomaly Prediction Platform

**Project type:** Research-oriented intelligent monitoring and anomaly prediction platform.

**Primary research domain:**

* Computer systems
* Performance monitoring
* Multivariate time-series analysis
* Machine learning
* Deep learning
* Anomaly detection
* Time-series forecasting
* Explainable AI

---

# 2. Dissertation Context

CSAP is the main software artifact supporting the master's dissertation:

> “Kompyuter tizimlarining ishlash ko‘rsatkichlaridagi anomaliyalarni mashinaviy o‘qitish asosida aniqlash va prognozlash usulini ishlab chiqish.”

The platform must provide practical infrastructure for implementing, testing, comparing, and evaluating the methods proposed in the dissertation.

---

# 3. Main Research Problem

Modern computer systems continuously generate large volumes of performance data.

Examples include:

* CPU utilization;
* memory utilization;
* disk I/O;
* network traffic;
* system load;
* process count;
* latency;
* I/O wait;
* packet statistics.

Normal system behavior is not necessarily constant. It may change according to workload, time, applications, users, and system state.

Therefore, CSAP must address:

1. detection of abnormal system behavior;
2. distinction between normal and anomalous temporal patterns;
3. reduction of false alarms;
4. estimation of anomaly severity;
5. forecasting of future anomalous behavior;
6. identification of potentially contributing metrics;
7. reproducible scientific evaluation of different methods.

---

# 4. Main Goal

The main goal of CSAP is to develop a software platform capable of:

> collecting multivariate computer-system performance metrics, processing them as time-series data, detecting anomalous states using machine-learning and deep-learning methods, forecasting potential future anomalies, and providing interpretable results through a unified research and monitoring interface.

---

# 5. Target Users

## Researcher

Can:

* configure datasets;
* train models;
* run experiments;
* compare algorithms;
* inspect metrics;
* analyze anomalies;
* export results.

## Administrator

Can:

* manage users;
* register monitored systems;
* configure collectors;
* configure alerts;
* manage system settings.

## Viewer

Can:

* view dashboards;
* inspect metrics;
* view anomalies;
* view predictions;
* view reports.

---

# 6. Core Functional Requirements

## FR-01 — System Registration

The platform must support registration of monitored computers/servers.

Each server should have:

* unique ID;
* name;
* hostname;
* operating system;
* environment;
* status;
* registration timestamp;
* optional metadata.

---

## FR-02 — Metric Collection

The system must collect performance metrics.

Minimum metric groups:

### CPU

* utilization;
* user time;
* system time;
* idle time;
* iowait;
* load.

### Memory

* total;
* used;
* available;
* cached;
* swap;
* utilization.

### Disk

* read throughput;
* write throughput;
* IOPS;
* utilization;
* latency.

### Network

* incoming traffic;
* outgoing traffic;
* packets;
* errors;
* dropped packets.

### System

* process count;
* thread count;
* load average;
* uptime.

The architecture must permit additional metrics.

---

# 7. Monitoring Requirements

CSAP must provide:

* real-time metric visualization;
* historical visualization;
* selectable time ranges;
* metric filtering;
* server filtering;
* anomaly overlays;
* prediction overlays.

---

# 8. Data Processing

The platform must support:

1. missing-value handling;
2. duplicate detection;
3. timestamp normalization;
4. resampling;
5. scaling/normalization;
6. feature engineering;
7. sliding-window construction;
8. train/validation/test splitting.

Time-series chronology must be preserved.

---

# 9. Classical ML Models

The initial ML benchmark should include:

1. Isolation Forest;
2. Local Outlier Factor;
3. One-Class SVM;
4. PCA-based anomaly detection.

The architecture must allow future algorithms to be added without rewriting the pipeline.

---

# 10. Deep Learning Models

The platform must support:

1. Autoencoder;
2. LSTM Autoencoder.

The LSTM Autoencoder is the principal temporal anomaly-detection model for the research prototype.

---

# 11. Anomaly Detection

For reconstruction-based models, anomaly score should be derived from reconstruction error.

A conceptual form is:

$$
A_t =
\frac{1}{n}
\sum_{i=1}^{n}
(x_{t,i}-\hat{x}_{t,i})^2
$$

where:

* \(x\) is the observed value;
* \(\hat{x}\) is the reconstructed value;
* \(A_t\) is anomaly score.

The exact implementation must be documented and tested.

---

# 12. Thresholding

The platform must support:

### Static threshold

A fixed threshold selected from training/validation data.

### Adaptive threshold

A threshold dynamically derived from historical model behavior.

A possible research formulation is:

$$
T_t = \mu_t + k\sigma_t
$$

where:

* \(\mu_t\) is a rolling statistic;
* \(\sigma_t\) is a rolling dispersion statistic;
* \(k\) is a configurable parameter.

This formula is a research implementation option and must be experimentally validated rather than assumed to be optimal.

---

# 13. Anomaly Severity

The platform should classify anomalies into configurable levels such as:

* NORMAL;
* WARNING;
* MEDIUM;
* HIGH;
* CRITICAL.

Severity logic must be explicitly defined and versioned.

---

# 14. Forecasting

The system must support future metric prediction.

Initial forecasting model:

**LSTM**

Optional comparative model:

**GRU**

Input:

historical multivariate time-series window.

Output:

future values over a configurable forecasting horizon.

---

# 15. Future Anomaly Prediction

Forecasted metrics should be evaluated against the anomaly-detection mechanism to estimate future anomaly risk.

Conceptual pipeline:

```text
Historical metrics
        ↓
Forecasting model
        ↓
Future metrics
        ↓
Anomaly scoring
        ↓
Future anomaly risk
```

The platform must distinguish between:

* currently detected anomaly;
* forecasted metric deviation;
* predicted future anomaly risk.

These concepts must not be mixed.

---

# 16. Explainability

CSAP should provide feature-level explanations for detected anomalies.

The initial explainability technology is:

**SHAP**

The interface should allow researchers to inspect:

* contributing metrics;
* contribution magnitude;
* contribution direction where supported;
* explanation timestamp;
* model version.

Explanations must not be represented as causal proof unless causal methodology has actually been implemented.

Use terminology such as:

> “potential contributing factor”

when causal certainty is not established.

---

# 17. Fault Injection

A controlled experimental environment should support configurable fault scenarios:

* CPU stress;
* memory stress;
* disk I/O stress;
* network stress;
* process-load stress;
* combined stress.

Fault injection must be explicitly marked as experimental/laboratory functionality.

The system must record:

* injection start;
* injection end;
* target system;
* fault type;
* configuration;
* experiment ID.

---

# 18. Scientific Evaluation

The platform must support evaluation using:

* Precision;
* Recall;
* F1-score;
* ROC-AUC where appropriate;
* PR-AUC;
* False Positive Rate;
* False Negative Rate;
* Detection Latency.

Forecasting should additionally use appropriate forecasting metrics such as:

* MAE;
* RMSE;
* MAPE where mathematically appropriate.

Metric selection must depend on the experimental task and data characteristics.

---

# 19. Experiment Management

Every experiment should have:

* experiment ID;
* dataset/version;
* model;
* model version;
* parameters;
* preprocessing configuration;
* training configuration;
* random seed;
* evaluation configuration;
* metrics;
* artifacts;
* execution timestamp.

MLflow should be used for experiment tracking where appropriate.

---

# 20. Model Registry

Models should have:

* unique ID;
* name;
* version;
* algorithm;
* parameters;
* training dataset;
* training timestamp;
* evaluation results;
* artifact location;
* status.

Example statuses:

```text
DRAFT
TRAINED
EVALUATED
VALIDATED
DEPLOYED
ARCHIVED
```

---

# 21. Alerting

The system should support anomaly alerts.

Initial channels may include:

* dashboard notification;
* application notification;
* Telegram notification.

Alerts should contain:

* server;
* metric/context;
* timestamp;
* anomaly score;
* severity;
* model version;
* potential contributing factors where available.

---

# 22. Reporting

The platform should eventually support export of:

* anomaly reports;
* experiment reports;
* model comparison tables;
* metric histories;
* prediction results;
* evaluation results.

Formats may include:

* CSV;
* JSON;
* Excel;
* PDF.

---

# 23. Frontend Requirements

The frontend should contain at least:

### Dashboard

System overview.

### Monitoring

Real-time and historical metrics.

### Anomalies

Detected anomaly list and details.

### Predictions

Forecasting and future risk.

### Models

Model registry and model status.

### Experiments

Scientific experiment tracking.

### Explainability

Feature contribution analysis.

### Reports

Research and monitoring reports.

### Settings

System configuration.

---

# 24. Backend Requirements

Backend responsibilities:

* authentication;
* authorization;
* server management;
* metrics API;
* anomaly API;
* prediction API;
* model management;
* experiment management;
* alert management;
* reporting;
* system health.

Backend technology:

**FastAPI + Python**

---

# 25. Database Requirements

Primary database:

**PostgreSQL + TimescaleDB**

The database must support:

* relational application data;
* high-volume time-series metrics;
* efficient time-range queries;
* experiment metadata;
* model metadata;
* anomaly records;
* forecast records.

---

# 26. Monitoring Infrastructure

Initial monitoring architecture:

```text
Computer/Server
      ↓
Exporter / Agent
      ↓
Prometheus
      ↓
CSAP Data Pipeline
      ↓
TimescaleDB
      ↓
ML Pipeline
```

---

# 27. Non-Functional Requirements

## Performance

The system should support near-real-time monitoring under a configurable sampling interval.

## Reliability

Temporary failures in data collection must not corrupt historical data.

## Security

Secrets must be externalized.

## Maintainability

Modules must have clear responsibilities.

## Reproducibility

ML experiments must be repeatable from recorded configurations.

## Extensibility

New models and metrics should be addable without major architectural changes.

---

# 28. Scientific Integrity

CSAP must never fabricate:

* experimental results;
* model metrics;
* datasets;
* scientific references;
* benchmark results.

If no real result exists, the application must clearly represent the result as unavailable/not evaluated.

---

# 29. Initial Scope

The first production-quality research prototype should prioritize:

```text
Monitoring
+
Time-Series Storage
+
Classical ML
+
LSTM Autoencoder
+
Adaptive Threshold
+
Forecasting
+
Experiment Tracking
+
Dashboard
```

Explainability, fault injection, and advanced alerting can be implemented after the core anomaly pipeline is stable.

---

# 30. Out of Scope for Initial Version

Do not initially implement unless explicitly approved:

* Kubernetes;
* large-scale distributed Kafka architecture;
* federated learning;
* reinforcement learning;
* graph neural networks;
* mobile applications;
* native desktop applications;
* complex microservice infrastructure.

These may become future research extensions.

---

# 31. Success Criteria

The CSAP research prototype will be considered successful when:

1. It can collect real system metrics.
2. Metrics can be stored and queried as time series.
3. Classical anomaly detectors work through a common interface.
4. LSTM Autoencoder can be trained and evaluated.
5. Anomaly scores can be calculated.
6. Static and adaptive thresholds can be compared.
7. Future metrics can be forecast.
8. Future anomaly risk can be estimated.
9. Experiments are reproducible.
10. Results can be visualized.
11. No scientific result is fabricated.
12. The implementation can support dissertation experiments.
