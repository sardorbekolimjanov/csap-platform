# CSAP — API Specification

## 1. Purpose

This document defines the REST API contract for the **CSAP — Computer System Anomaly Prediction Platform**.

The API connects:

```text
React Frontend
      ↓
FastAPI
      ↓
Application Services
      ↓
Database / ML Engine / Monitoring
```

The API must provide a stable contract between the frontend and backend.

All API implementation must follow this document.

---

# 2. API Technology

Primary technologies:

* FastAPI
* Python
* Pydantic
* SQLAlchemy
* PostgreSQL
* TimescaleDB

API documentation must be automatically available through FastAPI/OpenAPI.

Development endpoints:

```text
/api/v1
```

---

# 3. Base URL

Development:

```text
http://localhost:8000
```

API base:

```text
http://localhost:8000/api/v1
```

Production URL must be configured through environment variables.

The frontend must never hard-code a production hostname.

---

# 4. API Versioning

All public application endpoints must use:

```text
/api/v1
```

Example:

```text
GET /api/v1/systems
```

Breaking API changes require a new API version.

---

# 5. Response Format

Successful responses should use predictable JSON structures.

For single resources:

```json
{
  "data": {
    "id": "uuid",
    "name": "server-01"
  }
}
```

For collections:

```json
{
  "data": [],
  "pagination": {
    "page": 1,
    "page_size": 50,
    "total": 120
  }
}
```

---

# 6. Error Format

All application errors should use a consistent structure:

```json
{
  "error": {
    "code": "RESOURCE_NOT_FOUND",
    "message": "System was not found.",
    "details": {},
    "request_id": "uuid"
  }
}
```

The API must not expose internal stack traces.

---

# 7. Standard HTTP Status Codes

Use:

```text
200 OK
201 Created
202 Accepted
204 No Content

400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
422 Unprocessable Entity
429 Too Many Requests

500 Internal Server Error
503 Service Unavailable
```

Long-running ML operations should normally return:

```text
202 Accepted
```

with a job/run identifier.

---

# 8. Request ID

Every API request should have a request identifier.

Recommended header:

```text
X-Request-ID
```

If the client does not provide one, the backend generates it.

The identifier must appear in:

* response headers;
* error responses;
* relevant application logs.

---

# 9. Authentication

Authentication endpoints:

```text
POST /api/v1/auth/login
POST /api/v1/auth/refresh
POST /api/v1/auth/logout
GET  /api/v1/auth/me
```

Authentication implementation may use JWT or another secure token mechanism.

The exact authentication mechanism must be configurable.

---

# 10. Login

```text
POST /api/v1/auth/login
```

Request:

```json
{
  "username": "researcher",
  "password": "********"
}
```

Response:

```json
{
  "data": {
    "access_token": "token",
    "refresh_token": "token",
    "token_type": "bearer",
    "expires_in": 3600
  }
}
```

Passwords must never appear in logs.

---

# 11. Current User

```text
GET /api/v1/auth/me
```

Response:

```json
{
  "data": {
    "id": "uuid",
    "username": "researcher",
    "email": "researcher@example.com",
    "full_name": "Researcher",
    "roles": [
      "RESEARCHER"
    ]
  }
}
```

---

# 12. Systems

## List Systems

```text
GET /api/v1/systems
```

Query parameters:

```text
page
page_size
search
environment
monitoring_enabled
```

Example:

```text
GET /api/v1/systems?page=1&page_size=50&environment=research
```

---

## Get System

```text
GET /api/v1/systems/{system_id}
```

---

## Create System

```text
POST /api/v1/systems
```

Request:

```json
{
  "name": "research-server-01",
  "hostname": "server01",
  "operating_system": "Linux",
  "environment": "research",
  "description": "Research monitoring server",
  "monitoring_enabled": true
}
```

Response:

```json
{
  "data": {
    "id": "uuid",
    "name": "research-server-01",
    "hostname": "server01",
    "operating_system": "Linux",
    "environment": "research",
    "monitoring_enabled": true
  }
}
```

---

## Update System

```text
PATCH /api/v1/systems/{system_id}
```

