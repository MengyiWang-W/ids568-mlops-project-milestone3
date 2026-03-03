IDS568 – Milestone 3

End-to-End ML Pipeline Governance Documentation

1. Overview

This project implements an end-to-end ML pipeline orchestrated by Apache Airflow, tracked with MLflow, and validated via CI/CD.

The objective of this lineage report is to document:

Run comparisons and analysis

Justification for production candidate selection

Identified risks and monitoring needs

The model lifecycle includes preprocessing, training, validation, registration, versioning, and governance controls.

2. Run Comparisons and Analysis

Five model configurations were evaluated using different regularization parameters:

C = 0.1
C = 0.5
C = 1.0
C = 2.0
C = 5.0

All runs were logged in the MLflow experiment: milestone3.

2.1 Observed Metrics

All evaluated runs achieved:

accuracy = 0.60

This indicates that under the current dataset and feature engineering approach:

Hyperparameter variation did not materially change performance.

Model performance plateaued at 0.60 accuracy.

MLflow comparison tools (table view + chart view) were used to visualize parameter–metric relationships and confirm metric consistency across runs.

2.2 Version Mapping

Each successful Airflow DAG run created:

A new MLflow run

A corresponding model version in milestone3_model

Example:

Version 12
C = 5.0
accuracy = 0.60
Alias: @production

Version registration was triggered via the register_model Airflow task.

3. Production Candidate Selection Justification
3.1 Selection Criteria

Because all runs achieved identical accuracy (0.60), selection was governed by:

CI validation success (accuracy ≥ 0.6 threshold)

Successful Airflow task completion

Proper model registration in MLflow

Most recent validated configuration

Version 12 met all criteria.

3.2 Governance Mechanism

Production selection uses MLflow alias:

@production

This allows:

Stable model reference without relying on version numbers

Clean separation between candidate versions and production designation

Manual or automated rollback capability

The selected version contains the following tags:

C: 5.0

accuracy: 0.60

pipeline: train_pipeline

promotion_reason: meets_threshold

These tags provide traceability and explainability for model promotion.

4. CI-Based Validation and Quality Gate

GitHub Actions workflow: .github/workflows/train_and_validate.yml

Triggered on:

push to main branch

CI Process:

Install dependencies

Run training script (train.py)

Run model_validation.py

Validation rule:

accuracy ≥ 0.6

If validation fails:

CI exits with non-zero code

Merge is blocked

Production promotion does not occur

This enforces a minimum performance threshold before accepting new changes into the main branch.

CI validation is ephemeral and does not persist MLflow artifacts; it serves as a quality gate mechanism rather than a production deployment trigger.

5. Identified Risks and Monitoring Needs
5.1 Data Drift

Risk:
Changes in input data distribution may degrade model performance.

Mitigation:

Continuous retraining via Airflow DAG

MLflow metric comparison across versions

5.2 Metric Degradation

Risk:
Model accuracy may fall below the CI threshold (0.6).

Mitigation:

CI validation blocks degraded models

Version comparison in MLflow

Alias reassignment if rollback is required

5.3 Schema Change

Risk:
Changes in input schema could break preprocessing.

Mitigation:

Centralized preprocessing logic

DAG-level failure detection

Retry strategy via Airflow default_args

5.4 Overfitting

Risk:
Limited dataset size may cause unstable performance.

Mitigation:

Metric tracking across runs

Hyperparameter experimentation

Future extension: cross-validation or additional metrics

6. Lineage Traceability Summary

The pipeline ensures full lineage traceability:

Data → Preprocess → Train → Evaluate → Register → Version → Alias (@production)

For each model version, we can identify:

Source training run

Hyperparameters

Accuracy metric

Promotion reason

Pipeline task origin

Version history

Rollback is possible by reassigning the @production alias to a previous version.

7. Conclusion

This pipeline demonstrates:

End-to-end experiment tracking

Versioned model registry

Alias-based governance

CI-enforced performance validation

Idempotent retraining workflow

Rollback capability

Although hyperparameter variation did not materially change performance under the current dataset, governance mechanisms ensure safe promotion, monitoring readiness, and controlled lifecycle management.