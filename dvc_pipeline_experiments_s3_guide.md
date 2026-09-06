# DVC Pipeline, Experiments & S3 Remote Storage

A practical step-by-step guide for building an ML pipeline with **Git, DVC, DVC Experiments, DVCLive, and AWS S3**.

> **Security warning:** AWS access credentials were included in the source notes. They are intentionally **not included in this document**. If those credentials are real and active, revoke/rotate them immediately in AWS IAM and create new credentials. Never commit AWS keys to Git, `params.yaml`, or any source file.

---

# Part 1 — Building the ML Pipeline

## 1. Create a GitHub Repository

Create a GitHub repository and clone it locally.

```bash
git clone <repository-url>
cd <repository-name>
```

For example, the repository can contain your ML experiments and source code.

---

## 2. Create the `src` Directory

Add a `src` folder containing the individual components of your ML workflow.

Example:

```text
project/
├── src/
│   ├── data_ingestion.py
│   ├── feature_engineering.py
│   ├── model_building.py
│   └── model_evaluation.py
├── data/
├── models/
└── reports/
```

Each component should be runnable independently before connecting everything into a DVC pipeline.

### Recommended approach

Test each component individually:

```bash
python src/data_ingestion.py
python src/feature_engineering.py
python src/model_building.py
python src/model_evaluation.py
```

Make sure each script works correctly before creating the DVC pipeline.

---

## 3. Add Generated Directories to `.gitignore`

Add the following directories to `.gitignore`:

```gitignore
data/
models/
reports/
```

### Why?

These directories can contain generated ML artifacts such as:

- Raw/processed datasets
- Trained models
- Evaluation reports
- Generated outputs

DVC can manage the relevant data/artifacts while Git tracks the pipeline metadata and source code.

---

## 4. Initial Git Commit

Save the initial project structure:

```bash
git add .
git commit -m "initial project setup"
git push
```

---

# Part 2 — Setting Up a DVC Pipeline Without Parameters

## 5. Create `dvc.yaml`

Create a `dvc.yaml` file and define the pipeline stages.

Example:

```yaml
stages:

  data_ingestion:
    cmd: python src/data_ingestion.py
    deps:
      - src/data_ingestion.py
    outs:
      - data/raw

  feature_engineering:
    cmd: python src/feature_engineering.py
    deps:
      - src/feature_engineering.py
      - data/raw
    outs:
      - data/processed

  model_building:
    cmd: python src/model_building.py
    deps:
      - src/model_building.py
      - data/processed
    outs:
      - models/model.pkl

  model_evaluation:
    cmd: python src/model_evaluation.py
    deps:
      - src/model_evaluation.py
      - models/model.pkl
    outs:
      - reports/metrics.json
```

> Adjust the commands, dependencies, and outputs according to your project.

---

## 6. Initialize DVC and Run the Pipeline

Initialize DVC:

```bash
dvc init
```

Run the pipeline:

```bash
dvc repro
```

`dvc repro` checks the pipeline dependencies and executes the stages that need to be reproduced.

### Visualize the Pipeline

Use:

```bash
dvc dag
```

This displays the **Directed Acyclic Graph (DAG)** of the pipeline.

Example:

```text
data_ingestion
       │
       ▼
feature_engineering
       │
       ▼
model_building
       │
       ▼
model_evaluation
```

The DAG makes it easier to understand the order and dependencies between pipeline stages.

---

## 7. Commit the Pipeline to Git

After verifying that the pipeline works:

```bash
git add .
git commit -m "add DVC pipeline"
git push
```

---

# Part 3 — Setting Up a DVC Pipeline with Parameters

## 8. Create `params.yaml`

Create a file called:

```text
params.yaml
```

This file stores configurable ML parameters separately from the Python code.

Example:

```yaml
data_ingestion:
  test_size: 0.2

feature_engineering:
  max_features: 5000

model_building:
  n_estimators: 100
  max_depth: 10
```

---

## 9. Load Parameters in Python

Import YAML:

```python
import yaml
```

Create a parameter-loading function:

```python
def load_params(params_path: str) -> dict:
    """Load parameters from a YAML file."""
    try:
        with open(params_path, 'r') as file:
            params = yaml.safe_load(file)

        logger.debug('Parameters retrieved from %s', params_path)
        return params

    except FileNotFoundError:
        logger.error('File not found: %s', params_path)
        raise

    except yaml.YAMLError as e:
        logger.error('YAML error: %s', e)
        raise

    except Exception as e:
        logger.error('Unexpected error: %s', e)
        raise
```

---

## 10. Use Parameters in `main()`

### Data Ingestion

```python
params = load_params(params_path='params.yaml')
test_size = params['data_ingestion']['test_size']
```

### Feature Engineering

```python
params = load_params(params_path='params.yaml')
max_features = params['feature_engineering']['max_features']
```

### Model Building

```python
params = load_params('params.yaml')['model_building']
```

---

## 11. Reproduce the Pipeline

Run:

```bash
dvc repro
```

DVC now considers the parameters when determining whether a stage needs to be reproduced.

For example, if you change:

```yaml
data_ingestion:
  test_size: 0.2
```

to:

```yaml
data_ingestion:
  test_size: 0.3
```

and run:

```bash
dvc repro
```

DVC can identify the affected stage and reproduce the necessary downstream stages.

---

## 12. Commit the Parameterized Pipeline

```bash
git add .
git commit -m "add pipeline parameters"
git push
```

---

# Part 4 — DVC Experiments with DVCLive

DVC Experiments allow you to try different parameter combinations without creating a permanent Git commit for every experiment.

---

## 13. Install DVCLive

Install:

```bash
pip install dvclive
```

---

## 14. Add DVCLive to the Project

Import:

```python
from dvclive import Live
import yaml
```

Use the `load_params()` function described earlier and load the parameters in `main()`.

---

## 15. Log Metrics with DVCLive

Add the following structure inside `main()`:

```python
with Live(save_dvc_exp=True) as live:

    live.log_metric('accuracy', accuracy_score(y_test, y_pred))
    live.log_metric('precision', precision_score(y_test, y_pred))
    live.log_metric('recall', recall_score(y_test, y_pred))

    live.log_params(params)
```

> **Important:** The original notes used `y_test` as both arguments to `accuracy_score`, `precision_score`, and `recall_score`. Normally these functions should receive the true labels and the model predictions, such as `y_test` and `y_pred`.

---

## 16. Run a DVC Experiment

Run:

```bash
dvc exp run
```

DVC creates an experiment based on the current pipeline and parameters.

DVCLive can create a directory containing experiment metrics and related information.

Depending on the project configuration, you may see files/directories such as:

```text
dvclive/
├── metrics.json
├── params.yaml
└── ...
```

Each `dvc exp run` can represent a separate experiment.

---

## 17. View Experiments

Use:

```bash
dvc exp show
```

This displays experiment results in the terminal.

You can compare things such as:

```text
Experiment      Accuracy    Precision    Recall
------------------------------------------------
baseline        0.82        0.80         0.78
exp-1           0.85        0.83         0.82
exp-2           0.87        0.86         0.84
```

You can also use the DVC extension for VS Code if you prefer a graphical interface.

---

## 18. Install the DVC VS Code Extension

Install the DVC extension from the VS Code Extensions marketplace.

It can help you visualize and manage DVC-related workflows and experiments.

---

## 19. Remove an Experiment

To remove an experiment:

```bash
dvc exp remove <exp-name>
```

Example:

```bash
dvc exp remove exp-123456
```

This step is optional.

---

## 20. Apply an Existing Experiment

If you want to apply the results/changes from an existing experiment:

```bash
dvc exp apply <exp-name>
```

Example:

```bash
dvc exp apply exp-123456
```

This is useful when you find an experiment configuration that you want to continue working with.

---

## 21. Run More Experiments

Change values in `params.yaml`.

For example:

```yaml
model_building:
  n_estimators: 200
  max_depth: 15
```

Then run:

```bash
dvc exp run
```

Try another configuration:

```yaml
model_building:
  n_estimators: 300
  max_depth: 20
```

Run again:

```bash
dvc exp run
```

