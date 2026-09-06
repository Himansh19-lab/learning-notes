# CI/CD for ML and AI — Learn It From First Principles

## 1. Topic Overview

### What is CI/CD?

**CI/CD is a software engineering practice for automatically building, testing, validating, and delivering software changes.**

CI/CD commonly refers to:

- **CI — Continuous Integration**
- **CD — Continuous Delivery**
- **CD — Continuous Deployment**

In ML engineering, CI/CD extends beyond application code because ML systems also depend on:

- Data
- Features
- Models
- Training pipelines
- Infrastructure
- Configuration
- Dependencies

A useful mental model is:

```text
Developer Change
      ↓
Source Control
      ↓
CI Pipeline
      ↓
Build
      ↓
Test
      ↓
Validate
      ↓
Package
      ↓
Deploy
      ↓
Monitor
```

---

# 2. The Problem

Imagine a team has five ML engineers.

Each engineer changes:

```text
Python code
Dockerfiles
Dependencies
Training code
APIs
Infrastructure
```

Without automation, deployment might look like:

```text
Developer
   ↓
"Works on my machine"
   ↓
Manual testing
   ↓
Manual build
   ↓
Manual deployment
   ↓
Production failure
```

This creates problems:

- Human error
- Inconsistent environments
- Difficult rollbacks
- Slow releases
- Bugs discovered late
- Poor reproducibility
- Deployment risk

CI/CD attempts to make the path from code change to production **repeatable, automated, and testable**.

---

# 3. Why Do We Need CI?

Suppose three developers work on the same project:

```text
Developer A → feature A
Developer B → feature B
Developer C → bug fix
```

They all push code.

Without CI, the team may discover integration problems only after merging.

With CI:

```text
Developer
    ↓
Git Push
    ↓
CI Pipeline
    ↓
Lint
    ↓
Unit Tests
    ↓
Integration Tests
    ↓
Build
    ↓
Result
```

Every change gets automatically checked.

---

# 4. Continuous Integration

### Simple definition

**Continuous Integration means frequently integrating code changes into a shared repository and automatically validating those changes.**

The key idea is:

> **Integrate small changes frequently and detect problems early.**

Typical CI activities:

```text
Checkout code
     ↓
Install dependencies
     ↓
Lint
     ↓
Run unit tests
     ↓
Run integration tests
     ↓
Security checks
     ↓
Build artifact
```

---

# 5. Continuous Delivery

Continuous Delivery means keeping software in a state where it can be released to production reliably.

Conceptually:

```text
Code
 ↓
CI
 ↓
Build
 ↓
Test
 ↓
Package
 ↓
Deploy to Staging
 ↓
Validation
 ↓
Ready for Production
```

Production deployment may still require a manual approval.

---

# 6. Continuous Deployment

Continuous Deployment goes one step further.

If all automated checks pass:

```text
Code
 ↓
CI
 ↓
Tests
 ↓
Build
 ↓
Deploy
 ↓
Production
```

No manual production approval is necessarily required.

### Important distinction

```text
Continuous Delivery
→ Production-ready, release may require approval

Continuous Deployment
→ Automatically deploy successful changes
```

---

# 7. CI vs CD

| Concept | Main Goal |
|---|---|
| CI | Automatically validate code changes |
| Continuous Delivery | Keep software ready to release |
| Continuous Deployment | Automatically release validated changes |

A simple mental model:

```text
CI
 ↓
"Is this change safe?"

Continuous Delivery
 ↓
"Is this release ready?"

Continuous Deployment
 ↓
"Release it automatically."
```

---

# 8. CI/CD in Machine Learning

Traditional software:

```text
Code
 ↓
Build
 ↓
Test
 ↓
Deploy
```

ML systems:

```text
Code
 +
Data
 +
Features
 +
Model
 +
Infrastructure
      ↓
Build
      ↓
Test
      ↓
Validate
      ↓
Train / Package
      ↓
Deploy
      ↓
Monitor
```

This creates additional challenges.

For example:

```text
Code is correct
BUT
Data schema changed
```

The application may still run while the model behaves incorrectly.

Therefore ML CI/CD often needs additional validation.

---

# 9. ML CI/CD Pipeline

