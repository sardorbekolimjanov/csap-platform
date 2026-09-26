# CSAP — Codex Development Instructions

## 1. Project Identity

Project name:

**CSAP — Computer System Anomaly Prediction Platform**

Project purpose:

CSAP is a research-oriented and production-capable software platform for monitoring computer-system performance metrics, detecting anomalous behavior using machine learning and deep learning, forecasting future anomalous states, explaining detected anomalies, and evaluating machine-learning methods through reproducible experiments.

The project is directly associated with the following master's dissertation:

> “Kompyuter tizimlarining ishlash ko‘rsatkichlaridagi anomaliyalarni mashinaviy o‘qitish asosida aniqlash va prognozlash usulini ishlab chiqish.”

The implementation must support the scientific objectives of the dissertation and must not become merely a conventional monitoring dashboard.

---

# 2. Primary Scientific Objectives

CSAP must support:

1. Collection of computer-system performance metrics.
2. Storage of multivariate time-series data.
3. Data preprocessing and feature engineering.
4. Classical machine-learning-based anomaly detection.
5. Deep-learning-based anomaly detection.
6. LSTM Autoencoder-based temporal anomaly detection.
7. Adaptive anomaly thresholding.
8. Future metric forecasting.
9. Future anomaly-risk forecasting.
10. Anomaly severity estimation.
11. Explainable anomaly detection.
12. Identification of potential contributing factors.
13. Controlled fault injection for experiments.
14. Reproducible ML experiments.
15. Comparison of different algorithms.
16. Scientific evaluation using appropriate metrics.
17. Visualization and reporting of experimental results.

---

# 3. Core Technology Stack

Use the following technologies unless a documented architectural reason requires otherwise.

## Backend

* Python 3.12+
* FastAPI
* Pydantic
* SQLAlchemy
* Alembic

## Frontend

* React
* TypeScript
* Vite
* Tailwind CSS
* Apache ECharts or another well-maintained charting library

## Database

* PostgreSQL
* TimescaleDB

## Monitoring

* Prometheus
* Node Exporter for Linux
* Windows Exporter for Windows

## Machine Learning

* NumPy
* Pandas
* SciPy
* scikit-learn

## Deep Learning

* PyTorch

## Experiment Tracking

* MLflow

## Explainable AI

* SHAP

## Infrastructure

* Docker
* Docker Compose
* Nginx where appropriate
* Git

## Optional infrastructure

Use Kafka, Celery, Redis, Kubernetes, or other infrastructure only when there is a documented requirement. Do not introduce unnecessary complexity.

---

# 4. Architecture Principles

The project must follow these principles:

1. Modular architecture.
2. Separation of concerns.
3. Clear separation between frontend, backend, ML, data collection, and persistence.
4. ML logic must not be tightly coupled to HTTP/API logic.
5. Database access must be isolated from business logic.
6. Configuration must be externalized.
7. Secrets must never be hardcoded.
8. Every important feature must have automated tests.
9. ML experiments must be reproducible.
10. New models must be addable without rewriting the entire ML subsystem.
11. APIs must be versionable.
12. Breaking changes must be documented.
13. Code must remain understandable to a master's-level research project developer.
14. Avoid premature microservice decomposition.
15. Prefer a modular monolith during the initial development stages.

---

# 5. Scientific Integrity Rules

These rules are mandatory.

## Never fabricate:

* experimental results;
* F1 scores;
* Precision;
* Recall;
* ROC-AUC;
* PR-AUC;
* false-positive rates;
* detection latency;
* datasets;
* citations;
* DOI values;
* benchmark results;
* model performance;
* scientific conclusions.

If an experiment has not actually been executed, use:

* `null`;
* `not_evaluated`;
* `pending`;
* or an explicitly labelled placeholder.

Never generate fake scientific evidence merely to make the UI look complete.

---

# 6. ML Architecture Rules

The ML subsystem must use a common abstraction for anomaly detectors.

Conceptually:

```text
BaseAnomalyDetector
├── IsolationForestDetector
├── LOFDetector
├── OneClassSVMDetector
├── PCAAnomalyDetector
├── AutoencoderDetector
└── LSTMAutoencoderDetector
```

Models must expose consistent operations where applicable:

* fit
* predict
* score
* save
* load
* evaluate

Model-specific functionality must remain inside the corresponding model implementation.

---

# 7. Main Research Model

The principal research direction is:

```text
Multivariate Time Series
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
Anomaly Decision
```

Forecasting:

```text
Historical Time Series
        ↓
Temporal Forecasting Model
        ↓
Future Metrics
        ↓
Future Anomaly Risk
        ↓
Early Warning
```

