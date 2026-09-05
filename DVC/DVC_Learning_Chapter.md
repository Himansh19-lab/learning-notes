# DVC (Data Version Control) — ML/MLOps Learning Chapter

> **Learning goal:** Understand DVC from first principles, use it in an ML project, and explain its role in reproducible ML pipelines and interviews.

## 1. Topic Overview

### One-line definition

**DVC (Data Version Control)** is an open-source tool that helps version, track, reproduce, and share large datasets, ML models, and data-processing pipelines alongside Git.

### Simple definition

Git is excellent for code, but ML projects also depend on things such as:

- Training datasets
- Validation/test datasets
- Model files
- Feature data
- Pipeline outputs

DVC lets us manage those large or generated artifacts without putting the actual files directly into Git.

### Where DVC fits

```text
Git
 └── Code + configuration + DVC metadata

DVC
 ├── Dataset versions
 ├── Model versions
 ├── Pipeline stages
 └── Reproducible experiments/pipelines

Remote Storage
 └── Actual large data/model artifacts
```

---

# 2. The Problem DVC Solves

Suppose you train a fraud-detection model today.

```text
code v1
dataset v7
features v3
model v12
accuracy = 94%
```

Two weeks later:

```text
code v2
dataset ??? 
features ???
model ???
accuracy = 91%
```

You may know which Git commit contains the code, but you may not know exactly which dataset and generated artifacts produced the model.

This is the core ML reproducibility problem.

## Why Git alone is not enough

Git works very well for text-based source code, but datasets and trained models can be huge.

Putting a 20 GB dataset directly into Git is generally a poor workflow.

DVC separates:

**Metadata/version history** from **large artifact storage**.

---

# 3. Why Do We Need DVC?

Without data versioning, teams can encounter:

- "Which dataset trained this model?"
- "Why did model performance change?"
- "Can I reproduce last month's model?"
- "Which version of the features was used?"
- "Can another engineer obtain exactly the same training inputs?"

DVC addresses these problems through versioned data references and reproducible pipelines.

## Important idea

DVC does **not** replace Git.

A common architecture is:

```text
Git → versions code and DVC metadata
DVC → tracks data/model/pipeline relationships
Remote Storage → stores large files
```

---

# 4. Intuition

Think of Git as a **recipe book**.

It tells you:

> "At commit X, the recipe looked like this."

DVC adds the equivalent of:

> "And this recipe used exactly these ingredients."

The actual ingredients can live in object storage or another DVC-supported remote.

---

# 5. How DVC Works

A simplified workflow:

```text
Raw Dataset
    ↓
DVC tracks dataset
    ↓
DVC metadata committed to Git
    ↓
Actual data stored in DVC remote
    ↓
Pipeline runs
    ↓
Model generated
    ↓
Model tracked by DVC
    ↓
Git records pipeline/code/config state
```

A teammate can then checkout a Git revision and use DVC to obtain the corresponding data and reproduce the pipeline.

---

# 6. Core Concepts

## 6.1 DVC-tracked files

A large file can be tracked by DVC rather than committed directly to Git.

For example:

```bash
dvc add data/train.csv
```

This typically creates a small `.dvc` metadata file.

Conceptually:

```text
data/train.csv
      ↓
DVC metadata (.dvc)
      ↓
Git tracks metadata
      ↓
DVC remote stores actual data
```

---

## 6.2 DVC Remote

The remote is where DVC stores the actual versioned artifacts.

Depending on the setup, this can be backed by storage such as:

- Local/shared storage
- Amazon S3
- Google Cloud Storage
- Azure Blob Storage
- Other supported remote backends

The important architectural idea is:

```text
Git repository
      │
      ├── source code
      └── DVC metadata
             │
             ↓
        DVC remote
             │
             └── datasets/models
```

---

## 6.3 DVC Pipeline

DVC can represent an ML workflow as stages.

Example:

```text
data preparation
       ↓
feature engineering
       ↓
training
       ↓
evaluation
```

A pipeline stage can describe:

- Command
- Dependencies
- Parameters
- Outputs

DVC can use this information to determine what needs to be rerun.

---

# 7. A Simple ML Project

Example structure:

```text
ml-project/
├── data/
│   ├── raw/
│   └── processed/
├── src/
│   ├── prepare.py
│   ├── train.py
│   └── evaluate.py
├── models/
├── params.yaml
├── dvc.yaml
├── dvc.lock
├── .gitignore
└── README.md
```