A realistic ML pipeline might look like:

```text
Git Push
   ↓
Unit Tests
   ↓
Code Quality
   ↓
Data Validation
   ↓
Model Tests
   ↓
Build Docker Image
   ↓
Train / Evaluate
   ↓
Model Validation
   ↓
Register Model
   ↓
Deploy to Staging
   ↓
Integration Tests
   ↓
Production Approval
   ↓
Production Deployment
   ↓
Monitoring
```

---

# 10. CI/CD Components

A typical CI/CD ecosystem contains:

### Source Control

Examples:

- Git
- GitHub
- GitLab
- Bitbucket

### CI/CD Platform

Examples:

- GitHub Actions
- GitLab CI/CD
- Jenkins
- CircleCI
- Azure Pipelines

### Containerization

```text
Docker
```

### Orchestration

```text
Kubernetes
```

### Cloud

Examples:

```text
AWS
Azure
GCP
```

### ML Lifecycle

Examples:

```text
MLflow
DVC
Feature Stores
Model Registries
```

---

# 11. Git as the Foundation

A common workflow:

```text
Developer
   ↓
git clone
   ↓
Create branch
   ↓
Make changes
   ↓
Commit
   ↓
Push
   ↓
Pull Request
   ↓
CI Pipeline
```

Typical commands:

```bash
git checkout -b feature/model-api

git add .

git commit -m "Add model prediction API"

git push origin feature/model-api
```

The pull request can trigger CI automatically.

---

# 12. CI Pipeline Example

A simplified pipeline:

```text
Pull Request
     ↓
Checkout repository
     ↓
Install Python
     ↓
Install dependencies
     ↓
Run linting
     ↓
Run tests
     ↓
Build application
     ↓
Build Docker image
     ↓
Security scan
     ↓
Pass / Fail
```

If any critical stage fails:

```text
Pipeline
   ↓
FAILED
   ↓
Deployment blocked
```

This is one of the central benefits of CI/CD.

---

# 13. GitHub Actions Example

A simplified GitHub Actions workflow:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: Run tests
        run: |
          pytest
```

The exact action versions and supported runtimes can change, so production workflows should follow current official documentation.

---

# 14. Why Automated Testing Matters

Without tests:

```text
Code Change
    ↓
Deploy
    ↓
Discover Bug
```

With CI:

```text
Code Change
    ↓
Tests
    ↓
Failure detected
    ↓
Fix before deployment
```

Common tests include:

### Unit tests

Test individual functions.

```python
def add(a, b):
    return a + b
```

Test:

```python
assert add(2, 3) == 5
```

### Integration tests

Test whether components work together.

Example:

```text
API
 ↓
Model
 ↓
Database
```

### End-to-end tests

Test a complete user/system workflow.

---

# 15. ML-Specific Testing

ML systems require more than standard software tests.

Consider testing:

### Data schema

```text
Expected:
age → numeric
income → numeric
country → string
```

If the schema suddenly changes:

```text
income → string
```

the pipeline should fail before production.

### Feature validation

Check:

- Missing values
- Range
- Data types
- Distribution
- Cardinality

### Model validation

Check:

- Accuracy
- Precision
- Recall
- F1
- ROC-AUC
- Business metrics

### Prediction sanity checks

Example:

```text
Prediction probability must be between 0 and 1
```

---

# 16. Model Validation Gates

A production pipeline can define minimum requirements:

```text
Model Evaluation
       ↓
Accuracy >= 90%?
       ↓
      Yes
       ↓
Precision >= 85%?
       ↓
      Yes
       ↓
Deploy
```

If:

```text
Accuracy = 83%
```

then:

```text
Deployment blocked
```

This is called a **quality gate** or validation gate.

---

# 17. Docker in CI/CD

Docker helps create reproducible environments.

Instead of:

```text
Works on Developer A's laptop
```

we can package:

```text
Application
+
Python
+
Libraries
+
System dependencies
```

into an image.

Example Dockerfile:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "app.py"]
```

CI can build the image:

```bash
docker build -t my-ml-api:latest .
```

Then test it before deployment.

---

# 18. Container CI/CD Flow