Explainability:

```text
Detected Anomaly
        ↓
Feature Contribution
        ↓
Potential Contributing Factors
```

---

# 8. Data Integrity

Time-series data must preserve chronological order.

Do not use random train/test splitting for temporal experiments unless there is a scientifically justified reason.

Prefer:

```text
Past data      → Training
Later data     → Validation
Future data    → Test
```

Avoid data leakage.

Preprocessing operations such as scaling must be fitted on training data and then applied to validation/test data.

---

# 9. Configuration

Use environment variables and configuration files.

Never hardcode:

* database passwords;
* JWT secrets;
* API keys;
* Telegram tokens;
* credentials;
* production endpoints.

Use `.env.example`.

Never commit real `.env` files containing secrets.

---

# 10. Testing Requirements

Each significant feature should have:

* unit tests;
* integration tests where appropriate;
* API tests for backend endpoints;
* ML tests for preprocessing/model behavior;
* regression tests for previously fixed bugs.

Before declaring a feature complete:

1. Run relevant tests.
2. Run lint/type checks where configured.
3. Verify migrations.
4. Verify Docker startup where applicable.
5. Report failures explicitly.

Never claim that tests passed unless they were actually executed.

---

# 11. Code Quality

Prefer:

* clear naming;
* small functions;
* explicit types;
* meaningful docstrings;
* structured logging;
* deterministic behavior where possible;
* dependency injection where appropriate;
* reusable components.

Avoid:

* unnecessary abstractions;
* duplicated logic;
* global mutable state;
* magic numbers;
* giant files;
* giant functions;
* unused dependencies;
* dead code.

---

# 12. API Rules

API endpoints must:

* use clear REST semantics;
* validate input with Pydantic;
* return predictable schemas;
* provide appropriate HTTP status codes;
* handle errors consistently;
* avoid leaking internal exceptions;
* support API versioning when appropriate.

Use a structure such as:

```text
/api/v1/
```

---

# 13. Database Rules

Database schema changes must use migrations.

Use Alembic.

Never manually modify production schema without a migration.

Time-series tables should be designed with TimescaleDB capabilities in mind.

Indexes must be added based on actual query requirements.

Do not over-index tables without justification.

---

# 14. Observability

The CSAP application itself should eventually expose:

* application logs;
* health endpoint;
* readiness endpoint;
* basic service metrics;
* model execution status;
* experiment status.

At minimum:

```text
/health
/ready
```

must be available once the backend foundation is implemented.

---

# 15. Documentation Rules

Documentation is part of the implementation.

Update relevant files under:

```text
docs/
```

when architecture, API, database, ML methodology, or experiment behavior changes.

Do not silently introduce architectural changes that contradict the documentation.

If implementation and documentation conflict:

1. Identify the conflict.
2. Determine the intended behavior.
3. Update the specification if the change is intentional.
4. Then implement the change.

---

# 16. Development Workflow

For every major task:

```text
Read relevant specifications
        ↓
Inspect existing implementation
        ↓
Plan changes
        ↓
Implement
        ↓
Write/update tests
        ↓
Run tests
        ↓
Review implementation
        ↓
Update documentation
        ↓
Report result
```

Do not rewrite unrelated parts of the project.

Do not introduce large architectural changes without explicit justification.

---

# 17. Git Discipline

Use meaningful commits.

Examples:

```text
feat: add metrics ingestion service
feat: implement isolation forest detector
feat: add lstm autoencoder
feat: implement adaptive threshold
fix: correct anomaly score calculation
test: add anomaly detector tests
docs: update ML specification
refactor: separate model registry service
```

Avoid vague commits such as:

```text
update
changes
fix stuff
new code
```

---

# 18. Definition of Done

A feature is not complete merely because code exists.

A feature is complete only when:

* implementation exists;
* relevant tests exist;
* tests pass;
* error handling exists;
* configuration is documented;
* API/schema changes are documented;
* no secrets are committed;
* scientific assumptions are documented;
* reproducibility requirements are satisfied where relevant.

---

# 19. Important Development Constraint

Do not attempt to implement the entire CSAP platform in one operation.

Follow the roadmap defined in:

```text
docs/ROADMAP.md
```

Implement one phase at a time.

After each phase:

1. verify;
2. test;
3. review;
4. document;
5. only then continue.

---

# 20. Priority Order

When requirements conflict, use this priority:

1. Scientific correctness
2. Data integrity
3. Security
4. Architectural consistency
5. Testability
6. Maintainability
7. Performance
8. UI convenience

Do not sacrifice scientific correctness merely to make the application appear more complete.
