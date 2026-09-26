# CSAP — System Architecture

## 1. Document Purpose

This document defines the technical architecture of the **CSAP — Computer System Anomaly Prediction Platform**.

CSAP is a research-oriented software platform designed to support the collection, processing, analysis, detection, forecasting, explanation, and evaluation of anomalies in computer-system performance metrics.

The architecture must support both:

1. practical computer-system monitoring;
2. reproducible scientific experimentation for the master's dissertation:

> **“Kompyuter tizimlarining ishlash ko‘rsatkichlaridagi anomaliyalarni mashinaviy o‘qitish asosida aniqlash va prognozlash usulini ishlab chiqish.”**

This document is a living specification. Any architectural change must be reflected here.

---

# 2. Architectural Goals

The architecture must satisfy the following goals:

* modularity;
* reproducibility;
* scientific traceability;
* separation of monitoring and research components;
* extensibility of ML algorithms;
* time-series integrity;
* prevention of data leakage;
* explainability;
* controlled experimentation;
* maintainability;
* testability;
* observability;
* security;
* containerized deployment.

The system must be designed so that a new anomaly-detection algorithm can be added without rewriting the monitoring, database, API, or frontend layers.

---

# 3. High-Level Architecture

```text
┌──────────────────────────────────────────────────────────────┐
│                        USER / RESEARCHER                     │
│                     Web Browser / Dashboard                  │
└───────────────────────────────┬──────────────────────────────┘
                                │ HTTPS
                                ▼
┌──────────────────────────────────────────────────────────────┐
│                    FRONTEND APPLICATION                      │
│                  React + TypeScript + Vite                  │
│                                                              │
│  Dashboard │ Monitoring │ Anomalies │ Forecasts │ XAI       │
│  Experiments │ Models │ Fault Injection │ Reports           │
└───────────────────────────────┬──────────────────────────────┘
                                │ REST API
                                ▼
┌──────────────────────────────────────────────────────────────┐
│                       BACKEND API                            │
│                         FastAPI                              │
│                                                              │
│ Auth │ Systems │ Metrics │ Anomalies │ Forecasts            │
│ Models │ Experiments │ XAI │ Alerts │ Reports               │
└───────┬───────────────────────┬───────────────────┬──────────┘
        │                       │                   │
        ▼                       ▼                   ▼
┌───────────────┐       ┌────────────────┐  ┌─────────────────┐
│ PostgreSQL +  │       │  ML ENGINE     │  │ Prometheus      │
│ TimescaleDB   │       │                │  │                 │
│               │       │ Classical ML   │  │ System Metrics  │
│ Metadata      │       │ Autoencoder    │  │                 │
│ Time Series   │       │ LSTM-AE        │  │                 │
│ Experiments   │       │ Forecasting    │  │                 │
│ Models        │       │ XAI / SHAP     │  │                 │
└───────────────┘       └────────────────┘  └────────┬────────┘
                                                     │
                                                     ▼
                                      ┌─────────────────────────┐
                                      │ Metric Exporters / Agent│
                                      │                         │
                                      │ Node Exporter           │
                                      │ Windows Exporter        │
                                      │ Custom Collector        │
                                      └───────────┬─────────────┘
                                                  │
                                                  ▼
                                      ┌─────────────────────────┐
                                      │ Computer / Server       │
                                      │                         │
                                      │ CPU │ RAM │ Disk │ Net │
                                      │ Load │ Processes │ etc.│
                                      └─────────────────────────┘
```

---

# 4. Architectural Layers

CSAP is organized into the following logical layers:

```text
Presentation Layer
        ↓
API / Application Layer
        ↓
Domain / Research Layer
        ↓
ML / Analytics Layer
        ↓
Data Layer
        ↓
Infrastructure Layer
```

Each layer must have a clearly defined responsibility.

---

# 5. Presentation Layer

## Technology

* React
* TypeScript
* Vite
* Tailwind CSS
* Apache ECharts

## Responsibilities

The frontend is responsible for:

* displaying system status;
* displaying real-time metrics;
* visualizing historical time series;
* displaying detected anomalies;
* displaying anomaly severity;
* displaying forecasts;
* displaying future anomaly risk;
* displaying explanations;
* configuring experiments;
* viewing model performance;
* launching controlled fault-injection experiments;
* generating and downloading reports.

