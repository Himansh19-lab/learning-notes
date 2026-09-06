# DVC — Interactive Learning Guide

> A practical, step-by-step guide to learning **DVC (Data Version Control)** with Git.
>
> This guide focuses on the complete workflow: create a Git repository → generate data → introduce DVC → configure a remote → version data → create V2/V3 → verify state → restore an older data version.

---

# 1. What Problem Does DVC Solve?

When working on machine-learning projects, Git is excellent for versioning source code, but datasets can become large and change frequently.

Imagine your project produces:

```text
data.csv — Version 1
data.csv — Version 2
data.csv — Version 3
```

You want to answer:

- What did the dataset look like for this model?
- Which data version belongs to this Git commit?
- Can I reproduce an old experiment?
- Can I switch back to an earlier dataset?
- Can I keep large data outside the Git repository?

DVC provides a workflow for **versioning data alongside Git-managed source code**.

A useful mental model is:

```text
                 Git
                  │
        Code + DVC metadata
                  │
                  ▼
              data.dvc
                  │
                  │ points to
                  ▼
              DVC Remote
             /     |     \
          Data V1 Data V2 Data V3
```

---

# 2. Git + DVC: The Core Idea

The most important concept is understanding what each tool manages.

```text
Git
 │
 ├── Source code
 ├── Project configuration
 ├── data.dvc
 ├── .gitignore
 └── Git history
```

DVC manages the actual dataset contents:

```text
DVC
 │
 ├── Data files
 ├── Data versions
 └── Remote data storage
```

The relationship is:

```text
Git commit
     │
     ▼
data.dvc
     │
     ▼
DVC-tracked data version
```

This allows you to use Git history to identify the desired project state and DVC to retrieve the corresponding data.

---

# 3. Important DVC Concepts

| Concept | Meaning |
|---|---|
| DVC | Data Version Control |
| `dvc init` | Initializes DVC in a Git repository |
| `dvc add` | Starts tracking data with DVC |
| `data.dvc` | Lightweight metadata file describing the tracked data |
| DVC remote | Location where DVC stores data |
| `dvc commit` | Records the current state of DVC-tracked data |
| `dvc push` | Sends DVC-tracked data to the remote |
| `dvc pull` | Retrieves DVC-tracked data from the remote |
| `dvc status` | Checks whether tracked data has changed |
| `git checkout` | Moves the project to a particular Git commit |

---

# 4. Complete Learning Workflow

The complete workflow we will build is:

```text
1. Create Git repository
        ↓
2. Clone repository locally
        ↓
3. Create code.py
        ↓
4. Generate data/data.csv
        ↓
5. Git add → commit → push
        ↓
6. Install DVC
        ↓
7. dvc init
        ↓
8. Create DVC remote
        ↓
9. dvc remote add
        ↓
10. dvc add data/
        ↓
11. Stop Git from tracking data/
        ↓
12. dvc add data/ again
        ↓
13. Git add → commit → push
        ↓
14. dvc commit → dvc push
        ↓
15. Create Data V2
        ↓
16. dvc status
        ↓
17. dvc commit → dvc push
        ↓
18. Git add → commit → push
        ↓
19. Create Data V3
        ↓
20. Repeat versioning workflow
        ↓
21. Checkout old Git commit
        ↓
22. dvc pull
        ↓
23. Restore old data version
```

---

# 5. Step 1 — Create a Git Repository

Create a new repository on your Git hosting service.

Then clone it:

```bash
git clone <repository-url>
```

Move into the repository:

```bash
cd <repository-name>
```

Check the Git status:

```bash
git status
```

You should have a clean repository.

---

# 6. Step 2 — Create `code.py`

Create:

```text
code.py
```

The script will create a new `data` directory and save a CSV file inside it.

Example:

```python
from pathlib import Path
import csv

data_dir = Path("data")
data_dir.mkdir(exist_ok=True)

rows = [
    ["id", "name", "value"],
    [1, "A", 100],
    [2, "B", 200],
]

with open(data_dir / "data.csv", "w", newline="") as f:
    writer = csv.writer(f)
    writer.writerows(rows)

print("Created:", data_dir / "data.csv")
```

