# Git + DVC Data Versioning Workflow

A practical step-by-step guide for using **Git** for code versioning and **DVC (Data Version Control)** for dataset versioning.

---

## 1. Create a Git Repository

Create a new repository on GitHub and clone it locally.

```bash
git clone <repository-url>
cd <repository-name>
```

---

## 2. Create the Python Code

Create a `code.py` file and add your code.

The code will generate/save a CSV file inside a new `data` folder.

Example structure:

```text
project/
├── code.py
└── data/
    └── data.csv
```

---

## 3. Track the Initial Code with Git

First, save the initial version of the project using Git.

```bash
git add .
git commit -m "initial commit"
git push
```

---

# Set Up DVC

## 4. Install DVC

Install DVC using pip:

```bash
pip install dvc
```

Initialize DVC in the Git repository:

```bash
dvc init
```

This creates DVC-related files such as:

```text
.dvc/
.dvcignore
```

> **Important:** DVC works together with Git. Git tracks code and metadata, while DVC tracks large data files.

---

## 5. Create a Local Storage Directory

For this example, create a local directory named `S3`.

```bash
mkdir S3
```

This directory will act as our **local DVC remote storage**.

> When working with AWS, the `S3` location can instead be an actual AWS S3 bucket URL.

---

## 6. Configure the DVC Remote

Add the remote storage:

```bash
dvc remote add -d myremote S3
```

### What does this mean?

```text
-d
```

Means this remote will be the **default DVC remote**.

```text
myremote
```

This is the name of the remote.

You can choose another name, for example:

```bash
dvc remote add -d storage S3
```

```text
S3
```

This is the location of the remote storage.

For AWS, this could be an S3 URL such as:

```text
s3://my-bucket/my-data
```

---

# Start Tracking Data with DVC

## 7. Add the Data Directory to DVC

Initially, Git is already tracking the `data/` directory.

Now tell DVC to track it:

```bash
dvc add data/
```

Because Git is already tracking `data/`, DVC will require us to stop Git from tracking the actual data files.

Run:

```bash
git rm -r --cached data
git commit -m "stop tracking data"
```

### Why do we do this?

Initially:

```text
Git → tracks data/
```

After switching to DVC:

```text
Git → tracks code + DVC metadata
DVC → tracks actual data
```

This is important because we don't want Git and DVC both tracking the same dataset.

---

## 8. Add the Data to DVC Again

Now run:

```bash
dvc add data/
```

DVC will create:

```text
data.dvc
```

It will also update `.gitignore` so that Git ignores the actual data directory.

The project may now look like:

```text
project/
├── .dvc/
├── .dvcignore
├── .gitignore
├── code.py
├── data/
│   └── data.csv
└── data.dvc
```

### What does `data.dvc` contain?

`data.dvc` is a small metadata file that tells DVC how to identify and manage the dataset.

Git tracks this file, while DVC manages the actual dataset.

Now commit the DVC metadata:

```bash
git add .
git commit -m "add data to DVC"
git push
```

---

# Store the Data in the DVC Remote

## 9. Run `dvc commit` and `dvc push`

Commit the current data state:

```bash
dvc commit
```

Then push the data to the configured DVC remote:

```bash
dvc push
```

At this point:

```text
GitHub
   │
   ├── code.py
   ├── data.dvc
   ├── .gitignore
   └── DVC metadata

DVC Remote
   │
   └── actual dataset
```

---

# Create Version 2 of the Dataset

## 10. Modify the Code and Generate New Data

Make changes to `code.py`.

For example, append a new row to the CSV file.

Run your Python code:

```bash
python code.py
```

Now check whether DVC detects a change:

```bash
dvc status
```

DVC should report that the data has changed.

---

## 11. Commit and Push the Updated Data

Once the data has changed:

```bash
dvc commit
dvc push
```

### What happened?

The new dataset version has been stored in the DVC remote.

---

## 12. Save Version 2 in Git

Now commit the updated DVC metadata using Git:

```bash
git add .
git commit -m "update dataset to v2"
git push
```

Think of this as saving a **version 2 checkpoint**.

The important distinction is:

```text
Git:
    Saves the code + DVC metadata

DVC:
    Saves the actual dataset
```

Both are required for proper data versioning.

---

## 13. Check Git and DVC Status

Check Git:

```bash
git status
```

Check DVC:

```bash
dvc status
```

Ideally, everything should be up to date.

You should see something similar to:

```text
Git:
nothing to commit, working tree clean

DVC:
Data and pipelines are up to date.
```

---

# Create Version 3

## 14. Repeat the Process

Now repeat the same workflow whenever the dataset changes.

### Step 1 — Modify the data

Update `code.py` and generate the new data:

```bash
python code.py
```

### Step 2 — Check DVC

```bash
dvc status
```

### Step 3 — Commit the new dataset version

```bash
dvc commit
```

### Step 4 — Push the dataset

```bash
dvc push
```

### Step 5 — Commit the DVC metadata with Git

```bash
git add .
git commit -m "update dataset to v3"
git push
```

Now we have:

```text
Version 1
    ↓
Version 2
    ↓
Version 3
```

Each Git commit points to the corresponding DVC dataset version.

---

# Versioning Benefit of DVC

## Retrieve an Older Version of the Dataset

Suppose we want to retrieve **Version 1** of our dataset.

First, find the Git commit:

```bash
git log --oneline
```

Example:

```text
a8f31d2 update dataset to v3
b72c9e1 update dataset to v2
06997c7 initial dataset version
```

Suppose the Version 1 commit ID is:

```text
06997c7
```

---

## Step 1 — Checkout the Old Git Commit

```bash
git checkout 06997c7
```

This moves the project back to that Git version.

---

## Step 2 — Pull the Corresponding DVC Data

Now run:

```bash
dvc pull
```

DVC reads the `data.dvc` file associated with that Git commit and downloads the corresponding dataset from the DVC remote.

You have now retrieved the **Version 1 dataset**.

---

## Step 3 — Return to the Latest Version

After inspecting the old version, return to the main branch:

```bash
git checkout master
```

If your repository uses `main` instead:

```bash
git checkout main
```

Then retrieve the latest dataset:

```bash
dvc pull
```

---

# Complete Workflow

The overall workflow can be remembered as:

```text
                CODE
                  │
                  ▼
             Git repository
                  │
          git add / commit / push
                  │
                  ▼
        ┌─────────────────────┐
        │                     │
        ▼                     ▼
      Code              DVC metadata
                            │
                            ▼
                         DVC Remote
                            │
                            ▼
                     Actual Dataset
```

### Normal development cycle

```bash
# Modify code/data
python code.py

# Check data changes
dvc status

# Save data version
dvc commit

# Upload data
dvc push

# Save DVC metadata
git add .
git commit -m "update dataset"
git push
```

---

# Git vs DVC

| Tool | Tracks | Example |
|---|---|---|
| **Git** | Code and small metadata files | `code.py`, `data.dvc` |
| **DVC** | Large datasets and ML artifacts | `data.csv`, model files |
| **GitHub** | Git repository | Code + Git history |
| **DVC Remote** | Actual dataset files | Dataset versions |

### Simple Mental Model

```text
Git = Version control for CODE

DVC = Version control for DATA

Git + DVC = Version control for ML projects
```

---

# Important Commands Cheat Sheet

## Git

```bash
git clone <url>
git add .
git commit -m "message"
git push
git status
git log --oneline
git checkout <commit-id>
git checkout main
```

## DVC

```bash
pip install dvc
dvc init
dvc remote add -d myremote S3
dvc add data/
dvc status
dvc commit
dvc push
dvc pull
```

---

# End-to-End Example

```bash
# Clone repository
git clone <repository-url>
cd <repository-name>

# Create code and data
python code.py

# Initial Git commit
git add .
git commit -m "initial commit"
git push

# Install and initialize DVC
pip install dvc
dvc init

# Create local remote
mkdir S3

# Configure DVC remote
dvc remote add -d myremote S3

# Start tracking data with DVC
dvc add data/

# Stop Git from tracking actual data
git rm -r --cached data
git commit -m "stop tracking data"

# Add data to DVC
dvc add data/

# Save DVC metadata
git add .
git commit -m "add data to DVC"
git push

# Store data in DVC remote
dvc commit
dvc push

# -------------------------
# Version 2
# -------------------------

python code.py
dvc status
dvc commit
dvc push

git add .
git commit -m "update dataset to v2"
git push

# -------------------------
# Retrieve an old version
# -------------------------

git log --oneline
git checkout 06997c7
dvc pull

# Return to latest version
git checkout main
dvc pull
```

> **Key takeaway:** Git tells you **which version of the data you need**, while DVC gives you the **actual data corresponding to that version**.