The frontend must not contain scientific model logic.

ML calculations must be performed by the backend/ML engine.

---

# 6. Backend API Layer

## Technology

* Python 3.12+
* FastAPI
* Pydantic
* SQLAlchemy
* Alembic

## Responsibilities

The backend provides the central application API.

Main domains:

```text
/api/v1/auth
/api/v1/systems
/api/v1/metrics
/api/v1/anomalies
/api/v1/forecasts
/api/v1/models
/api/v1/experiments
/api/v1/xai
/api/v1/alerts
/api/v1/reports
/api/v1/fault-injection
```

The API layer must:

* validate requests;
* authorize users;
* coordinate application services;
* retrieve data;
* invoke ML services;
* store results;
* return structured responses.

The API must not contain large ML algorithms directly inside route handlers.

Route handlers should delegate work to application/domain services.

---

# 7. Monitoring and Data Collection Layer

## Components

* Prometheus
* Node Exporter
* Windows Exporter
* Custom Collector/Agent

## Main Metrics

Initial metrics include:

### CPU

* CPU utilization;
* per-core utilization;
* load average where available.

### Memory

* total memory;
* used memory;
* available memory;
* memory utilization.

### Disk

* disk utilization;
* read throughput;
* write throughput;
* IOPS where available.

### Network

* received bytes;
* transmitted bytes;
* packet statistics;
* network errors.

### System

* process count;
* system uptime;
* system load;
* selected process-level metrics.

Additional metrics may be introduced later.

---

# 8. Data Flow

The standard metric flow is:

```text
Computer
   ↓
Exporter / Agent
   ↓
Prometheus
   ↓
Data Ingestion Service
   ↓
Validation
   ↓
Normalization
   ↓
PostgreSQL / TimescaleDB
   ↓
Feature Engineering
   ↓
ML Engine
   ↓
Anomaly Score
   ↓
Adaptive Threshold
   ↓
Anomaly Decision
   ↓
Severity
   ↓
Forecasting
   ↓
Future Anomaly Risk
   ↓
XAI
   ↓
API
   ↓
Dashboard
```

Every stage must preserve timestamp information.

---

# 9. Time-Series Data Model

The fundamental observation is represented as:

$$
X_t =
[
CPU_t,
RAM_t,
Disk_t,
Network_t,
Load_t,
...
]
$$

where:

* \(t\) is the timestamp;
* each component represents a system metric.

The system must preserve:

* timestamp;
* system identifier;
* metric identifier;
* metric value;
* source;
* collection status;
* optional labels.

The architecture must support both univariate and multivariate analysis.

---

# 10. Data Processing Pipeline

Before ML processing, raw metrics pass through:

```text
Raw Data
   ↓
Validation
   ↓
Missing-value handling
   ↓
Duplicate detection
   ↓
Timestamp alignment
   ↓
Outlier-aware preprocessing
   ↓
Normalization / Standardization
   ↓
Feature construction
   ↓
Sliding-window generation
   ↓
Model input
```

Important rule:

**Preprocessing parameters must be fitted only on the training data when required.**

The system must prevent information from validation/test/future periods leaking into training.

---

# 11. ML Engine

The ML engine is an independent research component.

It must use an extensible interface.

Conceptual abstraction:

```python
BaseAnomalyDetector
├── IsolationForestDetector
├── LOFDetector
├── OneClassSVMDetector
├── PCAAnomalyDetector
├── AutoencoderDetector
└── LSTMAutoencoderDetector
```

Each detector should provide a consistent interface for:

* training;
* prediction;
* anomaly score calculation;
* model serialization;
* model metadata;
* evaluation.

Example conceptual interface:

```text
fit(training_data)
predict(data)
score(data)
save(path)
load(path)
get_metadata()
```

---

# 12. Classical ML Layer

The first experimental baseline should include:

### Isolation Forest

Used as an unsupervised anomaly-detection baseline.

### Local Outlier Factor

Used for local-density-based anomaly detection.

### One-Class SVM

Used as a boundary-based unsupervised detector.

### PCA-based Detection