---

## Delete System

```text
DELETE /api/v1/systems/{system_id}
```

Deletion must respect research-data retention rules.

If historical data exists, soft deletion should be preferred.

---

# 13. Metrics

## List Metric Definitions

```text
GET /api/v1/metrics
```

Query parameters:

```text
system_id
category
is_enabled
```

---

## Get Metric Definition

```text
GET /api/v1/metrics/{metric_id}
```

---

## Create Metric Definition

```text
POST /api/v1/metrics
```

Request:

```json
{
  "system_id": "uuid",
  "name": "cpu_usage_percent",
  "display_name": "CPU Usage",
  "category": "cpu",
  "unit": "%",
  "description": "CPU utilization percentage",
  "source": "node_exporter",
  "collection_interval_seconds": 15,
  "is_enabled": true
}
```

---

# 14. Metric Samples

Metric samples are normally written by ingestion services rather than manually through the frontend.

Internal endpoint:

```text
POST /api/v1/metrics/ingest
```

Request:

```json
{
  "system_id": "uuid",
  "samples": [
    {
      "metric_id": "uuid",
      "timestamp": "2026-09-26T10:00:00Z",
      "value": 42.7,
      "quality": "VALID"
    }
  ]
}
```

The ingestion endpoint must support batch insertion.

---

# 15. Historical Metrics

```text
GET /api/v1/systems/{system_id}/metrics
```

Query parameters:

```text
metric_ids
start
end
interval
aggregation
```

Example:

```text
GET /api/v1/systems/{id}/metrics
    ?start=2026-09-26T00:00:00Z
    &end=2026-09-26T12:00:00Z
    &interval=1m
    &aggregation=mean
```

Supported aggregation may include:

```text
mean
min
max
sum
count
```

The backend must validate that aggregation is meaningful for the requested metric.

---

# 16. Real-Time Metrics

```text
GET /api/v1/systems/{system_id}/metrics/latest
```

Response:

```json
{
  "data": [
    {
      "metric_id": "uuid",
      "metric_name": "cpu_usage_percent",
      "timestamp": "2026-09-26T10:00:00Z",
      "value": 42.7,
      "quality": "VALID"
    }
  ]
}
```

---

# 17. Anomalies

## List Anomalies

```text
GET /api/v1/anomalies
```

Query parameters:

```text
system_id
model_version_id
start
end
severity
is_anomaly
page
page_size
```

---

## Get Anomaly

```text
GET /api/v1/anomalies/{anomaly_id}
```

Response:

```json
{
  "data": {
    "id": "uuid",
    "system_id": "uuid",
    "timestamp": "2026-09-26T10:00:00Z",
    "anomaly_score": 0.84,
    "threshold": 0.61,
    "is_anomaly": true,
    "severity": "HIGH",
    "detection_method": "lstm_autoencoder"
  }
}
```

---

# 18. Anomaly Explanation

```text
GET /api/v1/anomalies/{anomaly_id}/explanation
```

Response:

```json
{
  "data": {
    "method": "shap",
    "contributions": [
      {
        "metric_id": "uuid",
        "metric_name": "cpu_usage_percent",
        "contribution_value": 0.72,
        "rank": 1
      },
      {
        "metric_id": "uuid",
        "metric_name": "memory_usage_percent",
        "contribution_value": 0.31,
        "rank": 2
      }
    ]
  }
}
```

The frontend must present these as model contributions/potential contributing factors rather than causal proof.

---

# 19. Models

## List Models

```text
GET /api/v1/models
```

Filters:

```text
model_type
task_type
status
```

---

## Get Model

```text
GET /api/v1/models/{model_id}
```

---

## Create Model Definition

```text
POST /api/v1/models
```

Request:

```json
{
  "name": "LSTM Autoencoder",
  "model_type": "lstm_autoencoder",
  "task_type": "anomaly_detection",
  "description": "Sequence reconstruction model",
  "framework": "pytorch"
}
```

---

# 20. Model Versions

```text
GET /api/v1/models/{model_id}/versions
```

```text
GET /api/v1/model-versions/{model_version_id}
```

Response:

```json
{
  "data": {
    "id": "uuid",
    "model_id": "uuid",
    "version": "1.0.0",
    "parameters": {},
    "preprocessing_config": {},
    "dataset_version": "dataset-v1",
    "random_seed": 42,
    "status": "READY"
  }
}
```

---

# 21. Model Training

Training may be long-running.

Endpoint:

```text
POST /api/v1/models/train
```

Request:

```json
{
  "model_type": "lstm_autoencoder",
  "system_id": "uuid",
  "dataset_version": "dataset-v1",
  "training_start": "2026-09-01T00:00:00Z",
  "training_end": "2026-09-15T00:00:00Z",
  "parameters": {
    "window_size": 60,
    "hidden_size": 64,
    "epochs": 50,
    "batch_size": 64,
    "learning_rate": 0.001
  },
  "random_seed": 42
}
```

Response:

```text
202 Accepted
```

```json
{
  "data": {
    "job_id": "uuid",
    "status": "QUEUED"
  }
}
```

---

# 22. ML Jobs

Long-running operations must expose job status.

```text
GET /api/v1/jobs/{job_id}
```

Response:

```json
{
  "data": {
    "id": "uuid",
    "type": "model_training",
    "status": "RUNNING",
    "progress": 62,
    "started_at": "2026-09-26T10:00:00Z",
    "finished_at": null,
    "error": null
  }
}
```

Possible statuses:

```text
QUEUED
RUNNING
COMPLETED
FAILED
CANCELLED
```

---

# 23. Model Inference

```text
POST /api/v1/models/{model_version_id}/predict
```

The endpoint must support the model's declared task.

For anomaly detection:

```json
{
  "system_id": "uuid",
  "start": "2026-09-26T10:00:00Z",
  "end": "2026-09-26T11:00:00Z"
}
```

Response:

```json
{
  "data": {
    "job_id": "uuid",
    "status": "QUEUED"
  }
}
```

For large time ranges, inference must be asynchronous.

---

# 24. Forecasting

## Generate Forecast

```text
POST /api/v1/forecasts
```

Request:

```json
{
  "system_id": "uuid",
  "model_version_id": "uuid",
  "forecast_horizon": 30,
  "start": "2026-09-26T10:00:00Z"
}
```

Response:

```text
202 Accepted
```

```json
{
  "data": {
    "job_id": "uuid",
    "status": "QUEUED"
  }
}
```

---

# 25. Get Forecast

```text
GET /api/v1/forecasts/{forecast_result_id}
```

---

# 26. Future Anomaly Risk

```text
GET /api/v1/forecasts/{forecast_result_id}/risk
```

Response:

```json
{
  "data": {
    "forecast_result_id": "uuid",
    "risk_level": "HIGH",
    "predicted_anomaly_points": 4,
    "horizon_steps": 30
  }
}
```

The API must clearly distinguish predicted risk from confirmed anomaly observations.

---

# 27. Experiments

## Create Experiment

```text
POST /api/v1/experiments
```

Request:

```json
{
  "name": "Baseline comparison",
  "description": "Comparison of classical anomaly detectors",
  "experiment_type": "baseline_comparison",
  "dataset_version": "dataset-v1"
}
```

---

## List Experiments

```text
GET /api/v1/experiments
```

Filters:

```text
experiment_type
created_by
status
page
page_size
```

---

## Get Experiment

```text
GET /api/v1/experiments/{experiment_id}
```

---

# 28. Experiment Runs

```text
GET /api/v1/experiments/{experiment_id}/runs
```

---

## Start Experiment Run

```text
POST /api/v1/experiments/{experiment_id}/runs
```

Request:

```json
{
  "model_version_id": "uuid",
  "parameters": {},
  "random_seed": 42,
  "training_period": {
    "start": "2026-09-01T00:00:00Z",
    "end": "2026-09-10T00:00:00Z"
  },
  "validation_period": {
    "start": "2026-09-10T00:00:00Z",
    "end": "2026-09-12T00:00:00Z"
  },
  "test_period": {
    "start": "2026-09-12T00:00:00Z",
    "end": "2026-09-15T00:00:00Z"
  }
}
```

