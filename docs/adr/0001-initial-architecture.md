# ADR-0001: Initial CSAP Architecture

## Status

**Accepted**

## Date

2026-09-26

## Decision Type

Architecture

---

# 1. Context

CSAP — **Computer System Anomaly Prediction Platform** — is a research-oriented platform being developed in support of the master's dissertation:

> **“Kompyuter tizimlarining ishlash ko‘rsatkichlaridagi anomaliyalarni mashinaviy o‘qitish asosida aniqlash va prognozlash usulini ishlab chiqish.”**

The platform must combine:

* computer-system monitoring;
* time-series collection;
* anomaly detection;
* classical machine-learning baselines;
* deep-learning models;
* LSTM Autoencoder;
* adaptive thresholding;
* forecasting;
* future anomaly-risk estimation;
* explainability;
* controlled fault injection;
* experiment tracking;
* scientific evaluation;
* reporting.

The architecture must support research reproducibility while remaining practical enough to implement and maintain as a master's-level software project.

---

# 2. Decision

The initial CSAP architecture will use a:

> **Modular Monolith with a dedicated ML/Analytics layer**

The initial technology stack is:

| Layer               | Technology                       |
| ------------------- | -------------------------------- |
| Frontend            | React + TypeScript + Vite        |
| UI                  | Tailwind CSS                     |
| Visualization       | Apache ECharts                   |
| Backend             | Python + FastAPI                 |
| Validation          | Pydantic                         |
| ORM                 | SQLAlchemy                       |
| Migrations          | Alembic                          |
| Database            | PostgreSQL                       |
| Time-series         | TimescaleDB                      |
| Monitoring          | Prometheus                       |
| Exporters           | Node Exporter / Windows Exporter |
| Classical ML        | scikit-learn                     |
| Deep Learning       | PyTorch                          |
| Explainability      | SHAP                             |
| Experiment Tracking | MLflow                           |
| Deployment          | Docker Compose                   |
| Version Control     | Git                              |

---

# 3. Architectural Structure

The system will follow:

```text
┌──────────────────────────────────────┐
│             React Frontend           │
└──────────────────┬───────────────────┘
                   │ REST API
                   ▼
┌──────────────────────────────────────┐
│              FastAPI                 │
│        Application/API Layer         │
└──────────────┬───────────────┬───────┘
               │               │
               ▼               ▼
      ┌────────────────┐ ┌───────────────┐
      │ PostgreSQL +   │ │ ML / Analytics │
      │ TimescaleDB    │ │ Engine         │
      └────────────────┘ └───────────────┘
                               │
                ┌──────────────┼───────────────┐
                ▼              ▼               ▼
          Classical ML   Deep Learning    Forecasting/XAI

Prometheus
    ↓
Exporters
    ↓
Computer Systems
```

---

# 4. Why Modular Monolith?

The initial implementation will not use a large microservice architecture.

The reasons are:

1. The project is primarily a research platform.
2. Early development requires rapid iteration.
3. A distributed architecture would introduce unnecessary operational complexity.
4. Debugging is easier when core components are initially integrated.
5. Scientific reproducibility is easier to maintain with fewer independently deployed services.
6. The system can later be decomposed if measured requirements justify it.

The codebase must nevertheless maintain strong module boundaries.

---

# 5. Why FastAPI?

FastAPI is selected for the backend because the platform requires:

* Python-native ML integration;
* typed request/response schemas;
* asynchronous API capabilities;
* automatic OpenAPI documentation;
* straightforward integration with NumPy, Pandas, scikit-learn and PyTorch.

The backend will therefore act as the bridge between the frontend, database, monitoring infrastructure, and ML engine.

---

# 6. Why React?

React is selected for the frontend because CSAP requires an interactive research dashboard with:

* real-time metric visualization;
* historical time-series charts;
* anomaly visualization;
* forecast visualization;
* experiment configuration;
* model information;
* explainability views;
* report interfaces.

The frontend must remain independent from ML implementation details.

---

# 7. Why PostgreSQL + TimescaleDB?

PostgreSQL is selected as the primary relational database.

TimescaleDB is used for high-volume time-series data where appropriate.

This combination allows the platform to store both:

```text
Relational data
    ↓
users
systems
models
experiments
reports
```

and:

```text
Time-series data
    ↓
metric_samples
forecast_values
potential future time-series results
```

The project will not use a separate time-series database unless future measured requirements justify it.

---

# 8. Why Prometheus?

Prometheus is selected for operational monitoring and metric collection because CSAP needs:

* system metric collection;
* time-series querying;
* exporter integration;
* real-time monitoring.

Prometheus and TimescaleDB have different responsibilities.

```text
Prometheus
→ operational monitoring / collection

TimescaleDB
→ research-oriented historical storage
```

The architecture must avoid treating them as interchangeable databases.

---

# 9. Why scikit-learn?

scikit-learn provides standardized implementations for baseline anomaly-detection algorithms such as:

* Isolation Forest;
* Local Outlier Factor;
* One-Class SVM;
* PCA-based methods.

These models are required for comparative experiments.

The implementation must expose them through a common anomaly-detector abstraction.

---

# 10. Why PyTorch?

PyTorch is selected for the deep-learning research layer.

The primary research model is:

> **LSTM Autoencoder**

PyTorch also allows future implementation of:

* LSTM forecasting;
* GRU;
* other sequence models;
* custom neural architectures.

---

# 11. Why MLflow?

MLflow is selected for experiment and model tracking.

Research runs should preserve:

* parameters;
* metrics;
* artifacts;
* model versions;
* experiment identity.

The system database will continue to store application-level experiment metadata.

MLflow is complementary rather than a replacement for the application database.