Now you can compare the experiments:

```bash
dvc exp show
```

---

## 22. Save the Final Experiment

Once you find an experiment configuration that you want to keep permanently:

```bash
git add .
git commit -m "save best experiment"
git push
```

> Before committing, review exactly what changed. Temporary experiment outputs should not accidentally be committed to Git.

---

# Part 5 — Adding AWS S3 as a DVC Remote

DVC can store datasets and ML artifacts in cloud storage.

AWS S3 is one option.

---

## 23. Log in to AWS

Go to the AWS Console and sign in.

---

## 24. Create an IAM User

Create an IAM user with only the permissions required for the DVC S3 workflow.

### Security best practice

Avoid giving broad administrator permissions just to make DVC work.

Use the principle of **least privilege**.

---

## 25. Create an S3 Bucket

Create an S3 bucket with a globally unique bucket name.

Example:

```text
dvc-s3-first-mlops-proj
```

Your actual bucket name may be different.

---

## 26. Install DVC with S3 Support

Install the S3 dependency:

```bash
pip install "dvc[s3]"
```

Also install AWS CLI:

```bash
pip install awscli
```

Alternatively, install AWS CLI using the official AWS installation method for your operating system.

---

## 27. Configure AWS CLI

Run:

```bash
aws configure
```

AWS CLI will ask for credentials and configuration such as:

```text
AWS Access Key ID:
AWS Secret Access Key:
Default region name:
Default output format:
```

### Never put credentials in Git

Do **not** put credentials directly inside:

```text
dvc.yaml
params.yaml
code.py
.gitignore
README.md
```

Do not commit files containing AWS secrets.

Prefer AWS credential configuration, environment variables, IAM roles, or other secure credential mechanisms appropriate for your environment.

---

## 28. Add S3 as the DVC Remote

Configure DVC:

```bash
dvc remote add -d dvcstore s3://<bucket-name>
```

For the example bucket:

```bash
dvc remote add -d dvcstore s3://dvc-s3-first-mlops-proj
```

Where:

```text
dvcstore
    ↓
DVC remote name

s3://dvc-s3-first-mlops-proj
    ↓
AWS S3 bucket used as remote storage
```

---

## 29. Push the Experiment/Data to S3

After deciding which experiment or data version you want to keep:

```bash
dvc commit
dvc push
```

This stores the corresponding DVC-tracked data/artifacts in the configured S3 remote.

Then commit the Git metadata:

```bash
git add .
git commit -m "configure DVC S3 remote"
git push
```

---

# Part 6 — Let DVC Generate `dvc.yaml`

Instead of manually writing every stage in `dvc.yaml`, DVC can generate a stage for you.

## 30. Add a Stage Using `dvc stage add`

Example:

```bash
dvc stage add \
  -n data_ingestion \
  -d src/data_ingestion.py \
  -o data/raw \
  python src/data_ingestion.py
```

### Breakdown

```text
dvc stage add
```

Create a DVC pipeline stage.

```text
-n data_ingestion
```

Stage name.

```text
-d src/data_ingestion.py
```

Dependency of the stage.

```text
-o data/raw
```

Output produced by the stage.

```text
python src/data_ingestion.py
```

Command executed by the stage.

DVC will generate/update the corresponding `dvc.yaml` configuration.

---

# Part 7 — Typical DVC Project Structure

A complete project can look like:

```text
mlops-project/
│
├── .dvc/
├── .dvcignore
├── .git/
├── .gitignore
│
├── dvc.yaml
├── params.yaml
├── dvc.lock
│
├── src/
│   ├── data_ingestion.py
│   ├── feature_engineering.py
│   ├── model_building.py
│   └── model_evaluation.py
│
├── data/
│   ├── raw/
│   └── processed/
│
├── models/
│   └── model.pkl
│
├── reports/
│   └── metrics.json
│
└── dvclive/
    ├── metrics.json
    └── params.yaml
```

---

# Part 8 — Complete Workflow

## Initial Project Setup