Run:

```bash
python code.py
```

The project should now look like:

```text
project/
│
├── code.py
│
└── data/
    └── data.csv
```

Check:

```bash
git status
```

---

# 7. Step 3 — First Git Commit

Initially, Git will see the generated data as a normal file.

Add everything:

```bash
git add .
```

Commit:

```bash
git commit -m "add initial project and data"
```

Push:

```bash
git push
```

At this point:

```text
Git
 │
 ├── code.py
 └── data/
      └── data.csv
```

Git is currently tracking the data.

This is what we will change by introducing DVC.

---

# 8. Step 4 — Install DVC

Install DVC:

```bash
pip install dvc
```

Verify the installation:

```bash
dvc --version
```

If the command works, DVC is available in your environment.

---

# 9. Step 5 — Initialize DVC

Inside the Git repository:

```bash
dvc init
```

DVC creates its configuration/metadata area and a `.dvcignore` file.

The project will now contain DVC-related files/directories.

Conceptually:

```text
project/
│
├── .dvc/
├── .dvcignore
├── code.py
└── data/
    └── data.csv
```

---

# 10. Step 6 — Create a DVC Remote

For this learning workflow, create a local directory:

```bash
mkdir S3
```

This is simply a local directory used as the DVC remote in this exercise.

Conceptually:

```text
project/
│
├── S3/
├── code.py
└── data/
```

> When working with AWS, the remote can instead point to an S3 bucket URL.

---

# 11. Step 7 — Configure the DVC Remote

Run:

```bash
dvc remote add -d myremote S3
```

Understand the command:

```text
-d
│
└── default remote

myremote
│
└── name of the remote

S3
│
└── remote location
```

`myremote` is simply a name. It can be changed.

For example:

```bash
dvc remote add -d myremote S3
```

means:

```text
Remote name:
    myremote

Remote location:
    S3

Default:
    yes
```

---

# 12. Step 8 — First `dvc add data/`

Now tell DVC to track the data:

```bash
dvc add data/
```

Because the `data/` directory was already tracked by Git, you first need to remove it from Git's index.

Use:

```bash
git rm -r --cached data
```

Important:

This does **not** mean "delete my local data."

It means:

```text
Stop tracking data/ with Git
```

The files can remain in your working directory.

Commit this Git change:

```bash
git commit -m "stop tracking data"
```

---

# 13. Step 9 — Run `dvc add data/` Again

Now run:

```bash
dvc add data/
```

DVC creates:

```text
data.dvc
```

and updates:

```text
.gitignore
```

The structure becomes approximately:

```text
project/
│
├── .dvc/
├── .dvcignore
├── .gitignore
├── code.py
├── data.dvc
│
├── data/
│   └── data.csv
│
└── S3/
```

The important change is:

```text
Before:

Git
 └── data/

After:

Git
 └── data.dvc

DVC
 └── data/
```

---

# 14. Why Does `data.dvc` Matter?

`data.dvc` is the lightweight metadata that allows Git and DVC to work together.

Think of it as a pointer/description of the DVC-tracked data state.

```text
Git commit
     │
     ▼
 data.dvc
     │
     ▼
data version
     │
     ▼
DVC remote
```

Git does not need to store the complete dataset contents in its normal repository history.

---

# 15. Step 10 — Add DVC Metadata to Git

After:

```bash
dvc add data/
```

check:

```bash
git status
```

Then:

```bash
git add .
```

Commit:

```bash
git commit -m "start tracking data with DVC"
```

Push:

```bash
git push
```

Now Git stores the project metadata needed to identify the DVC data version.

---

# 16. Step 11 — Commit Data with DVC

Run:

```bash
dvc commit
```

Then push the DVC-tracked data:

```bash
dvc push
```

This creates the important separation:

```text
git push
    ↓
Git repository

dvc push
    ↓
DVC remote
```

---