```text
Git Push
   ↓
CI
   ↓
Test
   ↓
Docker Build
   ↓
Docker Test
   ↓
Push Image
   ↓
Container Registry
   ↓
Deployment
```

A container registry might store:

```text
my-ml-api:1.0.0
my-ml-api:1.1.0
my-ml-api:1.2.0
```

---

# 19. Deployment Strategies

There are several ways to release a new version.

## Rolling Deployment

Gradually replace old instances:

```text
Old Version
Old Version
Old Version
      ↓
New Version
Old Version
Old Version
      ↓
New Version
New Version
Old Version
      ↓
New Version
New Version
New Version
```

---

## Blue-Green Deployment

Maintain two environments:

```text
Blue → Current production
Green → New version
```

Test Green.

Then switch traffic:

```text
Users
  ↓
Green
```

If problems occur:

```text
Users
  ↓
Blue
```

Rollback can be fast.

---

# 20. Canary Deployment

Send a small percentage of traffic to the new version.

```text
Users
  │
  ├── 95% → Model V1
  │
  └── 5%  → Model V2
```

Monitor:

- Error rate
- Latency
- Business metrics
- Model performance

If healthy:

```text
5%
 ↓
20%
 ↓
50%
 ↓
100%
```

If unhealthy:

```text
Traffic → V1
```

---

# 21. CI/CD and Kubernetes

A common production architecture:

```text
Git
 ↓
CI/CD
 ↓
Docker Image
 ↓
Container Registry
 ↓
Kubernetes
 ↓
Pods
 ↓
Model API
```

Kubernetes can manage:

- Deployment
- Scaling
- Networking
- Health checks
- Rollouts
- Rollbacks

---

# 22. Infrastructure as Code

Production infrastructure should ideally be reproducible.

Tools include:

- Terraform
- Pulumi
- Cloud-native infrastructure tools

Conceptually:

```text
Infrastructure Code
        ↓
CI
        ↓
Validate
        ↓
Plan
        ↓
Approval
        ↓
Apply
```

This allows infrastructure changes to go through a controlled workflow.

---

# 23. Secrets Management

Never put secrets directly into source code.

Bad:

```python
API_KEY = "my-secret-key"
```

Better:

```text
Secret Manager
      ↓
CI/CD
      ↓
Environment / Workload
```

Examples include:

- Cloud secret managers
- Kubernetes Secrets
- CI/CD secret stores
- Vault

Secrets should also never be printed in logs.

---

# 24. Environment Promotion

A common workflow:

```text
Development
     ↓
Testing
     ↓
Staging
     ↓
Production
```

Example:

```text
Developer
   ↓
Dev
   ↓
CI
   ↓
Staging
   ↓
Automated Tests
   ↓
Approval
   ↓
Production
```

Each environment can provide increasing confidence.

---

# 25. Artifact Versioning

A CI/CD pipeline should make artifacts identifiable.

Example:

```text
Application:
v1.4.0

Docker:
my-api:1.4.0

Model:
fraud-model:v17

Git:
commit abc123
```

This enables traceability:

```text
Production
    ↓
Docker Image v1.4.0
    ↓
Git Commit abc123
    ↓
Model Version 17
    ↓
Training Run
    ↓
Dataset Version
```

This is extremely valuable for ML debugging and audits.

---

# 26. ML Model CI/CD vs Model Retraining

These are related but different.

### CI/CD

Moves validated software/model artifacts through environments.

```text
Code
 ↓
Build
 ↓
Test
 ↓
Deploy
```

### Continuous Training

Automatically retrains a model when appropriate.

```text
New Data
   ↓
Validation
   ↓
Training
   ↓
Evaluation
   ↓
Model Registry
   ↓
Deployment
```

A mature MLOps platform may combine both.

---

# 27. Continuous Training

A possible ML workflow:

```text
New Data
   ↓
Data Validation
   ↓
Feature Pipeline
   ↓
Training
   ↓
Cross-Validation
   ↓
Evaluation
   ↓
Compare with Current Model
   ↓
Better?
  /   \
No     Yes
↓       ↓
Stop   Register
         ↓
      Deploy
```

The important principle is:

> **New data should not automatically mean "deploy the new model."**

The model should pass validation criteria first.

---

# 28. CI/CD with MLflow