Used as a dimensionality-reduction and reconstruction-based baseline.

These algorithms are primarily intended for baseline comparison.

The system must store:

* algorithm;
* parameters;
* dataset;
* training period;
* test period;
* random seed;
* preprocessing configuration;
* evaluation metrics;
* model version.

---

# 13. Deep Learning Layer

The main research direction is a sequence-based model.

Primary candidate:

**LSTM Autoencoder**

Architecture:

```text
Input Sequence
      ↓
LSTM Encoder
      ↓
Latent Representation
      ↓
LSTM Decoder
      ↓
Reconstructed Sequence
```

For a time window:

$$
X_{t-w+1:t}
$$

the model generates:

$$
\hat{X}_{t-w+1:t}
$$

The reconstruction error is used to calculate an anomaly score.

For example:

$$
A_t =
\frac{1}{n}
\sum_{i=1}^{n}
(x_{t,i}-\hat{x}_{t,i})^2
$$

The exact scoring formulation must remain configurable and experimentally validated.

---

# 14. Anomaly Detection Pipeline

The main detection pipeline is:

```text
Time-Series Window
        ↓
Preprocessing
        ↓
LSTM Autoencoder
        ↓
Reconstruction
        ↓
Reconstruction Error
        ↓
Anomaly Score
        ↓
Adaptive Threshold
        ↓
Anomaly Decision
```

Output:

```text
AnomalyResult
├── timestamp
├── system_id
├── model_id
├── anomaly_score
├── threshold
├── is_anomaly
├── severity
└── contributing_metrics
```

---

# 15. Adaptive Threshold

The system must support dynamic threshold calculation.

Initial configurable formulation:

$$
T_t = \mu_t + k\sigma_t
$$

where:

* \(\mu_t\) is the local mean of anomaly scores;
* \(\sigma_t\) is the local standard deviation;
* \(k\) is a configurable coefficient.

This formulation is a baseline, not a predetermined optimal method.

Alternative thresholding strategies must be implementable without changing the detector architecture.

Possible future methods:

* percentile-based threshold;
* rolling quantile;
* robust MAD-based threshold;
* exponentially weighted statistics;
* validation-based threshold;
* threshold optimization based on evaluation criteria.

---

# 16. Severity Calculation

Each detected anomaly should have a severity level.

Initial conceptual levels:

```text
NORMAL
LOW
MEDIUM
HIGH
CRITICAL
```

Severity should be calculated from measurable information such as:

* anomaly score;
* distance from threshold;
* persistence;
* number of affected metrics;
* forecasted risk.

Severity rules must be configurable.

The system must not treat severity labels as scientifically validated until evaluated experimentally.

---

# 17. Forecasting Layer

Forecasting is a separate but connected research component.

Input:

$$
X_{t-w:t}
$$

Output:

$$
\hat{X}_{t+1:t+h}
$$

where:

* \(w\) = historical window;
* \(h\) = forecasting horizon.

Possible models:

* LSTM;
* GRU;
* baseline statistical models;
* other sequence models introduced later.

The forecasting module must remain independent from the anomaly detector so that different forecasting models can be compared.

---

# 18. Future Anomaly Risk

The platform must support detection of potential future anomalies.

Conceptual pipeline:

```text
Historical Metrics
       ↓
Forecasting Model
       ↓
Future Metric Values
       ↓
Anomaly Detection Logic
       ↓
Future Anomaly Score
       ↓
Future Threshold Comparison
       ↓
Future Risk
```

Example output:

```text
FutureRisk
├── forecast_timestamp
├── predicted_metrics
├── predicted_anomaly_score
├── threshold
├── risk_level
└── affected_metrics
```

The system must clearly distinguish:

* observed anomaly;
* predicted metric value;
* predicted anomaly risk.

A forecast is not equivalent to a confirmed future event.

---

# 19. Explainability Layer

The platform should provide model explanations where technically appropriate.

Primary technology:

**SHAP**

The XAI layer may provide:

* important features;
* metric contribution;
* feature importance;
* explanation values;
* local explanations for individual anomalies.

Example:

```text
Anomaly
   ↓
Model Score
   ↓
SHAP
   ↓
Metric Contributions
   ↓
Explanation
```