# 17. Git Push vs DVC Push

This distinction is extremely important.

## `git push`

```bash
git push
```

Pushes Git commits.

These can contain:

```text
code.py
data.dvc
.gitignore
.dvc configuration/metadata
```

## `dvc push`

```bash
dvc push
```

Pushes DVC-tracked data to the DVC remote.

So:

```text
git push
    = push Git history/metadata

dvc push
    = push actual DVC-tracked data
```

You typically need both for a reproducible project.

---

# 18. First Stable Version

At this point, think of the project as:

```text
Git
 │
 └── Commit A
       │
       └── data.dvc
              │
              ▼
           Data V1

DVC Remote
 │
 └── Data V1
```

This is your first version.

---

# 19. Step 12 — Create Data Version 2

Now modify `code.py`.

The goal is to append a new row.

For example:

```text
[3, "C", 300]
```

Run:

```bash
python code.py
```

Now the dataset has changed.

Check:

```bash
dvc status
```

DVC should report that the tracked data has changed.

The important concept is:

```text
Data V1
   ↓
code.py changes
   ↓
Data V2
```

---

# 20. Step 13 — Save Data V2 with DVC

Commit the changed DVC data state:

```bash
dvc commit
```

Push it:

```bash
dvc push
```

Now the remote has the new data version.

Conceptually:

```text
DVC Remote

Data V1
Data V2
```

---

# 21. Step 14 — Save V2 Metadata in Git

Now save the updated DVC metadata in Git:

```bash
git add .
```

Commit:

```bash
git commit -m "update dataset to v2"
```

Push:

```bash
git push
```

Now:

```text
Git
 │
 ├── Commit A → Data V1
 │
 └── Commit B → Data V2
```

And:

```text
DVC Remote
 │
 ├── Data V1
 └── Data V2
```

---

# 22. The Two-Part Versioning Pattern

For each dataset version, think:

```text
                 DATA VERSION
                      │
            ┌─────────┴─────────┐
            ▼                   ▼
        Git commit           DVC push
            │                   │
      data.dvc state       actual data
```

So when creating a new data version:

```bash
dvc commit
dvc push

git add .
git commit -m "update dataset to v2"
git push
```

---

# 23. Step 15 — Verify Everything

Check Git:

```bash
git status
```

Check DVC:

```bash
dvc status
```

The goal is:

```text
Git:
    clean

DVC:
    data is up to date
```

This means the local project is synchronized.

---

# 24. Step 16 — Create Data Version 3

Repeat the process.

Modify:

```text
code.py
```

Generate the new data:

```bash
python code.py
```

Check:

```bash
dvc status
```

Then:

```bash
dvc commit
dvc push
```

Save the metadata in Git:

```bash
git add .
git commit -m "update dataset to v3"
git push
```

Finally:

```bash
git status
dvc status
```

---

# 25. Your Version History

You now have:

```text
Git History

Commit A
   │
   └── data.dvc → Data V1

Commit B
   │
   └── data.dvc → Data V2

Commit C
   │
   └── data.dvc → Data V3
```

And:

```text
DVC Remote

Data V1
Data V2
Data V3
```

This is where DVC becomes powerful.

---

# 26. Versioning Benefit

Suppose six months later you need the first version of the dataset.

You know the Git commit:

```text
06997c7
```

You can inspect Git history:

```bash
git log --oneline
```

Example:

```text
a83f2c1 update dataset to v3
b7284d2 update dataset to v2
06997c7 initial data version
```

---

# 27. Step 17 — Restore Data Version 1

Checkout the old Git commit:

```bash
git checkout 06997c7
```

Now the repository is at the state represented by that Git commit.

The DVC metadata also corresponds to that historical state.

Retrieve the associated data:

```bash
dvc pull
```

Conceptually:

```text
git checkout 06997c7
          │
          ▼
 historical data.dvc
          │
          ▼
       Data V1
          │
          ▼
      dvc pull
          │
          ▼
 data/data.csv = V1
```

---

# 28. Why `git checkout` + `dvc pull`?