MLflow can fit into the pipeline:

```text
Git
 ↓
CI
 ↓
Training
 ↓
MLflow Tracking
 ↓
Evaluation
 ↓
Model Registry
 ↓
Deployment
```

For example:

```text
Training Run
     ↓
Parameters
Metrics
Artifacts
     ↓
MLflow
     ↓
Candidate Model
     ↓
Validation
     ↓
Registry
```

This provides experiment and model lineage that complements CI/CD.

---

# 29. DVC and Data Versioning

Git is excellent for source code, but large datasets are often managed differently.

Tools such as DVC can help version data and ML artifacts.

Conceptually:

```text
Git
 ↓
Code Version

DVC / Object Storage
 ↓
Data Version
```

Then:

```text
Code v10
+
Data v7
+
Model v15
```

can be associated with a specific experiment.

---

# 30. CI/CD Pipeline for a Production ML System

A more complete architecture:

```text
                    Developer
                        │
                        ↓
                    Git Push
                        │
                        ↓
                 Pull Request
                        │
                        ↓
              ┌─────────────────┐
              │       CI        │
              ├─────────────────┤
              │ Lint            │
              │ Unit Tests      │
              │ Data Tests      │
              │ Model Tests     │
              │ Security Scan   │
              │ Docker Build    │
              └────────┬────────┘
                       │
                     Pass
                       ↓
                 Staging Deploy
                       ↓
               Integration Tests
                       ↓
                 Model Validation
                       ↓
                 Approval / Gate
                       ↓
                Production Deploy
                       ↓
                  Monitoring
                       ↓
                 Drift / Alerts
                       ↓
                Retraining Flow
```

---

# 31. Rollbacks

A good deployment system should make rollback easy.

Suppose:

```text
Production
   ↓
Model v17
```

New release:

```text
Model v18
```

Performance becomes worse.

Rollback:

```text
v18
 ↓
Problem
 ↓
Rollback
 ↓
v17
```

This is why versioned artifacts are essential.

Never rely only on:

```text
latest
```

for critical production deployments.

Prefer immutable or traceable versions such as:

```text
image:1.4.2
model:v17
commit:abc123
```

---

# 32. Monitoring After Deployment

CI/CD does not end when deployment finishes.

A production ML lifecycle is:

```text
Build
 ↓
Test
 ↓
Deploy
 ↓
Monitor
 ↓
Learn
 ↓
Improve
```

Monitor application metrics:

```text
Latency
Error Rate
CPU
Memory
Throughput
```

Monitor ML metrics:

```text
Prediction Distribution
Data Drift
Model Performance
Feature Drift
Business Metrics
```

---

# 33. CI/CD vs MLOps

These concepts overlap but are not identical.

### CI/CD

Focuses heavily on:

```text
Code
Build
Test
Release
Deployment
```

### MLOps

Adds ML-specific lifecycle concerns:

```text
Data
Features
Experiments
Training
Models
Evaluation
Registry
Deployment
Monitoring
Retraining
```

Think:

```text
CI/CD
   +
ML-specific lifecycle
   =
MLOps
```

This is a useful simplified mental model, not a strict definition.

---

# 34. Common Mistakes

## Mistake 1: Deploying directly from a developer laptop

Better:

```text
Git
 ↓
CI
 ↓
Tests
 ↓
Artifact
 ↓
Deployment
```

---

## Mistake 2: No automated tests

This allows regressions to reach production.

---

## Mistake 3: Using mutable artifacts

Avoid depending exclusively on:

```text
latest
```

Use versioned artifacts.

---

## Mistake 4: Storing secrets in Git

Use secret management systems.

---

## Mistake 5: Deploying every newly trained model

A model should pass quality gates first.

---

## Mistake 6: No rollback plan

Every production release should have a recovery strategy.

---

## Mistake 7: Testing only code

ML systems also need:

```text
Data tests
Feature tests
Model tests
Inference tests
```

---

# 35. Security in CI/CD

A production pipeline should consider:

- Dependency vulnerabilities
- Container vulnerabilities
- Secret leakage
- Access control
- Supply-chain security
- Least privilege
- Signed artifacts where appropriate
- Audit logs

Example:

```text
Code
 ↓
Dependency Scan
 ↓
SAST
 ↓
Container Scan
 ↓
Tests
 ↓
Deploy
```

Do not treat security as an optional final step.

---

# 36. CI/CD Quality Gates

A pipeline might enforce:

```text
Unit tests       → PASS
Coverage         → >= threshold
Security scan    → PASS
Docker build     → PASS
Model accuracy   → >= threshold
Model recall     → >= threshold
Integration test → PASS
```

Only then:

```text
Deploy
```

Quality gates turn organizational requirements into automated controls.

---

# 37. Example Project Structure

A simple ML API project:

```text
ml-project/
│
├── app/
│   ├── main.py
│   └── model.py
│
├── tests/
│   ├── test_model.py
│   └── test_api.py
│
├── training/
│   └── train.py
│
├── Dockerfile
├── requirements.txt
├── pyproject.toml
│
└── .github/
    └── workflows/
        └── ci.yml
```

A larger project might also include:

```text
infra/
deploy/
configs/
data/
models/
monitoring/
```

The exact structure should match the project's complexity.

---

# 38. End-to-End Project

Build a **CI/CD pipeline for a machine-learning prediction API**.

## Objective

Train a model and expose it through an API.

### Architecture

```text
Developer
   ↓
GitHub
   ↓
GitHub Actions
   ↓
Tests
   ↓
Docker Build
   ↓
Container Registry
   ↓
Cloud / Kubernetes
   ↓
ML API
```

### ML workflow

```text
Training
   ↓
Evaluation
   ↓
MLflow
   ↓
Model Registry
   ↓
Deployment
```

### Suggested components

```text
Python
Scikit-learn
FastAPI
Pytest
Docker
GitHub Actions
MLflow
```

---

# 39. Example CI/CD Learning Project

## Step 1 — Build model

```python
model.fit(X_train, y_train)
```

## Step 2 — Test model

```python
assert accuracy >= 0.80
```

## Step 3 — Package API

```text
FastAPI
   ↓
/predict
```

## Step 4 — Dockerize

```bash
docker build -t ml-api:1.0.0 .
```

## Step 5 — CI

```text
Push
 ↓
Tests
 ↓
Build
```

## Step 6 — CD

```text
Build
 ↓
Push Image
 ↓
Deploy
```

## Step 7 — Monitor

```text
Latency
Errors
Predictions
Drift
```

---

# 40. Interview Questions

## Beginner

1. What is CI?
2. What is CD?
3. What is the difference between Continuous Delivery and Continuous Deployment?
4. Why is CI/CD useful?
5. What is a CI pipeline?
6. What is a deployment pipeline?
7. What is automated testing?
8. Why use Docker in CI/CD?
9. What is a container registry?
10. What is a rollback?

## Intermediate

11. Explain a typical CI/CD pipeline.
12. How does Git trigger CI?
13. How would you deploy a Dockerized ML API?
14. What tests should run in CI?
15. What is a quality gate?
16. How would you handle secrets?
17. What is blue-green deployment?
18. What is canary deployment?
19. How would you version artifacts?
20. How does MLflow fit into CI/CD?

## Advanced

21. Design CI/CD for an ML platform.
22. How would you implement continuous training?
23. How would you prevent a bad model from reaching production?
24. How would you design rollback for model deployments?
25. How would you secure a CI/CD pipeline?
26. How would you deploy models with Kubernetes?
27. How would you handle data validation in CI/CD?
28. How would you handle model drift?
29. How would you make deployments reproducible?
30. Design a complete MLOps CI/CD architecture.

---

# 41. Scenario-Based Interview Questions

### Scenario 1

A developer pushes code that passes unit tests but breaks the production model.

How would you improve the pipeline?

### Scenario 2

A new model has higher accuracy but significantly lower recall.

Would you automatically deploy it?

Why?

### Scenario 3

A Docker image works locally but fails in production.

How would you debug the pipeline?

### Scenario 4

A model deployment causes latency to increase by 300%.

What should happen?

### Scenario 5

A new dataset arrives every day.

How would you design a continuous-training pipeline?

---

# 42. Strong Interview Answer

### Question

> What is CI/CD?

### Short answer

