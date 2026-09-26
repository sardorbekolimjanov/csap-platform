# CSAP Development Roadmap

## Project

**CSAP — Computer System Anomaly Prediction Platform**

---

# Phase 0 — Project Definition

### Goal

Establish the project specifications before implementation.

### Tasks

* create repository;
* create `AGENTS.md`;
* create `docs/`;
* create `PROJECT_SPEC.md`;
* create `ARCHITECTURE.md`;
* create `ROADMAP.md`;
* review technology stack;
* define initial scope.

### Exit Criteria

* specifications reviewed;
* architecture approved;
* development environment defined.

---

# Phase 1 — Repository and Development Foundation

### Goal

Create a clean and testable software foundation.

### Tasks

* backend skeleton;
* frontend skeleton;
* Python environment;
* TypeScript environment;
* Docker configuration;
* Docker Compose;
* environment configuration;
* logging;
* health endpoint;
* testing framework;
* linting;
* formatting;
* basic CI.

### Expected structure

```text
backend/
frontend/
ml/
collector/
tests/
docs/
```

### Exit Criteria

The project starts successfully and basic tests pass.

---

# Phase 2 — Database

### Goal

Implement the persistence layer.

### Tasks

* PostgreSQL;
* TimescaleDB;
* SQLAlchemy;
* Alembic;
* database configuration;
* initial schema;
* migrations;
* indexes;
* time-series storage.

### Initial entities

```text
users
servers
metrics
anomalies
predictions
models
experiments
alerts
```

### Exit Criteria

Database starts through Docker and migrations execute successfully.

---

# Phase 3 — Monitoring and Data Collection

### Goal

Collect real computer-system performance metrics.

### Tasks

* Prometheus;
* Node Exporter;
* Windows Exporter;
* metric ingestion;
* server registration;
* metric normalization;
* historical storage;
* monitoring API.

### Initial metrics

```text
CPU
RAM
Disk
Network
Load
Processes
```

### Exit Criteria

Real system metrics appear in the CSAP dashboard and are stored in TimescaleDB.

---

# Phase 4 — Frontend Monitoring Dashboard

### Goal

Create the first usable CSAP interface.

### Pages

```text
Dashboard
Monitoring
Servers
```

### Features

* metric cards;
* time-series charts;
* server filtering;
* time-range filtering;
* system status.

### Exit Criteria

A researcher can select a server and inspect historical and current metrics.

---

# Phase 5 — Data Processing Pipeline

### Goal

Prepare time-series data for ML.

### Tasks

* missing-value handling;
* duplicate detection;
* timestamp processing;
* resampling;
* normalization;
* feature engineering;
* sliding windows;
* chronological dataset splitting.

### Exit Criteria

A reproducible dataset pipeline exists.

---

# Phase 6 — Classical Anomaly Detection

### Goal

Create baseline ML methods.

### Models

1. Isolation Forest
2. LOF
3. One-Class SVM
4. PCA-based method

### Tasks

* common detector interface;
* training;
* prediction;
* anomaly score;
* evaluation;
* model persistence.

### Exit Criteria

All baseline models can be trained and evaluated using the same experiment pipeline.

---

# Phase 7 — Autoencoder

### Goal

Introduce neural-network-based anomaly detection.

### Tasks

* PyTorch infrastructure;
* Autoencoder;
* training pipeline;
* reconstruction;
* reconstruction error;
* thresholding;
* evaluation.

### Exit Criteria

Autoencoder successfully participates in the same evaluation framework as classical models.

---

# Phase 8 — LSTM Autoencoder

### Goal

Implement temporal anomaly detection.

### Tasks

* sequence dataset;
* sliding-window loader;
* LSTM encoder;
* latent representation;
* LSTM decoder;
* reconstruction loss;
* anomaly score;
* model persistence;
* inference pipeline.

### Exit Criteria

LSTM Autoencoder can detect temporal anomalies on a real or controlled dataset.

---

# Phase 9 — Adaptive Threshold

### Goal

Research dynamic anomaly thresholds.

### Tasks

* static threshold baseline;
* adaptive threshold implementation;
* threshold configuration;
* threshold visualization;
* comparison experiments;
* false-positive analysis.

### Exit Criteria

Static and adaptive threshold methods can be experimentally compared.

---

# Phase 10 — Forecasting