This is one of the most important DVC concepts.

Git tells you:

```text
Which project state do I want?
```

DVC tells you:

```text
Which data contents belong to that project state?
```

Therefore:

```bash
git checkout <commit>
dvc pull
```

means:

```text
1. Move Git to the desired project version.
2. Read the DVC metadata from that version.
3. Download/restore the corresponding data.
```

---

# 29. Return to the Latest Version

If your default branch is `master`:

```bash
git checkout master
```

If your repository uses `main`:

```bash
git checkout main
```

Then:

```bash
dvc pull
```

This restores the data corresponding to the latest checked-out Git state.

---

# 30. Complete End-to-End Command Workflow

Here is the complete workflow in one place.

## Git setup

```bash
git clone <repository-url>
cd <repository-name>
```

## Generate initial data

```bash
python code.py
```

## Initial Git version

```bash
git add .
git commit -m "add initial project and data"
git push
```

## Install DVC

```bash
pip install dvc
dvc --version
```

## Initialize DVC

```bash
dvc init
```

## Create local remote

```bash
mkdir S3
```

## Configure remote

```bash
dvc remote add -d myremote S3
```

## Move data tracking from Git to DVC

```bash
dvc add data/
git rm -r --cached data
git commit -m "stop tracking data"
```

## Add data to DVC

```bash
dvc add data/
```

## Track DVC metadata with Git

```bash
git add .
git commit -m "start tracking data with DVC"
git push
```

## Store data in DVC remote

```bash
dvc commit
dvc push
```

---

# 31. Data V2

Modify:

```text
code.py
```

Generate:

```bash
python code.py
```

Check:

```bash
dvc status
```

Save:

```bash
dvc commit
dvc push
```

Save Git metadata:

```bash
git add .
git commit -m "update dataset to v2"
git push
```

Verify:

```bash
git status
dvc status
```

---

# 32. Data V3

Modify:

```text
code.py
```

Generate:

```bash
python code.py
```

Check:

```bash
dvc status
```

Save:

```bash
dvc commit
dvc push
```

Save Git metadata:

```bash
git add .
git commit -m "update dataset to v3"
git push
```

Verify:

```bash
git status
dvc status
```

---

# 33. Restore V1

Find the Git commit:

```bash
git log --oneline
```

Checkout:

```bash
git checkout 06997c7
```

Restore DVC data:

```bash
dvc pull
```

---

# 34. Git + DVC Cheat Sheet

| Task | Command |
|---|---|
| Initialize DVC | `dvc init` |
| Add data to DVC | `dvc add data/` |
| Configure remote | `dvc remote add -d myremote S3` |
| Check data state | `dvc status` |
| Record data state | `dvc commit` |
| Push data | `dvc push` |
| Pull data | `dvc pull` |
| Check Git state | `git status` |
| View Git history | `git log --oneline` |
| Checkout version | `git checkout <hash>` |
| Commit Git changes | `git commit` |
| Push Git changes | `git push` |

---

# 35. Git vs DVC Cheat Sheet

| Operation | Git | DVC |
|---|---:|---:|
| Source code | ✓ | |
| Git history | ✓ | |
| Large data versioning | | ✓ |
| Data metadata | ✓ via `data.dvc` | ✓ |
| Actual dataset storage | | ✓ |
| Commit code | `git commit` | |
| Push Git history | `git push` | |
| Push data | | `dvc push` |
| Pull data | | `dvc pull` |
| Check code changes | `git status` | |
| Check data changes | | `dvc status` |
| Restore project version | `git checkout` | |
| Restore associated data | | `dvc pull` |

---

# 36. The Most Important Mental Model

Remember:

```text
Git
 │
 ├── Code
 ├── Project history
 └── data.dvc
          │
          ▼
         DVC
          │
          ▼
      Data Remote
          │
     ┌────┼────┐
     ▼    ▼    ▼
    V1   V2   V3
```

And remember the commands:

```text
Git:

git add
   ↓
git commit
   ↓
git push
```