```bash
git clone <repository-url>
cd <repository-name>

mkdir src

# Create and test source files

# Add generated directories to .gitignore
echo "data/" >> .gitignore
echo "models/" >> .gitignore
echo "reports/" >> .gitignore

git add .
git commit -m "initial project setup"
git push
```

---

## Initialize DVC

```bash
dvc init
```

---

## Create Pipeline

Either manually create `dvc.yaml` or use:

```bash
dvc stage add \
  -n data_ingestion \
  -d src/data_ingestion.py \
  -o data/raw \
  python src/data_ingestion.py
```

Then reproduce:

```bash
dvc repro
```

Visualize:

```bash
dvc dag
```

---

## Add Parameters

Create:

```text
params.yaml
```

Load them in Python:

```python
params = load_params('params.yaml')
```

Then run:

```bash
dvc repro
```

---

## Run Experiments

Install DVCLive:

```bash
pip install dvclive
```

Run:

```bash
dvc exp run
```

View:

```bash
dvc exp show
```

Remove an experiment:

```bash
dvc exp remove <exp-name>
```

Apply an experiment:

```bash
dvc exp apply <exp-name>
```

---

## Configure S3

```bash
pip install "dvc[s3]"
pip install awscli
aws configure
```

Add the remote:

```bash
dvc remote add -d dvcstore s3://<bucket-name>
```

Push data:

```bash
dvc commit
dvc push
```

Finally:

```bash
git add .
git commit -m "save DVC pipeline configuration"
git push
```

---

# Git vs DVC in This Workflow

| Tool | Responsibility |
|---|---|
| **Git** | Source code and project metadata |
| **GitHub** | Remote Git repository |
| **DVC** | Dataset/artifact versioning |
| **dvc.yaml** | Pipeline definition |
| **dvc.lock** | Exact pipeline state/dependency information |
| **params.yaml** | Configurable ML parameters |
| **DVC Experiments** | Experiment tracking/versioning |
| **DVCLive** | Metrics and parameter logging |
| **AWS S3** | Remote storage for DVC-tracked data |

---

# Key Commands Cheat Sheet

## Git

```bash
git add .
git commit -m "message"
git push
git status
git log --oneline
```

## DVC Pipeline

```bash
dvc init
dvc repro
dvc dag
dvc status
```

## DVC Stages

```bash
dvc stage add -n <stage-name> -d <dependency> -o <output> <command>
```

## Parameters

```bash
dvc repro
```

## DVC Experiments

```bash
dvc exp run
dvc exp show
dvc exp remove <exp-name>
dvc exp apply <exp-name>
```

## DVC Remote

```bash
dvc remote add -d dvcstore s3://<bucket-name>
dvc commit
dvc push
dvc pull
```

---

# Mental Model

Think about the entire ML project like this:

```text
                    Git / GitHub
                         │
                         │
                Code + Pipeline
                         │
                         ▼
                    dvc.yaml
                         │
                         ▼
                    DVC Pipeline
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
       Ingestion    Feature Eng.   Model Building
          │              │              │
          └──────────────┼──────────────┘
                         ▼
                   Model Evaluation
                         │
                         ▼
                    DVCLive
                         │
                         ▼
                  Experiments
                         │
                         ▼
                    Best Version
                         │
                         ▼
                      DVC Push
                         │
                         ▼
                       AWS S3
```

### Simple rule to remember

```text
Git      → Code + pipeline definitions + metadata
DVC      → Data + ML artifacts
DVCLive  → Metrics + parameters
S3       → Remote storage for DVC
```

---

# Important Security Note

Never store AWS credentials in your Git repository.

If credentials have already been exposed publicly or committed to Git:

1. **Revoke the exposed access key immediately.**
2. Create a new access key only if needed.
3. Review IAM permissions.
4. Check CloudTrail/billing for unexpected activity.
5. Remove secrets from the repository history if they were committed.
6. Add sensitive files to `.gitignore` where appropriate.
7. Use secure credential mechanisms instead of hardcoding keys.

Example of what **not** to do:

```python
AWS_ACCESS_KEY = "AKIA..."
AWS_SECRET_KEY = "..."
```

Instead, use AWS's credential configuration mechanisms or environment-based credentials.