The interface should use terminology such as:

> “Potential contributing factors”

rather than claiming causal relationships unless causal evidence exists.

---

# 20. Fault Injection

Fault injection is an experimental component.

Its purpose is to create controlled abnormal conditions for evaluating detection methods.

Examples may include controlled:

* CPU load;
* memory pressure;
* disk I/O load;
* network load.

Fault injection must have:

```text
Experiment
├── start_time
├── end_time
├── target_system
├── fault_type
├── intensity
├── duration
└── experiment_id
```

Every injected fault must be explicitly recorded.

The system must never silently alter the monitored environment.

Safety limits must be implemented.

---

# 21. Experiment Management

Experiments are first-class entities.

Each experiment should contain:

```text
Experiment
├── experiment_id
├── name
├── description
├── dataset_version
├── model_version
├── parameters
├── preprocessing_version
├── random_seed
├── training_period
├── validation_period
├── test_period
├── Git commit
├── environment
├── metrics
└── artifacts
```

This is required for reproducibility.

---

# 22. Model Registry

The platform must maintain model metadata.

Example:

```text
Model
├── model_id
├── name
├── type
├── version
├── framework
├── parameters
├── dataset_version
├── training_timestamp
├── artifact_location
├── metrics
└── status
```

Possible statuses:

```text
TRAINING
READY
ACTIVE
ARCHIVED
FAILED
```

MLflow may be used as the experiment/model tracking layer.

---

# 23. Database Architecture

Primary database:

**PostgreSQL + TimescaleDB**

Logical data domains:

```text
users
systems
metric_definitions
metric_samples
anomaly_results
forecast_results
models
experiments
experiment_runs
fault_injections
alerts
reports
audit_logs
```

Time-series data should be optimized for timestamp-based queries.

Relational metadata should remain normalized and strongly typed.

---

# 24. API Architecture

The backend should use versioned APIs.

Base:

```text
/api/v1
```

Example resources:

```text
GET    /systems
POST   /systems

GET    /systems/{id}/metrics
GET    /systems/{id}/anomalies
GET    /systems/{id}/forecasts

POST   /models/train
GET    /models
GET    /models/{id}

POST   /experiments
GET    /experiments
GET    /experiments/{id}

POST   /fault-injection/start
POST   /fault-injection/stop

GET    /reports/{id}
```

Exact endpoint contracts must be defined separately in:

`docs/API_SPEC.md`

---

# 25. Authentication and Authorization

Initial roles:

```text
ADMIN
RESEARCHER
VIEWER
```

Conceptual permissions:

| Capability        | Admin | Researcher | Viewer |
| ----------------- | ----: | ---------: | -----: |
| View dashboard    |   Yes |        Yes |    Yes |
| View metrics      |   Yes |        Yes |    Yes |
| Run experiments   |   Yes |        Yes |     No |
| Train models      |   Yes |        Yes |     No |
| Configure systems |   Yes |        Yes |     No |
| Fault injection   |   Yes |        Yes |     No |
| Manage users      |   Yes |         No |     No |
| View reports      |   Yes |        Yes |    Yes |

Authentication implementation must be defined in the security specification.

---

# 26. Alerting

Alerting is downstream of anomaly detection.

```text
Anomaly
   ↓
Severity
   ↓
Alert Rule
   ↓
Notification
```

Potential notification channels:

* in-app notification;
* email;
* Telegram;
* webhook.

Alerts must contain:

* system;
* timestamp;
* anomaly score;
* threshold;
* severity;
* affected metrics;
* model;
* explanation summary.

---

# 27. Reporting

The reporting module should support scientific and operational reports.

Possible report types:

### Experiment Report

Contains:

* experiment configuration;
* dataset;
* model;
* parameters;
* metrics;
* charts;
* results.

### Model Comparison Report

Contains:

* algorithms;
* evaluation metrics;
* training/test information;
* parameter configurations;
* comparison tables.

### Anomaly Report

Contains:

* detected anomalies;
* timestamps;
* severity;
* affected metrics;
* explanations.

Reports must distinguish measured results from interpretations.

---

# 28. Deployment Architecture

Initial deployment should use Docker Compose.

Conceptual deployment:

```text
                 ┌─────────────────────┐
                 │      Browser        │
                 └──────────┬──────────┘
                            │
                            ▼
                 ┌─────────────────────┐
                 │      Frontend       │
                 │       React         │
                 └──────────┬──────────┘
                            │
                            ▼
                 ┌─────────────────────┐
                 │      FastAPI        │
                 │      Backend        │
                 └──────┬─────┬────────┘
                        │     │
             ┌──────────┘     └─────────────┐
             ▼                              ▼
    ┌────────────────┐             ┌────────────────┐
    │ PostgreSQL +   │             │   ML Engine    │
    │ TimescaleDB    │             │    PyTorch     │
    └────────────────┘             └────────────────┘

    ┌────────────────┐
    │  Prometheus    │
    └───────┬────────┘
            ▼
    ┌────────────────┐
    │ Exporters      │
    └────────────────┘
```

Production deployment may later use:

* Nginx;
* dedicated ML workers;
* Redis;
* Celery;
* Kubernetes.

These are not mandatory for the first research release.

---

# 29. Separation of Responsibilities

The following separation is mandatory:

```text
Frontend
    → presentation only

FastAPI
    → API and application orchestration

Domain services
    → business/research workflows

ML Engine
    → model training/inference

Database
    → persistent storage

Prometheus
    → metric collection/storage for monitoring

TimescaleDB
    → research-grade historical time-series storage

MLflow
    → experiment/model tracking
```

No layer should unnecessarily absorb responsibilities belonging to another layer.

---

# 30. Scientific Reproducibility

Every research result must be traceable to:

```text
Dataset
+
Preprocessing
+
Model
+
Parameters
+
Random Seed
+
Environment
+
Git Commit
+
Evaluation Procedure
```

The platform must make it possible to answer:

> “Which data, preprocessing method, model, parameters, and code version produced this result?”

---

# 31. Data Leakage Prevention

The architecture must explicitly prevent:

* future data entering training;
* test data influencing preprocessing;
* threshold calculation using test labels when inappropriate;
* forecast horizon leaking into historical features;
* experiment metadata accidentally entering model features.

Train/validation/test separation must be time-aware for time-series experiments.

---

# 32. Observability

The platform itself must be observable.

Backend should provide:

* structured logs;
* request IDs;
* error logs;
* health endpoints;
* readiness status;
* basic performance metrics.

Recommended endpoints:

```text
/health
/ready
```

ML jobs should log:

* model;
* dataset;
* experiment;
* duration;
* status;
* errors.

---

# 33. Error Handling

Errors should be represented using consistent API responses.

Conceptual structure:

```json
{
  "error": {
    "code": "MODEL_NOT_READY",
    "message": "Requested model is not ready for inference.",
    "request_id": "..."
  }
}
```

Internal stack traces must not be exposed to normal users.

---

# 34. Testing Architecture

Testing must exist at multiple levels.

```text
Unit Tests
    ↓
Integration Tests
    ↓
API Tests
    ↓
ML Pipeline Tests
    ↓
End-to-End Tests
```

ML-specific tests must verify:

* deterministic preprocessing;
* correct window generation;
* no temporal leakage;
* model input dimensions;
* anomaly-score calculation;
* threshold calculation;
* forecast output shape;
* experiment reproducibility.

---

# 35. Security Principles

The platform must:

* validate all external input;
* use secure authentication;
* protect credentials using environment variables/secrets;
* avoid storing plaintext passwords;
* restrict dangerous fault-injection operations;
* validate file uploads;
* protect administrative endpoints;
* log security-sensitive operations;
* avoid exposing internal infrastructure details.

---

# 36. Scalability Strategy

The first version should prioritize correctness and reproducibility over premature scalability.

Initial architecture:

```text
Modular Monolith + Separate ML Module
```

rather than immediately introducing many microservices.

Services can be separated later if required by measured bottlenecks.

Possible future separation:

```text
API Service
Metric Ingestion Service
ML Training Service
ML Inference Service
Forecasting Service
Notification Service
Report Service
```

The initial architecture must therefore use clear module boundaries even when deployed as fewer containers.

---

# 37. Critical Runtime Flows

## 37.1 Real-Time Monitoring