Typical responsibilities:

| File | Purpose |
|---|---|
| `dvc.yaml` | Pipeline definition |
| `dvc.lock` | Concrete dependency/output state for a pipeline |
| `params.yaml` | Experiment parameters |
| `.dvc` files | Metadata for DVC-tracked files/directories |
| Git | Code/config/version history |
| DVC remote | Large artifacts |

---

# 8. Basic Setup

## Install

A typical installation is:

```bash
pip install dvc
```

For a particular remote backend, an extra may be required depending on the backend.

## Initialize

Inside a Git repository:

```bash
git init
dvc init
```

Then commit the DVC configuration:

```bash
git add .dvc .gitignore
git commit -m "Initialize DVC"
```

---

# 9. Track a Dataset

Suppose:

```text
data/train.csv
```

Run:

```bash
dvc add data/train.csv
```

Then Git sees DVC metadata rather than treating the large dataset as a normal source file.

Commit the metadata:

```bash
git add data/train.csv.dvc .gitignore
git commit -m "Track training dataset with DVC"
```

---

# 10. Configure a Remote

Conceptually:

```bash
dvc remote add -d myremote <remote-location>
```

Then upload:

```bash
dvc push
```

To retrieve data later:

```bash
dvc pull
```

The mental model is:

```text
dvc add
   ↓
Track artifact locally

git commit
   ↓
Version the metadata

dvc push
   ↓
Upload artifact to remote

dvc pull
   ↓
Download artifact later
```

---

# 11. Pipeline Creation

Suppose training requires:

```bash
python src/train.py
```

A DVC stage can be created with a command such as:

```bash
dvc stage add -n train \
    -d src/train.py \
    -d data/processed/train.csv \
    -o models/model.pkl \
    python src/train.py
```

The exact command should be adapted to the project's paths and outputs.

DVC records the relationship:

```text
train.py
   +
processed data
   ↓
training command
   ↓
model.pkl
```

---

# 12. Why Pipeline Dependencies Matter

Imagine:

```text
raw data
   ↓
prepare.py
   ↓
processed data
   ↓
train.py
   ↓
model
```

If raw data changes, the processed data may need to change.

If processed data changes, training may need to run again.

This forms a dependency graph:

```text
Raw Data
   ↓
Preparation
   ↓
Processed Data
   ↓
Training
   ↓
Model
   ↓
Evaluation
```

DVC can use the pipeline definition and lock information to identify stages that need to be reproduced.

---

# 13. `dvc.yaml` and `dvc.lock`

## `dvc.yaml`

Think of this as the **pipeline definition**.

It describes what the pipeline is supposed to do.

## `dvc.lock`

Think of this as the **resolved state** of the pipeline.

It captures concrete information about the dependencies and outputs for a particular pipeline execution.

Together:

```text
dvc.yaml
   ↓
"What is the pipeline?"

dvc.lock
   ↓
"What exact dependency/output state was used?"
```

Both can be versioned in Git.

---

# 14. Reproducing a Pipeline

A typical workflow is:

```bash
git checkout <old-commit>
dvc pull
dvc repro
```

Conceptually:

```text
Checkout historical Git state
          ↓
Get corresponding DVC artifacts
          ↓
Reproduce pipeline
          ↓
Obtain historical result
```

This is one of DVC's most valuable properties for ML teams.

---

# 15. Parameters and Experiments

ML experiments often differ because of parameters such as:

```yaml
learning_rate: 0.01
epochs: 20
batch_size: 64
```

A parameter file can be versioned alongside the pipeline.

This helps connect:

```text
Git revision
      +
data version
      +
pipeline definition
      +
parameters
      ↓
experiment result
```

That makes experiments easier to reproduce and compare.

---

# 16. DVC vs Git

| Capability | Git | DVC |
|---|---|---|
| Source code | Excellent | Not its primary purpose |
| Large datasets | Poor fit | Designed for this |
| Model artifacts | Limited/awkward | Designed to track them |
| Pipeline dependencies | Not ML-focused | Yes |
| Git integration | Native | Integrates with Git |
| Large-file remote storage | Not the core workflow | Yes |

### Key interview point

**DVC complements Git rather than replacing it.**

---

# 17. DVC vs Git LFS

Git LFS also addresses large files.

The distinction depends on the team's workflow and requirements.

A useful mental model:

```text
Git LFS → large files in a Git-oriented workflow

DVC → data/model versioning + reproducible ML pipelines
```

DVC is especially attractive when data dependencies, pipelines, and reproducibility are central requirements.

---

# 18. DVC vs MLflow

These tools solve different but complementary problems.

| Tool | Primary Focus |
|---|---|
| DVC | Data/model versioning and reproducible pipelines |
| MLflow | Experiment tracking, model lifecycle/registry capabilities, and ML workflow management |

A production ML platform may use both:

```text
DVC
 ├── data versioning
 └── pipeline reproducibility

MLflow
 ├── metrics
 ├── parameters
 ├── experiment tracking
 └── model lifecycle management
```

---

# 19. Real-World MLOps Architecture

A simplified workflow:

```text
                 Git
                  │
       ┌──────────┴──────────┐
       │                     │
   Source Code          DVC Metadata
                             │
                             ↓
                        DVC Remote
                             │
                       ┌─────┴─────┐
                       │           │
                    Dataset      Models

                  Pipeline
                     │
                     ↓
              Training Compute
                     │
             ┌───────┴────────┐
             ↓                ↓
          Metrics           Model
             │                │
             └───────┬────────┘
                     ↓
              Experiment/Model
                 Management
```

In a broader MLOps stack, DVC may coexist with CI/CD, experiment tracking, orchestration, model serving, monitoring, and cloud infrastructure.

---

# 20. Common Problems and Debugging

## Problem: Teammate cannot obtain the dataset

### Diagnosis

Check:

```bash
dvc remote list
```

and whether the artifact exists in the remote.

### Fix

Configure the correct remote and run:

```bash
dvc pull
```

---

## Problem: Pipeline does not rerun

### Possible causes

- Dependencies did not change.
- DVC does not see a dependency change.
- The stage definition is incorrect.
- Outputs or parameters were not declared properly.

### Prevention

Make dependencies, parameters, and outputs explicit.

---

## Problem: Huge files accidentally committed to Git

### Diagnosis

Check Git status and repository history.

### Prevention

Add large artifacts to DVC before committing them to Git.

---

# 21. Common Mistakes

### Mistake 1: Thinking DVC replaces Git

Correct model:

```text
Git + DVC
```

not:

```text
Git OR DVC
```

### Mistake 2: Tracking data but not pipeline dependencies

Versioning a dataset alone does not automatically describe the entire ML computation.

### Mistake 3: No remote backup

Local DVC tracking is not sufficient for team collaboration if other users cannot access the artifacts.

### Mistake 4: Not versioning configuration

Parameters and pipeline definitions are part of reproducibility.

---

# 22. When to Use DVC

## Use it when

- Datasets are large.
- Models are large.
- Reproducibility matters.
- Multiple dataset versions must be maintained.
- ML pipelines need dependency tracking.
- Teams need to reproduce historical experiments.
- Git should remain lightweight.

## Consider alternatives or simpler workflows when

- The dataset is tiny and can safely live in Git.
- The project is a very small experiment where reproducibility requirements are minimal.
- Another platform already provides the required data/versioning workflow.

---

# 23. Production Perspective

### Notebook-level workflow

```text
Download data
↓
Train model
↓
Save model.pkl
```

### More production-oriented workflow

```text
Version code
↓
Version data
↓
Define pipeline
↓
Version parameters
↓
Run reproducible training
↓
Track metrics
↓
Validate artifacts
↓
Register/deploy model
↓
Monitor
↓
Retrain when appropriate
```

DVC addresses an important part of this lifecycle, especially **data/model artifacts and reproducible pipelines**. It is not by itself a complete MLOps platform.

---

# 24. End-to-End Mini Project

## Problem

Build a reproducible customer-churn model.

## Pipeline

```text
raw/customer.csv
        ↓
prepare.py
        ↓
processed/customer.csv
        ↓
train.py
        ↓
models/model.pkl
        ↓
evaluate.py
        ↓
metrics.json
```

## Suggested structure

```text
churn-project/
├── data/
│   ├── raw/
│   └── processed/
├── src/
│   ├── prepare.py
│   ├── train.py
│   └── evaluate.py
├── models/
├── params.yaml
├── dvc.yaml
├── dvc.lock
└── README.md
```

## Learning exercise

Implement the pipeline yourself.

Then test reproducibility:

1. Commit version A.
2. Change a parameter.
3. Run the pipeline.
4. Commit version B.
5. Return to version A.
6. Pull the corresponding data.
7. Reproduce version A.

The goal is to experience why data + code + parameters + pipeline definitions must be versioned together.

---

# 25. Interview Questions

## Beginner

1. What is DVC?
2. Why is DVC used in ML projects?
3. Why isn't Git alone always sufficient?
4. What is `dvc add`?
5. What is a DVC remote?
6. What does `dvc push` do?
7. What does `dvc pull` do?
8. What is `dvc.yaml`?
9. What is `dvc.lock`?
10. How does DVC integrate with Git?

## Intermediate

1. How does DVC improve reproducibility?
2. How are DVC metadata and actual data separated?
3. How does a DVC pipeline determine dependencies?
4. What happens when a pipeline dependency changes?
5. DVC vs Git LFS?
6. DVC vs MLflow?
7. How would you share DVC artifacts with a team?
8. How would you reproduce an old model?
9. How would you structure DVC in CI/CD?
10. What should and should not be committed to Git?

## Advanced / Scenario-Based

1. Your model's accuracy dropped after a new dataset release. How would DVC help investigate?
2. A teammate can access Git but cannot reproduce training. What would you inspect?
3. How would you design DVC storage for a multi-team organization?
4. How would you handle access control for the DVC remote?
5. How would you integrate DVC with CI/CD?
6. How would you prevent accidental use of an unversioned dataset?
7. How would you reproduce a six-month-old production model?
8. How would DVC fit into a cloud-native MLOps platform?
9. What are the limitations of DVC?
10. When would you choose another data-versioning approach?

---

# 26. Short Interview Answers

### What is DVC?

> DVC is a data and ML artifact versioning tool that integrates with Git to help teams version large datasets and models and build reproducible ML pipelines.

### Why do we need DVC?

> ML reproducibility depends not only on code but also on data, models, parameters, and pipeline dependencies. DVC helps version and reproduce those artifacts while keeping large files outside the normal Git history.

### DVC vs Git?

> Git primarily versions source code and text-based project files. DVC complements Git by managing large ML artifacts and data dependencies.

### DVC vs MLflow?

> DVC focuses strongly on data/model versioning and reproducible pipelines, while MLflow focuses on experiment tracking and model lifecycle capabilities. They can be used together.

---

# 27. Cheat Sheet

## Important commands

```bash
dvc init
dvc add <path>
dvc status
dvc repro
dvc push
dvc pull
dvc remote add -d <name> <location>
```

## Mental model

```text
Git
  ↓
Code + DVC metadata

DVC Remote
  ↓
Large data/model artifacts

DVC Pipeline
  ↓
Dependencies + commands + outputs
```

## Remember

- DVC complements Git.
- Large artifacts can live outside Git.
- `.dvc` metadata can be committed to Git.
- `dvc.yaml` describes pipeline stages.
- `dvc.lock` records concrete pipeline state.
- `dvc push` sends artifacts to the remote.
- `dvc pull` retrieves artifacts.
- `dvc repro` reproduces the pipeline.

---

# 28. Flashcards

**Q:** What does DVC stand for?  
**A:** Data Version Control.

**Q:** Why use DVC?  
**A:** To version large ML artifacts and improve reproducibility.

**Q:** Does DVC replace Git?  
**A:** No. DVC commonly works alongside Git.

**Q:** What does `dvc add` do?  
**A:** Starts tracking a data/model artifact with DVC.

**Q:** What does `dvc push` do?  
**A:** Uploads DVC-tracked artifacts to the configured remote.

**Q:** What does `dvc pull` do?  
**A:** Retrieves DVC-tracked artifacts from the remote.

**Q:** What is `dvc.yaml`?  
**A:** A definition of the DVC pipeline.

**Q:** What is `dvc.lock`?  
**A:** A record of the concrete dependency/output state of the pipeline.

**Q:** What is a DVC remote?  
**A:** Storage used for DVC-managed artifacts.

**Q:** What does reproducibility mean in ML?  
**A:** Being able to recreate a result using the corresponding code, data, configuration, and computation.

---

# 29. Knowledge Check

Try these before looking at the answers.

### Beginner

1. Why can a 20 GB dataset be problematic in Git?
2. What is the relationship between Git and DVC?
3. What command starts tracking a dataset?
4. What is a DVC remote?
5. What command downloads tracked artifacts?

### Intermediate