### Goal

Predict future system behavior.

### Tasks

* forecasting dataset;
* LSTM forecasting model;
* optional GRU baseline;
* configurable forecasting horizon;
* MAE/RMSE evaluation;
* visualization.

### Exit Criteria

Future system metrics can be forecast and evaluated.

---

# Phase 11 — Future Anomaly Risk

### Goal

Connect forecasting and anomaly detection.

### Pipeline

```text
Historical metrics
        ↓
Forecast
        ↓
Future metrics
        ↓
Anomaly score
        ↓
Future risk
```

### Exit Criteria

The system can distinguish current anomalies from forecasted future anomaly risk.

---

# Phase 12 — Explainable AI

### Goal

Explain detected anomalies.

### Tasks

* SHAP integration where technically appropriate;
* feature contribution;
* explanation storage;
* explanation visualization;
* potential contributing factors.

### Exit Criteria

The system can display interpretable feature-level information for supported models.

---

# Phase 13 — Fault Injection

### Goal

Create controlled experimental scenarios.

### Scenarios

* CPU stress;
* memory stress;
* disk I/O stress;
* network stress;
* process stress;
* combined scenarios.

### Requirements

Every injection must be associated with an experiment ID and timestamps.

### Exit Criteria

Controlled anomalous scenarios can be reproduced and measured.

---

# Phase 14 — Experiment Management

### Goal

Make CSAP a reproducible scientific research platform.

### Tasks

* MLflow;
* experiment IDs;
* parameter tracking;
* dataset versions;
* model versions;
* random seeds;
* artifact storage;
* comparison tables.

### Exit Criteria

A complete experiment can be reproduced from recorded metadata.

---

# Phase 15 — Scientific Evaluation

### Goal

Evaluate the proposed method.

### Detection metrics

* Precision;
* Recall;
* F1;
* ROC-AUC where appropriate;
* PR-AUC;
* FPR;
* FNR;
* Detection Latency.

### Forecasting metrics

* MAE;
* RMSE;
* MAPE where appropriate.

### Experiments

* baseline comparison;
* adaptive vs static threshold;
* LSTM Autoencoder evaluation;
* forecasting evaluation;
* ablation study;
* sensitivity analysis.

### Exit Criteria

Real experimental results are available and reproducible.

---

# Phase 16 — Alerts and Reporting

### Goal

Turn research results into an operationally useful platform.

### Tasks

* anomaly alerts;
* Telegram integration;
* severity;
* alert history;
* reports;
* CSV export;
* Excel export;
* PDF export.

### Exit Criteria

Users can receive and review anomaly notifications and generate reports.

---

# Phase 17 — Security and Hardening

### Tasks

* authentication;
* JWT;
* role-based access;
* password hashing;
* API validation;
* rate limiting where appropriate;
* secure configuration;
* audit logging;
* dependency review.

### Exit Criteria

Security baseline is implemented and documented.

---

# Phase 18 — Final Research Release

### Tasks

* end-to-end testing;
* performance testing;
* ML validation;
* documentation review;
* architecture review;
* database review;
* API review;
* UI review;
* Docker deployment test;
* reproducibility test.

### Deliverables

```text
CSAP source code
+
Docker configuration
+
Documentation
+
Experimental datasets
+
Experiment configurations
+
Model artifacts
+
Evaluation results
+
Dissertation integration
```

---

# Development Gate Rule

A phase must not automatically trigger the next phase.

After each phase:

```text
Implementation
      ↓
Tests
      ↓
Review
      ↓
Documentation
      ↓
User approval
      ↓
Next phase
```

The next phase begins only after the previous phase satisfies its exit criteria.

---

# Research Priority

The primary research pipeline is:

```text
Multivariate Time Series
        ↓
LSTM Autoencoder
        ↓
Reconstruction Error
        ↓
Adaptive Threshold
        ↓
Anomaly Detection
        ↓
Forecasting
        ↓
Future Anomaly Risk
        ↓
Explainability
```

This pipeline must remain the central scientific direction of CSAP.

---

# Future Extensions

Potential future research directions:

* Transformer-based temporal models;
* online learning;
* continual learning;
* graph-based system modeling;
* distributed anomaly detection;
* advanced root-cause analysis;
* concept-drift detection.

These are not part of the initial master's implementation unless explicitly approved.