```text
System
 ↓
Exporter
 ↓
Prometheus
 ↓
Backend
 ↓
Frontend
```

---

## 37.2 Anomaly Detection

```text
Historical Metrics
 ↓
Preprocessing
 ↓
Sliding Window
 ↓
LSTM Autoencoder
 ↓
Reconstruction Error
 ↓
Adaptive Threshold
 ↓
Anomaly
 ↓
Severity
 ↓
Dashboard
```

---

## 37.3 Forecasting

```text
Historical Window
 ↓
Forecasting Model
 ↓
Future Metrics
 ↓
Visualization
```

---

## 37.4 Future Anomaly Prediction

```text
Historical Window
 ↓
Forecasting Model
 ↓
Future Metrics
 ↓
Anomaly Scoring
 ↓
Threshold
 ↓
Future Risk
```

---

## 37.5 Explainability

```text
Detected Anomaly
 ↓
Model / Feature Representation
 ↓
SHAP
 ↓
Metric Contributions
 ↓
Explanation
```

---

## 37.6 Scientific Experiment

```text
Experiment Configuration
 ↓
Dataset Selection
 ↓
Preprocessing
 ↓
Model Training
 ↓
Inference
 ↓
Evaluation
 ↓
Result Storage
 ↓
Model / Experiment Registry
 ↓
Report
```

---

# 38. Architecture Decision Principles

The following principles are mandatory:

1. Prefer modularity over unnecessary microservices.
2. Prefer reproducibility over convenience.
3. Prefer explicit data contracts.
4. Prefer configuration over hard-coded research parameters.
5. Never mix future data into historical features.
6. Never hard-code ML results into the application.
7. Never fabricate scientific results.
8. Keep ML models replaceable.
9. Keep database access separate from business logic.
10. Keep frontend independent from model implementation.
11. Document significant architectural decisions.
12. Update this document whenever architecture changes.

---

# 39. Architecture Documentation Set

The complete architecture documentation should eventually include:

```text
docs/
├── PROJECT_SPEC.md
├── ARCHITECTURE.md
├── DATABASE_SPEC.md
├── API_SPEC.md
├── ML_SPEC.md
├── FORECASTING_SPEC.md
├── XAI_SPEC.md
├── EXPERIMENT_SPEC.md
├── TESTING_SPEC.md
├── ROADMAP.md
└── adr/
    ├── 0001-initial-architecture.md
    ├── 0002-timescaledb.md
    ├── 0003-lstm-autoencoder.md
    └── ...
```

Architecture diagrams should be kept close to the repository and updated when the actual system changes.

---

# 40. Definition of Architectural Completion

The architecture is considered sufficiently defined when:

* all major components are identified;
* responsibilities are separated;
* data flow is defined;
* ML integration is defined;
* forecasting integration is defined;
* XAI integration is defined;
* experiment tracking is defined;
* database responsibility is defined;
* API responsibility is defined;
* deployment structure is defined;
* testing boundaries are defined;
* reproducibility requirements are defined;
* security boundaries are defined.

Implementation may begin only after the architecture is reviewed against:

* `AGENTS.md`;
* `PROJECT_SPEC.md`;
* `ROADMAP.md`.

---

# 41. Important Implementation Constraint

**Do not implement the complete CSAP platform in a single step.**

Codex must implement the system according to the phases defined in `ROADMAP.md`.

Each phase must follow:

```text
Specification
    ↓
Implementation
    ↓
Tests
    ↓
Review
    ↓
Documentation Update
    ↓
Commit
    ↓
Next Phase
```

No later research module should be implemented before its required infrastructure is stable.

---

# 42. Current Architectural Target

The initial target architecture is:

```text
React
  ↓
FastAPI
  ↓
Application Services
  ↓
PostgreSQL + TimescaleDB
  ↘
   ML Engine
      ├── Classical ML
      ├── Autoencoder
      ├── LSTM Autoencoder
      ├── Adaptive Threshold
      ├── Forecasting
      └── XAI

Prometheus
  ↓
Exporters
  ↓
Computer Systems
```

This architecture is intentionally modular so that the platform can evolve from a basic monitoring system into a complete research platform for anomaly detection and forecasting.