---

# 12. Why SHAP?

SHAP is selected as the primary explainability technology.

The platform needs to provide information about which metrics contributed to a model output.

However:

> A model contribution is not automatically a causal explanation.

Therefore, the interface must use scientifically careful terminology such as:

* feature contribution;
* model contribution;
* potential contributing factor.

The platform must not claim causality without an appropriate causal methodology.

---

# 13. Scientific Reproducibility

Every important experiment must be traceable to:

```text
Dataset
+
Dataset Version
+
Preprocessing
+
Model
+
Model Version
+
Parameters
+
Random Seed
+
Train/Validation/Test Period
+
Git Commit
+
Environment
```

No experimental result may be presented as measured unless it was actually generated.

---

# 14. Time-Series Integrity

Because the research concerns temporal data, the architecture must explicitly prevent temporal leakage.

The system must ensure:

```text
Past
 ↓
Training
 ↓
Validation
 ↓
Testing
 ↓
Future
```

Future observations must not accidentally influence historical training or preprocessing.

Time-aware splitting must be used for the primary research experiments.

---

# 15. Main Research Pipeline

The principal research pipeline will be:

```text
System Metrics
      ↓
Data Collection
      ↓
Time-Series Storage
      ↓
Preprocessing
      ↓
Sliding Window
      ↓
LSTM Autoencoder
      ↓
Reconstruction Error
      ↓
Anomaly Score
      ↓
Adaptive Threshold
      ↓
Anomaly Detection
      ↓
Severity
      ↓
Forecasting
      ↓
Future Anomaly Risk
      ↓
Explainability
```

This pipeline forms the technical foundation of the dissertation-oriented platform.

---

# 16. Alternatives Considered

## 16.1 Microservices from the Beginning

Rejected for the initial version.

Reason:

* higher deployment complexity;
* more network boundaries;
* more infrastructure;
* more difficult debugging;
* unnecessary for the initial research scope.

Microservices may be introduced later if measurable requirements justify them.

---

## 16.2 MongoDB as the Primary Database

Not selected.

The platform contains strongly relational entities:

* users;
* systems;
* models;
* experiments;
* runs;
* anomalies;
* reports.

PostgreSQL provides stronger relational integrity while still supporting flexible JSONB fields.

---

## 16.3 InfluxDB as the Primary Database

Not selected for the initial architecture.

The system requires a combination of relational metadata and time-series data.

PostgreSQL + TimescaleDB provides a unified database environment for the initial implementation.

---

## 16.4 Kafka from the Beginning

Not selected.

Kafka may become useful for large-scale streaming, but the initial research platform does not require its operational complexity.

It may be introduced if actual ingestion requirements demonstrate the need.

---

## 16.5 Kubernetes from the Beginning

Not selected.

Docker Compose is sufficient for the initial research deployment.

Kubernetes may be considered later for:

* horizontal scaling;
* multi-node deployment;
* high availability;
* independent service scaling.

---

# 17. Consequences

## Positive Consequences

The selected architecture provides:

* relatively low initial complexity;
* strong Python/ML integration;
* clear module boundaries;
* reproducible experiments;
* scalable time-series storage;
* interactive visualization;
* straightforward local development;
* containerized deployment;
* future extensibility.

---

## Negative Consequences

The architecture also has limitations:

* the initial backend may contain multiple responsibilities;
* large ML training jobs may eventually require dedicated workers;
* very high ingestion volumes may require additional infrastructure;
* frontend and backend deployment scaling may initially be coupled;
* some components may eventually need to be separated.

These limitations are accepted for the initial research release.

---

# 18. Future Evolution

If actual measurements justify architectural expansion, the platform may evolve toward:

```text
API Service
     │
     ├── Ingestion Service
     ├── ML Training Service
     ├── ML Inference Service
     ├── Forecasting Service
     ├── Notification Service
     └── Reporting Service
```

Possible infrastructure additions:

```text
Redis
Celery
Kafka
Kubernetes
Nginx
Object Storage
```

These technologies must not be introduced merely for architectural complexity.

---

# 19. Decision Rules

Future architecture changes must satisfy at least one of the following:

1. measured performance bottleneck;
2. reliability requirement;
3. security requirement;
4. maintainability requirement;
5. reproducibility requirement;
6. demonstrated research requirement.

The following are insufficient reasons:

* technology popularity;
* unnecessary complexity;
* “enterprise” appearance;
* adding tools without a concrete requirement.

---

# 20. Implementation Constraint

The architecture must be implemented incrementally according to:

```text
Specification
    ↓
Foundation
    ↓
Database
    ↓
Monitoring
    ↓
Dashboard
    ↓
Classical ML
    ↓
Autoencoder
    ↓
LSTM Autoencoder
    ↓
Adaptive Threshold
    ↓
Forecasting
    ↓
Future Risk
    ↓
XAI
    ↓
Fault Injection
    ↓
Experiments
    ↓
Scientific Evaluation
```

No advanced module should bypass the required foundational layers.

---

# 21. Related Documents

This ADR is associated with:

```text
AGENTS.md
docs/PROJECT_SPEC.md
docs/ARCHITECTURE.md
docs/DATABASE_SPEC.md
docs/API_SPEC.md
docs/ROADMAP.md
```

Future architecture changes must create a new ADR rather than silently rewriting this decision.

---

# 22. Final Decision

CSAP will initially use a:

> **React + FastAPI + PostgreSQL/TimescaleDB + Prometheus + Python ML/Deep Learning + MLflow + SHAP + Docker Compose modular architecture.**

The architecture prioritizes:

**scientific reproducibility → modularity → correctness → maintainability → scalability.**