Response:

```text
202 Accepted
```

---

# 29. Experiment Metrics

```text
GET /api/v1/experiments/{experiment_id}/metrics
```

Possible response:

```json
{
  "data": [
    {
      "run_id": "uuid",
      "metric_name": "f1",
      "metric_value": 0.84,
      "split": "test"
    }
  ]
}
```

The API must not fabricate metrics when no experiment result exists.

---

# 30. Fault Injection

Fault injection is a privileged operation.

## Start

```text
POST /api/v1/fault-injection/start
```

Request:

```json
{
  "system_id": "uuid",
  "experiment_id": "uuid",
  "fault_type": "cpu_load",
  "intensity": 70,
  "duration_seconds": 60,
  "configuration": {}
}
```

Response:

```text
202 Accepted
```

```json
{
  "data": {
    "fault_injection_id": "uuid",
    "status": "RUNNING"
  }
}
```

---

## Stop

```text
POST /api/v1/fault-injection/{fault_injection_id}/stop
```

This endpoint must immediately terminate the controlled fault where technically possible.

---

## Get Fault Injection

```text
GET /api/v1/fault-injection/{fault_injection_id}
```

---

# 31. Alerts

## List Alerts

```text
GET /api/v1/alerts
```

Filters:

```text
system_id
severity
status
channel
start
end
```

---

## Acknowledge Alert

```text
POST /api/v1/alerts/{alert_id}/acknowledge
```

Response:

```json
{
  "data": {
    "id": "uuid",
    "status": "ACKNOWLEDGED"
  }
}
```

---

# 32. Reports

## Generate Report

```text
POST /api/v1/reports
```

Request:

```json
{
  "report_type": "experiment",
  "experiment_id": "uuid",
  "parameters": {}
}
```

Response:

```text
202 Accepted
```

```json
{
  "data": {
    "job_id": "uuid",
    "status": "QUEUED"
  }
}
```

---

## Get Report

```text
GET /api/v1/reports/{report_id}
```

---

# 33. Dashboard Endpoint

The frontend may require an aggregated endpoint.

```text
GET /api/v1/dashboard/overview
```

Query:

```text
system_id
start
end
```

Response should contain:

```json
{
  "data": {
    "system": {},
    "latest_metrics": [],
    "active_anomalies": [],
    "recent_anomalies": [],
    "latest_forecast": {},
    "future_risk": {},
    "active_alerts": []
  }
}
```

This endpoint is intended for dashboard initialization and should avoid requiring many sequential frontend requests.

---

# 34. Health

```text
GET /health
```

Response:

```json
{
  "status": "ok"
}
```

---

# 35. Readiness

```text
GET /ready
```

The endpoint must verify required dependencies.

Example:

```json
{
  "status": "ready",
  "dependencies": {
    "database": "ok",
    "prometheus": "ok",
    "ml_engine": "ok"
  }
}
```

---

# 36. Pagination

Collection endpoints should support:

```text
page
page_size
```

Default:

```text
page = 1
page_size = 50
```

Maximum:

```text
page_size = 200
```

The maximum must be configurable.

---

# 37. Filtering

Filtering parameters must be explicitly defined per endpoint.

Do not implement unrestricted dynamic SQL filters.

All filters must be validated through Pydantic models.

---

# 38. Sorting

Supported collection endpoints may provide:

```text
sort_by
sort_order
```

Example:

```text
sort_by=timestamp
sort_order=desc
```

Only whitelisted fields may be used for sorting.

---

# 39. Time Range Validation

For time-series endpoints:

```text
start < end
```

must be enforced.

The backend must reject invalid ranges.

Maximum query duration should be configurable to prevent accidental huge queries.

---

# 40. Pydantic Schema Organization

Schemas should be separated from ORM models.

Recommended structure:

```text id="8vq5jz"
backend/
└── app/
    └── schemas/
        ├── auth.py
        ├── systems.py
        ├── metrics.py
        ├── anomalies.py
        ├── forecasts.py
        ├── models.py
        ├── experiments.py
        ├── fault_injection.py
        ├── alerts.py
        ├── reports.py
        ├── jobs.py
        └── common.py
```