> "CI/CD is a set of engineering practices that automate the process of integrating, testing, building, and delivering software. Continuous Integration focuses on validating changes frequently, while Continuous Delivery keeps releases ready for deployment and Continuous Deployment can automatically release validated changes."

### Strong ML answer

> "In ML systems, CI/CD extends beyond application code. The pipeline should also validate data schemas, features, model behavior, and deployment artifacts. A typical ML pipeline might run unit and integration tests, validate data, build a Docker image, train or package a model, evaluate it against quality gates, register the approved model, deploy it to staging, run integration tests, and then promote it to production. MLflow can provide experiment and model tracking, while Docker and Kubernetes can provide reproducible packaging and deployment."

---

# 43. Flashcards

**Q:** What is CI?

**A:** Continuous Integration: frequently integrating code changes and automatically validating them.

**Q:** What is Continuous Delivery?

**A:** Keeping software in a releasable state, often with a manual production approval.

**Q:** What is Continuous Deployment?

**A:** Automatically deploying validated changes to production.

**Q:** What is a pipeline?

**A:** An automated sequence of build, test, validation, and deployment steps.

**Q:** What is a quality gate?

**A:** A condition that must pass before a pipeline can proceed.

**Q:** What is a rollback?

**A:** Reverting production to a previously known-good version.

**Q:** What is canary deployment?

**A:** Sending a small amount of traffic to a new version before gradually increasing it.

**Q:** What is blue-green deployment?

**A:** Maintaining separate current and new environments and switching traffic between them.

**Q:** Why use Docker?

**A:** To package applications and dependencies into reproducible environments.

**Q:** Why is ML CI/CD different from normal software CI/CD?

**A:** ML systems also depend on data, features, models, training, and model-specific validation.

**Q:** What is continuous training?

**A:** Automatically retraining models when data or predefined triggers indicate that retraining is appropriate.

**Q:** Why use MLflow?

**A:** To track experiments and manage model lifecycle information alongside the ML pipeline.

---

# 44. Knowledge Check

Try these before looking at the answers.

## Beginner

1. What problem does CI solve?
2. What is the difference between Continuous Delivery and Continuous Deployment?
3. Why are automated tests important?
4. What is a deployment artifact?
5. Why are rollbacks important?

## Intermediate

6. What tests should an ML CI pipeline include?
7. Why should secrets not be stored in Git?
8. What is a canary deployment?
9. What is a quality gate?
10. How does Docker improve deployment consistency?

## Advanced

11. Design a CI/CD pipeline for a machine-learning API.
12. How would you prevent a poor model from being deployed?
13. How would you implement continuous training?
14. How would you connect Git, Docker, MLflow, and Kubernetes?
15. How would you design a safe model rollback strategy?

---

# 45. Answer Key

### 1. CI

CI detects integration and code-quality problems early by automatically validating changes.

### 2. Delivery vs Deployment

Continuous Delivery keeps software ready for release; Continuous Deployment automatically releases validated changes.

### 3. Automated tests

They detect regressions before deployment.

### 4. Artifact

A deployable output such as a Docker image, package, binary, or model artifact.

### 5. Rollback

It provides a way to restore a known-good version after a failed release.

### 6. ML CI tests

Potentially:

```text
Unit tests
Integration tests
Data validation
Feature validation
Model tests
Inference tests
Security checks
```

### 7. Secrets

Secrets in source control can be exposed and are difficult to rotate safely.

### 8. Canary

A deployment where only a small portion of traffic is initially sent to the new version.

### 9. Quality gate

A required condition that controls whether the pipeline can proceed.

### 10. Docker

It packages software and dependencies into a consistent runtime environment.

---

# 46. Cheat Sheet

## CI

```text
Code
 ↓
Build
 ↓
Lint
 ↓
Test
 ↓
Security
 ↓
Artifact
```

## CD

```text
Artifact
 ↓
Staging
 ↓
Validation
 ↓
Production
```

## ML CI/CD

```text
Code
 +
Data
 +
Features
 +
Model
 ↓
Validate
 ↓
Build
 ↓
Test
 ↓
Evaluate
 ↓
Register
 ↓
Deploy
 ↓
Monitor
```

## Deployment strategies