DVC:

```text
dvc add
   ↓
dvc commit
   ↓
dvc push
```

Restore:

```text
git checkout <old-commit>
          ↓
       dvc pull
```

---
# 37. Common Mistakes

## Mistake 1 — Only running `git push`

```bash
git push
```

does not push DVC-tracked data to the DVC remote.

You also need:

```bash
dvc push
```

---

## Mistake 2 — Only running `dvc push`

The data may exist in the DVC remote, but Git still needs the corresponding DVC metadata committed and pushed.

Use:

```bash
git add .
git commit
git push
```

---

## Mistake 3 — Forgetting `dvc status`

After changing data, check:

```bash
dvc status
```

before assuming everything is synchronized.

---

## Mistake 4 — Thinking `git rm --cached` deletes your local dataset

The command:

```bash
git rm -r --cached data
```

removes the data from Git's index while leaving the working copy available for DVC to manage.

---

## Mistake 5 — Checking out an old Git commit but forgetting `dvc pull`

After:

```bash
git checkout <old-commit>
```

you may need:

```bash
dvc pull
```

to restore the corresponding data.

---

# 38. Practical Project Challenge

Build the following project from scratch:

```text
dvc-learning-project/
│
├── code.py
│
├── data/
│   └── data.csv
│
├── data.dvc
│
├── .dvc/
│
├── .dvcignore
│
├── .gitignore
│
└── S3/
```

Create:

```text
Data V1
Data V2
Data V3
```

Then prove that you can:

```text
V1 → V2 → V3
```

and:

```text
V3 → V1
```

using:

```bash
git checkout <V1-commit>
dvc pull
```

Then return to the latest version.

---

# 39. Final Workflow Diagram

```text
                 CREATE PROJECT
                       │
                       ▼
                Create code.py
                       │
                       ▼
                Generate data/
                       │
                       ▼
                 Git add/commit
                       │
                       ▼
                   Git push
                       │
                       ▼
                  dvc init
                       │
                       ▼
              Configure DVC remote
                       │
                       ▼
                  dvc add data/
                       │
                       ▼
             Git stops tracking data
                       │
                       ▼
                data.dvc created
                       │
                       ▼
              Git add/commit/push
                       │
                       ▼
                 dvc commit
                       │
                       ▼
                  dvc push
                       │
                       ▼
                    DATA V1
                       │
                       ▼
                 Modify code.py
                       │
                       ▼
                    DATA V2
                       │
                       ▼
            dvc status / commit / push
                       │
                       ▼
             git add / commit / push
                       │
                       ▼
                    DATA V3
                       │
                       ▼
                  Repeat process
                       │
                       ▼
                 git log --oneline
                       │
                       ▼
              git checkout <V1>
                       │
                       ▼
                   dvc pull
                       │
                       ▼
                 RESTORE DATA V1
```

---

# 40. Final Takeaway

DVC solves an important problem in ML engineering:

> **How do I version large or changing datasets while keeping them connected to my Git-based source-code history?**

The essential pattern is:

```text
                 Git
                  │
           Source + metadata
                  │
                  ▼
              data.dvc
                  │
                  ▼
                 DVC
                  │
                  ▼
             Data Remote
                  │
          ┌───────┼───────┐
          ▼       ▼       ▼
         V1      V2      V3
```

The most important commands to remember are:

```bash
dvc init
dvc add data/
dvc status
dvc commit
dvc push
dvc pull
```

and:

```bash
git add .
git commit
git push
git log --oneline
git checkout <commit>
```

The key workflow is:

```text
CHANGE DATA
    ↓
dvc status
    ↓
dvc commit
    ↓
dvc push
    ↓
git add
    ↓
git commit
    ↓
git push
```

And the key restoration workflow is:

```text
git checkout <historical-commit>
            ↓
        dvc pull
            ↓
      historical data
```

Once this workflow is understood, the next useful DVC topics are remote storage such as AWS S3, DVC pipelines, reproducible data-processing stages, experiments, metrics, and integration with MLflow.