Do not expose SQLAlchemy ORM objects directly as the public API contract.

---

# 41. Service Layer

Routes must remain thin.

Recommended architecture:

```text id="v0h9wo"
Router
  ↓
Application Service
  ↓
Domain Service
  ↓
Repository
  ↓
Database
```

ML operations:

```text id="xg3w7v"
Router
  ↓
ML Application Service
  ↓
ML Engine
  ↓
Model
```

---

# 42. Repository Layer

Database access should be isolated.

Recommended:

```text id="b6qj0n"
repositories/
├── user_repository.py
├── system_repository.py
├── metric_repository.py
├── anomaly_repository.py
├── forecast_repository.py
├── model_repository.py
├── experiment_repository.py
└── alert_repository.py
```

Repositories must not contain frontend/API concerns.

---

# 43. API Security

Every protected endpoint must verify authentication.

Role-based authorization must be enforced at the application layer.

Examples:

```text
VIEWER
→ read-only operations

RESEARCHER
→ experiments
→ model training
→ fault injection

ADMIN
→ user management
→ system administration
→ all researcher operations
```

---

# 44. Dangerous Operations

The following endpoints require elevated permissions:

```text
POST /fault-injection/start
POST /fault-injection/{id}/stop
POST /models/train
POST /experiments/{id}/runs
```

Fault injection must have additional safety validation.

---

# 45. API Idempotency

Operations that may be retried should support idempotency where appropriate.

For example:

```text
POST /reports
POST /models/train
POST /experiments/{id}/runs
```

can optionally accept:

```text
Idempotency-Key
```

The exact implementation will be defined during backend development.

---

# 46. API Rate Limiting

Rate limiting should be applied where appropriate.

Especially important for:

* authentication;
* report generation;
* model training;
* expensive inference;
* fault injection.

The exact limits are configuration values, not hard-coded scientific parameters.

---

# 47. OpenAPI

FastAPI must expose OpenAPI documentation in development.

Typical endpoints:

```text
/docs
/redoc
/openapi.json
```

Production exposure must be configurable.

The generated OpenAPI schema should remain synchronized with this document.

---

# 48. API Testing

Each endpoint must have tests for:

### Success

```text
valid request
expected response
```

### Validation

```text
missing field
invalid field
invalid UUID
invalid date range
```

### Authentication

```text
missing token
invalid token
expired token
```

### Authorization

```text
insufficient role
```

### Resource errors

```text
not found
conflict
```

### Internal failures

```text
database unavailable
ML job failure
external service failure
```

---

# 49. API Contract Rules

The following rules are mandatory:

1. Do not expose internal database implementation unnecessarily.
2. Do not expose stack traces.
3. Do not return fake ML results.
4. Do not claim a model is trained unless the training job completed successfully.
5. Do not return future anomaly predictions as confirmed events.
6. Validate all time ranges.
7. Validate all UUIDs.
8. Use UTC timestamps.
9. Keep long-running operations asynchronous.
10. Keep response structures consistent.
11. Version breaking API changes.
12. Keep OpenAPI documentation synchronized with implementation.

---

# 50. Definition of Done

The API specification is considered implemented when:

* authentication endpoints work;
* system endpoints work;
* metric endpoints work;
* anomaly endpoints work;
* model endpoints work;
* experiment endpoints work;
* forecasting endpoints work;
* fault-injection endpoints are protected;
* alert endpoints work;
* report endpoints work;
* job tracking works;
* health/readiness endpoints work;
* validation is implemented;
* error format is consistent;
* OpenAPI documentation is generated;
* API tests pass.

---

# 51. Relationship to Other Specifications

This document must remain consistent with:

```text
AGENTS.md
PROJECT_SPEC.md
ARCHITECTURE.md
DATABASE_SPEC.md
ML_SPEC.md
FORECASTING_SPEC.md
XAI_SPEC.md
EXPERIMENT_SPEC.md
TESTING_SPEC.md
ROADMAP.md
```

Any contradiction must be resolved before implementation continues.