```text
Rolling
Blue-Green
Canary
```

## Core tools

```text
Git
GitHub Actions / GitLab CI / Jenkins
Docker
Container Registry
Kubernetes
MLflow
DVC
Terraform
Cloud Platforms
```

## Core ML quality gates

```text
Data Quality
Model Quality
Inference Tests
Security
Latency
Business Metrics
```

---

# 47. If You Remember Only 5 Things

1. **CI automatically validates changes before they are integrated or released.**
2. **Continuous Delivery keeps software ready to release; Continuous Deployment automatically releases it.**
3. **ML CI/CD must validate not only code, but also data, features, models, and inference behavior.**
4. **Versioned artifacts, automated tests, quality gates, and rollback mechanisms make deployments safer.**
5. **CI/CD is a major building block of MLOps, but MLOps also includes experimentation, data, training, model management, monitoring, and retraining.**

---

# 48. 1-Minute Interview Explanation

> "CI/CD automates the process of integrating, testing, building, and delivering software. Continuous Integration focuses on validating changes frequently, while Continuous Delivery keeps releases ready for production and Continuous Deployment automatically releases validated changes. In ML, CI/CD also needs data validation, model testing, evaluation gates, artifact versioning, and model deployment controls. A typical pipeline might run tests, build a Docker image, evaluate a model, register it with MLflow, deploy to staging, run integration tests, and then promote it to production. Monitoring and rollback are essential parts of the production lifecycle."

---

# 49. 5-Minute Interview Explanation

CI/CD is a set of engineering practices for making software integration and delivery automated, repeatable, and reliable.

Continuous Integration means developers frequently integrate changes into a shared repository and automatically run validation such as linting, unit tests, integration tests, security checks, and builds.

Continuous Delivery extends this by keeping validated software in a state where it can be released reliably. A production deployment may still require manual approval.

Continuous Deployment goes further by automatically deploying changes that pass the required quality gates.

For ML systems, CI/CD is more complicated because the system depends on more than application code. Data schemas, feature transformations, training pipelines, model artifacts, model quality, and infrastructure all need validation.

A mature ML CI/CD pipeline might look like:

```text
Git Push
 ↓
Unit Tests
 ↓
Integration Tests
 ↓
Data Validation
 ↓
Model Validation
 ↓
Docker Build
 ↓
Security Scan
 ↓
Model Evaluation
 ↓
MLflow / Model Registry
 ↓
Staging Deployment
 ↓
Integration Tests
 ↓
Quality Gate
 ↓
Production
 ↓
Monitoring
```

The deployment process should also support safe release strategies such as rolling, blue-green, or canary deployments.

Versioning is critical. A production deployment should be traceable to the exact source-code commit, container image, model version, configuration, and potentially training/data version.

Finally, deployment is not the end. Production systems need monitoring, drift detection, alerting, and retraining workflows.

---

# 50. Learning Roadmap

```text
Git Fundamentals
      ↓
GitHub / GitLab
      ↓
CI Concepts
      ↓
Automated Testing
      ↓
GitHub Actions / Jenkins
      ↓
Docker
      ↓
Container Registry
      ↓
CD Concepts
      ↓
Deployment Strategies
      ↓
Kubernetes
      ↓
Infrastructure as Code
      ↓
MLflow
      ↓
Model Registry
      ↓
Continuous Training
      ↓
Monitoring
      ↓
Production MLOps
```

## Recommended project

Build a complete **ML CI/CD pipeline**:

```text
Dataset
   ↓
Training Code
   ↓
Git Repository
   ↓
Unit Tests
   ↓
Data Validation
   ↓
Model Evaluation
   ↓
MLflow Tracking
   ↓
Docker Image
   ↓
CI Pipeline
   ↓
Container Registry
   ↓
Staging
   ↓
Integration Tests
   ↓
Production
   ↓
Monitoring
   ↓
Rollback / Retraining
```

### Suggested stack

```text
Python
Scikit-learn
Pytest
Git
GitHub Actions
Docker
MLflow
FastAPI
Kubernetes
```

The goal is to understand not just how to write a CI/CD YAML file, but how **code → tests → artifacts → model validation → deployment → monitoring** forms one reliable production system.