6. Why are pipeline dependencies important?
7. What is the purpose of `dvc.yaml`?
8. What is the purpose of `dvc.lock`?
9. Why should parameters be versioned?
10. How would you reproduce an old training run?

### Advanced

11. A model changed but the code did not. What data-related questions would you ask?
12. How would you integrate DVC into CI/CD?
13. What happens if the DVC remote becomes unavailable?
14. How would you secure shared artifact storage?
15. What limitations would you consider before adopting DVC?

## Answer Key

1. Git is not designed as the primary storage mechanism for huge datasets.
2. Git tracks code/configuration and DVC metadata; DVC manages large ML artifacts and pipeline relationships.
3. `dvc add`.
4. Storage for DVC-managed artifacts.
5. `dvc pull`.
6. They define what inputs affect outputs and therefore what must be rerun.
7. To define the pipeline stages and their relationships.
8. To record concrete dependency/output state for a pipeline.
9. Different parameters can produce different models even with the same code/data.
10. Checkout the historical Git revision, obtain its DVC artifacts, then reproduce the pipeline.
11. Check dataset version, feature generation, parameters, dependency changes, and training environment.
12. Validate data/pipeline state in CI and reproduce training or validation stages automatically as appropriate.
13. Existing local artifacts may still work, but new machines/CI jobs may be unable to fetch required data.
14. Use appropriate identity/access controls, secure credentials, encryption, auditing, and least-privilege access.
15. Storage architecture, operational complexity, team adoption, integration requirements, and whether another platform already solves the problem.

---

# 30. Learning Roadmap

After DVC, a useful progression is:

```text
Git
 ↓
DVC fundamentals
 ↓
DVC pipelines
 ↓
Experiment tracking
 ↓
MLflow
 ↓
CI/CD for ML
 ↓
Docker
 ↓
Model serving
 ↓
Monitoring
 ↓
Kubernetes
 ↓
End-to-end MLOps system design
```

## Recommended next project

Build a churn or house-price ML pipeline with:

- Git
- DVC
- Scikit-learn
- MLflow
- Docker
- FastAPI
- CI/CD
- Basic model monitoring

The objective is to understand how each tool solves a different production problem.

---

# 31. If You Remember Only 5 Things

1. **DVC brings version-control ideas to large ML data and artifacts.**
2. **DVC complements Git rather than replacing it.**
3. **Git can store DVC metadata while large artifacts live in a DVC remote.**
4. **DVC pipelines connect data, code, parameters, commands, and outputs for reproducibility.**
5. **DVC is one component of MLOps, not the entire MLOps platform.**

## 1-Minute Interview Explanation

DVC, or Data Version Control, is a tool used in ML projects to version large datasets, models, and pipeline artifacts while integrating with Git. Git stores the source code and DVC metadata, while the actual large artifacts can be stored in a remote backend. DVC can also define reproducible pipelines through dependencies, commands, parameters, and outputs. This helps teams reproduce historical training runs and understand exactly which data and code produced a model.

## 5-Minute Interview Explanation

The central challenge DVC addresses is that ML reproducibility involves more than source code. A model depends on datasets, preprocessing, parameters, generated features, training code, and sometimes other artifacts. Git is excellent for source code but is not usually the ideal mechanism for storing huge datasets and model binaries.

DVC integrates with Git so that the Git repository can represent the versioned state of the ML project while DVC manages large artifacts and their relationships. DVC remotes provide storage for those artifacts, and DVC pipelines can describe how inputs are transformed into outputs. By versioning pipeline definitions, parameters, dependencies, and artifact metadata, teams can reproduce previous states of an ML workflow.

In a production environment, DVC can work alongside tools such as experiment tracking, CI/CD, model serving, orchestration, and monitoring. Its value is strongest when data lineage and reproducibility are important.

---

# 32. Final Mental Model

When you hear **DVC**, think:

```text
             ML REPRODUCIBILITY
                    │
        ┌───────────┼───────────┐
        ↓           ↓           ↓
      Code        Data       Parameters
        │           │           │
        └───────────┼───────────┘
                    ↓
                 Pipeline
                    ↓
                  Model
                    ↓
             Reproducible Result

Git → version project state
DVC → version/manage ML artifacts + pipelines
Remote → store large artifacts
```

The key question to ask is:

> **"Can another engineer reproduce this exact model from a known project state?"**

If the answer is no, your ML workflow probably has a reproducibility gap.
