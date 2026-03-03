# ids568-milestone3-mwang233
## End-to-End ML Pipeline with Airflow, MLflow, Docker, and CI/CD

---

## 1. System Architecture
This project implements a fully containerized ML pipeline using:
- Apache Airflow (workflow orchestration)
- MLflow (experiment tracking & model registry)
- PostgreSQL (Airflow metadata DB)
- Redis (Celery broker)
- Docker Compose (container orchestration)
- GitHub Actions (CI-based model validation)

Pipeline tasks:
1. `preprocess_data`
2. `train_model`
3. `register_model`
DAG name: `train_pipeline`

1.1 ML Lifecycle Alignment

This project implements a production-style ML lifecycle including:

Data preprocessing (feature engineering stage)

Model training with hyperparameter experimentation

Metric evaluation and threshold validation

Model artifact logging

Model registry versioning

Production candidate promotion

CI-based automated validation

Reproducible containerized deployment

This reflects the following ML lifecycle stages:

Development → Experimentation (MLflow runs)

Validation → CI accuracy threshold check

Governance → Versioned model registry

Promotion → Alias-based production selection

Monitoring-ready architecture via retraining DAG

This ensures traceability, reproducibility, and rollback capability.

## 2. How to Run the System
### 2.1 Start the System
```bash
docker compose up -d
```
Verify containers:
docker compose ps

### 2.2 Access Airflow
Airflow UI:
http://localhost:8080

Default credentials:
username: airflow
password: airflow

Trigger DAG:
DAG: train_pipeline

Optional: Trigger with custom hyperparameter

Click "Trigger DAG with config"

Example JSON:
{
  "C": 5.0
}

This enables dynamic hyperparameter experimentation via DAG configuration.
Click "Trigger DAG"
Expected result:
All three tasks succeed

Retry enabled (if configured)

No manual intervention required

### 2.3 Access MLflow
MLflow UI:
http://localhost:5000

Expected:
Experiment: milestone3

Logged metrics (accuracy)
Parameter: C

Model artifact under model

Registered model: milestone3_model

Multiple model versions visible (e.g., v1–v12)
Each DAG run generates a new MLflow run and potentially a new model version.

## 3. Idempotency & Retry Strategy
DAG Idempotency

Training runs create new MLflow runs.

No destructive overwriting of previous models.

Re-triggering DAG does not corrupt previous runs.

Retry Strategy

Configured in Airflow default_args:

retries

retry_delay

Failures automatically retried.

## 4. Experiment Tracking Methodology
Tracked elements:
Hyperparameter: C

Metric: accuracy
Model artifact
Versioned model registry entry

Experiment name:
milestone3

Model registered as:
milestone3_model

Model versioning enables rollback capability.

## 5. Model Registry & Governance
Model lifecycle:
Training logs model

Model registered in MLflow Registry

Versioned storage

Production candidate selected based on accuracy

Rollback capability:
Previous model versions remain available

Can manually transition versions via MLflow UI

Production selection uses MLflow alias (@production).

The best-performing model version is assigned the alias "production",
allowing stable reference without relying on version numbers.

Older versions remain available for rollback.

## 6. CI-Based Model Validation
GitHub Actions workflow:

.github/workflows/train_and_validate.yml

Triggered on:

Push to main branch

CI Steps:

Install dependencies

Run training script

Run model_validation.py

Validation rule:CI enforces a minimum performance threshold:accuracy >= 0.6

If the trained model fails validation:

• CI pipeline fails
• Merge is blocked
• No production promotion occurs

This acts as a quality gate before model promotion.

## 7. Monitoring & Operational Notes
Potential risks:

Data drift

Metric degradation

Schema change

Model overfitting

Mitigation strategies:

Continuous retraining pipeline

MLflow metric monitoring

Version comparison

Airflow health checks

## 8. Reproducibility
Fully reproducible via:

docker compose up -d

All dependencies defined in:

requirements.txt

CI ensures automated verification.

All services (Airflow, MLflow, PostgreSQL, Redis)
are fully containerized and reproducible via Docker Compose.

## 9. Submission Checklist Alignment
✔ Containerized system
✔ Airflow DAG with 3 tasks
✔ MLflow experiment tracking
✔ Model registry versioning
✔ CI validation workflow
✔ Lineage report included
✔ README with run instructions

## 10. Rubric Mapping
| Requirement | Implementation |
|-------------|---------------|
| Containerized system | Docker Compose |
| Airflow orchestration | train_pipeline DAG |
| Retry & failure handling | Airflow default_args |
| Hyperparameter experimentation | MLflow experiment tracking |
| Model registry & versioning | milestone3_model |
| CI validation gate | GitHub Actions workflow |
| Idempotent pipeline | Non-destructive versioned runs |
| Lifecycle governance | Alias-based production selection |


Note: This project focuses on experiment tracking and registry governance.
Automatic model serving deployment is not implemented in this milestone.