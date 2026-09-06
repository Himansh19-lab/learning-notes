# Git — Learn It From First Principles

## 1. Topic Overview

### What is Git?

**Git is a distributed version control system used to track changes to files and coordinate work across software projects.**

Git is especially important in ML, AI, and MLOps because projects contain:

- Python code
- Training scripts
- Configuration
- Tests
- Dockerfiles
- CI/CD workflows
- Infrastructure code
- Documentation

A useful mental model is:

```text
Working Directory
      ↓
   git add
      ↓
Staging Area
      ↓
  git commit
      ↓
Local Repository
      ↓
   git push
      ↓
Remote Repository
```

### Simple definition

Git records the history of changes to a project so you can inspect, compare, restore, and collaborate on different versions.

### Technical definition

Git is a distributed version control system in which repositories contain snapshots of project history, represented through commits and connected by references such as branches and tags.

---

# 2. The Problem Git Solves

Imagine working on a project without version control.

You might create:

```text
project_final.py
project_final_v2.py
project_final_v3.py
project_final_really_final.py
project_final_really_final_v2.py
```

This quickly becomes difficult to manage.

Git replaces this with a structured history:

```text
Commit A
   ↓
Commit B
   ↓
Commit C
   ↓
Commit D
```

You can inspect what changed between commits and return to earlier versions when necessary.

---

# 3. Why Version Control Matters

Git provides:

- Change history
- Collaboration
- Branching
- Merging
- Reverting
- Code review workflows
- Traceability
- Reproducibility

For an ML project, you might want to know:

```text
Which code produced this model?
Which configuration was used?
Which commit was deployed?
Who changed the training pipeline?
When did this behavior change?
```

Git helps answer these questions for source-controlled project files.

---

# 4. Git vs GitHub

This is a very important distinction.

## Git

Git is the **version control system**.

It runs locally and manages repository history.

```text
Your Computer
    ↓
   Git
```

## GitHub

GitHub is a hosted platform for Git repositories and collaboration.

```text
Your Computer
    ↓
   Git
    ↓
GitHub Repository
```

Other platforms include:

- GitLab
- Bitbucket

### Mental shortcut

```text
Git       → Version control technology
GitHub    → Platform built around Git repositories
```

---

# 5. Core Git Concepts

The most important concepts are:

```text
Repository
Commit
Branch
Working Directory
Staging Area
Remote
Merge
Pull
Push
Clone
Fetch
```

Let's understand each one.

---

# 6. Repository

A **repository**, or repo, is a Git-managed project containing files and their version history.

Conceptually:

```text
Repository
│
├── Source Code
├── Tests
├── Configuration
├── Documentation
└── Git History
```

A local repository contains the project's Git metadata and history.

---

# 7. Working Directory

The **working directory** contains the files you are currently editing.

Example:

```text
project/
│
├── train.py
├── model.py
├── tests/
└── README.md
```

You edit:

```text
train.py
```

Git notices that the working tree differs from the latest committed version.

Check:

```bash
git status
```

---

# 8. Staging Area

The staging area is where you prepare changes for the next commit.

Suppose you modify:

```text
train.py
model.py
README.md
```

You only want to commit:

```text
train.py
model.py
```

You can stage specific files:

```bash
git add train.py model.py
```

Now:

```text
Working Directory
      ↓
    git add
      ↓
Staging Area
      ↓
   git commit
```

The staging area gives you control over what goes into a commit.

---

# 9. Commit

A **commit** is a recorded snapshot of staged changes.

Example:

```bash
git commit -m "Add model training pipeline"
```

A commit contains information such as:

- Snapshot of tracked content
- Author information
- Commit message
- Parent commit reference
- Commit identifier

Think of a commit as a named checkpoint in project history.

---

# 10. Git's Basic Workflow

The most important workflow to remember:

```text
Edit files
    ↓
git status
    ↓
git add
    ↓
git commit
    ↓
git push
```

Example:

```bash
git status

git add train.py

git commit -m "Improve model training"

git push
```

---

# 11. `git init`

To create a new Git repository:

```bash
git init
```

This creates Git's repository metadata in the project directory.

Typical workflow:

```bash
mkdir ml-project
cd ml-project

git init
```

Then:

```bash
git status
```

---

# 12. `git clone`

To copy an existing repository:

```bash
git clone <repository-url>
```

Conceptually:

```text
Remote Repository
       ↓
   git clone
       ↓
Local Repository
```

Cloning normally gives you:

- Project files
- Git history
- Remote configuration

---

# 13. `git status`

One of the most useful Git commands:

```bash
git status
```

It tells you about:

- Modified files
- Staged files
- Untracked files
- Current branch
- Other useful repository state

When learning Git, use `git status` frequently.

---

# 14. `git add`

Stage changes:

```bash
git add train.py
```

Stage multiple files:

```bash
git add train.py model.py
```

Stage all relevant changes:

```bash
git add .
```

Be careful with broad staging commands in projects containing generated files or secrets.

---

# 15. `git commit`

Create a commit:

```bash
git commit -m "Add feature engineering"
```

Good commit messages describe the change.

Good:

```text
Add feature validation
Fix prediction endpoint
Update training pipeline
```

Less useful:

```text
changes
update
stuff
final
```

A commit should represent a meaningful unit of work when practical.

---

# 16. `git log`

View commit history:

```bash
git log
```

A compact form:

```bash
git log --oneline
```

Example:

```text
a81f3c2 Add model monitoring
b92d1a4 Fix prediction API
c31a7ef Add training pipeline
```

Each commit has an identifier, commonly displayed as a hash.

---

# 17. Commit Hash

A Git commit has a unique identifier derived from its contents and metadata.

Example:

```text
a81f3c2...
```

This allows you to refer to a specific point in history.

For example:

```bash
git show a81f3c2
```

---

# 18. Branches

A branch is a movable reference to a line of development.

Imagine:

```text
A
↓
B
↓
C
```

You create a feature branch:

```text
        D
       /
A → B → C
```

One line can continue on `main`, while another develops a feature.

Branches allow developers to work independently.

---

# 19. Why Use Branches?

Suppose production currently contains:

```text
main
```

You want to build:

```text
new-model-api
```

Create a branch:

```bash
git switch -c new-model-api
```

Then:

```text
main
  ↓
feature branch
```

You can develop without directly changing the main line of development.

---

# 20. Creating and Switching Branches

Create and switch:

```bash
git switch -c feature/model-api
```

Switch to an existing branch:

```bash
git switch main
```

List branches:

```bash
git branch
```

List local and remote branches:

```bash
git branch -a
```

Older workflows often use:

```bash
git checkout
```

Modern Git generally provides `git switch` and `git restore` as clearer commands for common branch/file operations.

---

# 21. Merging

Suppose:

```text
main
  ↓
A → B → C

feature
      ↓
      D → E
```

You want to bring the feature into main.

```bash
git switch main
git merge feature/model-api
```

Conceptually:

```text
A → B → C
     \     \
      D → E
            ↓
          Merge
```

Depending on the history, Git may perform a fast-forward or create a merge commit.

---

# 22. Merge Conflicts

A conflict can occur when two branches change overlapping parts of a file in incompatible ways.

Example:

```text
main:
learning_rate = 0.01

feature:
learning_rate = 0.001
```

Git cannot always decide which change is correct.

You may see:

```text
<<<<<<< HEAD
learning_rate = 0.01
=======
learning_rate = 0.001
>>>>>>> feature
```

You must manually choose the correct final content.

Then:

```bash
git add <resolved-file>
git commit
```

The exact command sequence can vary depending on the merge operation.

---

# 23. Pull

`git pull` generally combines fetching remote changes with integrating them into the current branch.

Conceptually:

```text
Remote
  ↓
Fetch
  ↓
Integrate
  ↓
Local branch
```

Typical command:

```bash
git pull
```

A pull can result in:

- Fast-forward
- Merge
- Rebase, depending on configuration/options

Because pulling changes can affect your working branch, inspect repository status before and after significant operations.

---

# 24. Fetch

`git fetch` downloads remote updates without automatically integrating them into your current branch.

```bash
git fetch
```

This is useful when you want to inspect remote changes first.

Conceptually:

```text
Remote Repository
       ↓
   git fetch
       ↓
Remote-tracking references
       ↓
Review
       ↓
Merge / Rebase when appropriate
```

---

# 25. Push

`git push` sends local commits to a remote repository.

```bash
git push
```

First-time branch publishing may use:

```bash
git push -u origin feature/model-api
```

Conceptually:

```text
Local Repository
       ↓
    git push
       ↓
Remote Repository
```

---

# 26. Remote

A remote is a named reference to another repository.

Common remote name:

```text
origin
```

View remotes:

```bash
git remote -v
```

Example:

```text
origin  https://example.com/project.git
```

The remote might be hosted on GitHub, GitLab, Bitbucket, or another Git server.

---

# 27. `origin` and `main`

Two common terms:

### `origin`

Usually the default name for the remote repository.

### `main`

A common name for the primary branch.

They are different concepts:

```text
origin → remote name

main   → branch name
```

So:

```text
origin/main
```

means the `main` branch as represented by the `origin` remote.

---

# 28. Pull Requests

A **Pull Request (PR)** is a collaboration mechanism commonly provided by Git hosting platforms.

Typical workflow:

```text
Create Branch
     ↓
Make Changes
     ↓
Commit
     ↓
Push
     ↓
Open Pull Request
     ↓
Code Review
     ↓
CI Tests
     ↓
Approval
     ↓
Merge
```

A PR is not itself a core Git object. It is a platform-level collaboration workflow built around Git branches and commits.

---

# 29. Git and CI/CD

Git is often the trigger for CI/CD pipelines.

```text
Developer
   ↓
git push
   ↓
GitHub / GitLab
   ↓
CI Pipeline
   ↓
Tests
   ↓
Build
   ↓
Deploy
```

For ML:

```text
git push
   ↓
CI
   ↓
Unit Tests
   ↓
Data Validation
   ↓
Model Tests
   ↓
Docker Build
   ↓
Deploy
```

---

# 30. Git in MLOps

Git is essential for versioning source code and configuration.

A mature ML system may combine:

```text
Git
 ↓
Code Version

DVC / Object Storage
 ↓
Data Version

MLflow
 ↓
Experiment + Model Tracking

Container Registry
 ↓
Deployment Artifact

Kubernetes
 ↓
Runtime
```

Git alone should not be treated as the complete solution for large datasets, model artifacts, or experiment tracking.

---

# 31. `.gitignore`

Some files should not be committed.

Example:

```text
.env
__pycache__/
*.pyc
.venv/
.ipynb_checkpoints/
models/*.pkl
data/
```

Create:

```text
.gitignore
```

Example:

```gitignore
.venv/
__pycache__/
*.pyc
.env
.ipynb_checkpoints/
```

The exact contents depend on your project.

---

# 32. Never Commit Secrets

Never put credentials directly into Git:

```python
API_KEY = "secret"
PASSWORD = "secret"
AWS_SECRET_ACCESS_KEY = "secret"
```

Instead use:

```text
Environment Variables
Secret Manager
CI/CD Secret Store
Cloud Secret Service
```

Important:

> Adding a secret to `.gitignore` does not remove a secret that was already committed.

If a secret has been exposed, rotate/revoke it and then clean the repository history when appropriate.

---

# 33. Git Tags

Tags can mark important points in history.

Example:

```bash
git tag v1.0.0
```

Push a tag:

```bash
git push origin v1.0.0
```

Tags are useful for releases:

```text
v1.0.0
v1.1.0
v2.0.0
```

---

# 34. Semantic Versioning

Many software projects use:

```text
MAJOR.MINOR.PATCH
```

Example:

```text
1.4.2
```

Conceptually:

```text
1 → Major
4 → Minor
2 → Patch
```

The exact meaning depends on the project's versioning policy.

---

# 35. Reverting Changes

If a bad commit has already been shared, `git revert` is often the safer way to undo its effect.

```bash
git revert <commit>
```

This creates a new commit that reverses the selected commit's changes.

Conceptually:

```text
A → B → C
         ↓
       Revert C
         ↓
A → B → C → D
```

The original commit remains in history.

---

# 36. Reset

`git reset` can move branch references and modify the staging area and/or working tree depending on the mode.

Common modes:

```bash
git reset --soft
git reset --mixed
git reset --hard
```

Be careful with:

```bash
git reset --hard
```

because it can discard uncommitted changes.

### Mental distinction

```text
git revert
→ Create a new commit that undoes an earlier commit.

git reset
→ Move/reset repository state and references.
```

For shared branches, `revert` is generally safer than rewriting published history.

---

# 37. Rebase

Rebase moves or replays commits onto another base.

Suppose:

```text
A → B → C       main

     \
      D → E     feature
```

Rebase the feature:

```text
A → B → C → D' → E'
```

This creates rewritten commits.

Rebase can produce a cleaner linear history, but it rewrites commit identities.

Important rule:

> Be careful rebasing commits that other people are already using.

---

# 38. Merge vs Rebase

| Merge | Rebase |
|---|---|
| Preserves existing branch history | Rewrites/replays commits |
| Can create merge commit | Often creates a linear history |
| Generally safer for shared history | Requires more care |
| Explicitly records integration | Makes commits appear based on a new parent |

Both are useful. The team's workflow determines which is preferred.

---

# 39. Stash

Sometimes you have uncommitted work but need to switch branches.

You can temporarily stash changes:

```bash
git stash
```

Then switch branches:

```bash
git switch main
```

Later:

```bash
git stash pop
```

Conceptually:

```text
Uncommitted Work
      ↓
    stash
      ↓
Temporary storage
      ↓
Switch branches
      ↓
stash pop
      ↓
Restore work
```

Do not use stash as a substitute for meaningful commits.

---

# 40. Comparing Changes

View unstaged changes:

```bash
git diff
```

View staged changes:

```bash
git diff --staged
```

Compare commits:

```bash
git diff <commit1> <commit2>
```

This is extremely useful for debugging.

---

# 41. Inspecting a Commit

Use:

```bash
git show <commit>
```

You can inspect:

- Commit metadata
- Commit message
- Changed files
- Diff

---

# 42. Finding Who Changed a Line

Use:

```bash
git blame <file>
```

This can show which commit and author last changed each line.

Example:

```bash
git blame train.py
```

This is useful for tracing the history of a line, but it should not be used as a tool for assigning blame to people.

---

# 43. Git Workflow for a Team

A common workflow:

```text
main
 │
 ├── feature/data-validation
 │
 ├── feature/model-api
 │
 └── bugfix/prediction-error
```

Each developer:

```text
Create branch
    ↓
Implement change
    ↓
Commit
    ↓
Push
    ↓
Pull Request
    ↓
CI
    ↓
Review
    ↓
Merge
```

This integrates naturally with CI/CD.

---

# 44. Git Workflow for ML Projects

A practical ML workflow:

```text
Create Feature Branch
        ↓
Update Training Code
        ↓
Run Local Tests
        ↓
Commit
        ↓
Push
        ↓
Pull Request
        ↓
CI
        ↓
Data / Model Tests
        ↓
Review
        ↓
Merge
        ↓
Training / Deployment Pipeline
```

Model artifacts and datasets may be stored using specialized systems rather than Git itself.

---

# 45. What Should Be Stored in Git?

Good candidates:

```text
Python source code
Configuration
Tests
Dockerfiles
CI/CD workflows
Infrastructure code
Documentation
Small metadata files
Schemas
```

Be careful with:

```text
Large datasets
Large model binaries
Secrets
Generated files
Temporary files
```

For large ML artifacts, consider:

```text
DVC
Object Storage
Model Registry
Artifact Registry
```

---

# 46. Git LFS

Git Large File Storage (Git LFS) is designed to handle large files by storing lightweight pointers in Git while keeping large file contents in separate storage.

It can be useful for:

```text
Large model files
Large datasets
Large media files
```

But Git LFS is not automatically the best solution for every ML data or model-management problem.

For production ML systems, consider the overall data/artifact architecture.

---

# 47. Commit Best Practices

Good commits are:

- Small enough to understand
- Focused
- Descriptive
- Testable
- Related to one logical change

Example:

```text
Add input schema validation
```

rather than:

```text
Update everything
```

A useful history:

```text
Add feature validation
Add model training test
Fix prediction API
Add Docker health check
```

---

# 48. Branch Naming

Consistent branch names make collaboration easier.

Examples:

```text
feature/model-api
feature/data-validation
bugfix/prediction-error
hotfix/security-patch
refactor/training-pipeline
```

The exact naming convention should follow your team's standards.

---

# 49. Git Hooks

Git hooks allow scripts to run at certain Git lifecycle events.

Examples:

```text
pre-commit
pre-push
```

They can be used for:

- Formatting
- Linting
- Tests
- Secret checks

Example:

```text
git commit
    ↓
pre-commit hook
    ↓
Lint
    ↓
Format
    ↓
Commit
```

Hooks can improve developer feedback, but important checks should also run in CI because local hooks can be bypassed or may not be installed consistently.

---

# 50. Git Security

Important security practices:

- Never commit secrets.
- Use protected branches.
- Require code review for important branches.
- Use least-privilege access.
- Protect CI/CD credentials.
- Scan dependencies.
- Audit repository access.
- Rotate compromised credentials.
- Avoid executing untrusted repository code without understanding the risks.

---

# 51. Git and Reproducibility

For ML, reproducibility often requires connecting:

```text
Code
 ↓
Git Commit
 ↓
Configuration
 ↓
Data Version
 ↓
Training Run
 ↓
Model Version
```

For example:

```text
Git Commit: abc123
Data Version: v7
MLflow Run: run-42
Model Version: 12
Docker Image: 1.5.0
```

This creates traceability across the ML lifecycle.

---

# 52. End-to-End Git Workflow

A common workflow for a new feature:

```bash
# Get latest main
git switch main
git pull

# Create feature branch
git switch -c feature/model-api

# Make changes
# ...

# Inspect changes
git status
git diff

# Stage
git add .

# Commit
git commit -m "Add model prediction API"

# Push branch
git push -u origin feature/model-api
```

Then:

```text
Open Pull Request
        ↓
CI Tests
        ↓
Code Review
        ↓
Merge
```

---

# 53. Git Commands Cheat Sheet

## Setup

```bash
git init
git clone <url>
```

## Status

```bash
git status
```

## Stage

```bash
git add <file>
git add .
```

## Commit

```bash
git commit -m "message"
```

## History

```bash
git log
git log --oneline
```

## Branches

```bash
git branch
git switch -c feature/name
git switch main
```

## Remote

```bash
git remote -v
git fetch
git pull
git push
```

## Compare

```bash
git diff
git diff --staged
```

## Inspect

```bash
git show <commit>
```

## Undo

```bash
git revert <commit>
git reset
```

## Temporary work

```bash
git stash
git stash pop
```

## Tags

```bash
git tag v1.0.0
git push origin v1.0.0
```

---

# 54. Common Problems and Debugging

## Problem 1: "I committed the wrong file"

First inspect:

```bash
git show --stat
```

If the commit has not been shared and you need to amend it:

```bash
git commit --amend
```

Use history-rewriting commands carefully, especially on shared branches.

---

## Problem 2: Merge conflict

Workflow:

```text
Merge
 ↓
Conflict
 ↓
Open conflicted files
 ↓
Resolve manually
 ↓
git add
 ↓
Complete merge
```

Always verify the resulting code and run tests.

---

## Problem 3: Push rejected

Often the remote branch contains commits you do not have locally.

A common approach is:

```bash
git fetch
```

Then inspect the divergence and decide whether to merge or rebase according to your team's workflow.

Avoid blindly using force push.

---

## Problem 4: Accidentally committed a secret

Do not assume deleting the file is enough.

Immediate priorities:

```text
Revoke / rotate secret
        ↓
Assess exposure
        ↓
Remove from active history
        ↓
Notify affected parties
        ↓
Add prevention controls
```

History rewriting may be necessary depending on the situation.

---

# 55. Common Mistakes

### Mistake 1

Using Git as a backup system.

Git is version control, not a complete backup strategy.

### Mistake 2

Committing secrets.

Never do this.

### Mistake 3

Huge commits.

Large mixed commits are harder to review and debug.

### Mistake 4

Using unclear commit messages.

### Mistake 5

Working directly on protected branches.

Use feature branches and pull requests when appropriate.

### Mistake 6

Force-pushing shared branches without coordination.

### Mistake 7

Tracking huge datasets directly in ordinary Git.

Use appropriate data/artifact tooling.

### Mistake 8

Assuming `.gitignore` removes already tracked files.

It does not.

---

# 56. Git vs GitHub vs GitLab

| Technology | Role |
|---|---|
| Git | Version control system |
| GitHub | Hosted Git collaboration platform |
| GitLab | Hosted Git and DevOps platform |
| Bitbucket | Hosted Git collaboration platform |

The Git concepts themselves are separate from the hosting platform.

---

# 57. Git vs DVC vs MLflow

This distinction is important in MLOps.

| Tool | Primary Purpose |
|---|---|
| Git | Source code and project history |
| DVC | Data/model artifact versioning workflows |
| MLflow | Experiment tracking and model lifecycle |
| Container Registry | Container images |
| Object Storage | Large files/artifacts |

A production ML system may use several of these together.

```text
Git
 ↓
Code

DVC / Object Storage
 ↓
Data

MLflow
 ↓
Experiments + Models

Registry
 ↓
Deployable Artifacts
```

---

# 58. Interview Questions

## Beginner

1. What is Git?
2. What is version control?
3. What is a Git repository?
4. What is a commit?
5. What is a branch?
6. What is the staging area?
7. What does `git status` do?
8. What does `git clone` do?
9. What does `git push` do?
10. What does `git pull` do?

## Intermediate

11. Git vs GitHub?
12. What is the difference between fetch and pull?
13. What is a merge conflict?
14. What is a Pull Request?
15. What is `git revert`?
16. What is `git reset`?
17. What is `git stash`?
18. What is rebasing?
19. Merge vs rebase?
20. Why use `.gitignore`?

## Advanced

21. Explain Git's object model at a high level.
22. What is a commit hash?
23. How does Git store history?
24. What happens during a merge?
25. What happens during a rebase?
26. Why is force-pushing dangerous?
27. How would you design a Git workflow for a large ML team?
28. How would you handle large datasets in an ML project?
29. How does Git integrate with CI/CD?
30. How would you recover from an accidentally committed secret?

---

# 59. Scenario-Based Interview Questions

### Scenario 1

A developer accidentally pushes an API key to a public repository.

What should happen immediately?

### Scenario 2

Two developers modify the same lines of a Python file.

How would you resolve the conflict?

### Scenario 3

A production deployment needs to be traced back to the exact code version.

How would Git help?

### Scenario 4

Your ML repository contains a 10 GB model file and a 100 GB dataset.

Would you commit them directly to ordinary Git?

Why or why not?

### Scenario 5

Your team has 20 developers working on an ML platform.

Design a branch, review, and CI workflow.

---

# 60. Strong Interview Answer

### Question

> What is Git?

### Short answer

> "Git is a distributed version control system used to track changes to files and collaborate on software projects. It stores project history through commits and supports branches, merging, and distributed repositories."

### Strong answer

> "Git is a distributed version control system that tracks project history through commits. Developers typically work in a local repository, create branches for isolated changes, stage and commit changes, and synchronize with a remote repository using fetch, pull, and push. Git integrates naturally with pull-request-based code review and CI/CD systems. In ML projects, Git is primarily used for source code, configuration, tests, and infrastructure code, while large datasets, model artifacts, and experiment tracking are often handled by specialized tools such as DVC, object storage, and MLflow."

---

# 61. Flashcards

**Q:** What is Git?

**A:** A distributed version control system.

**Q:** What is a repository?

**A:** A project managed by Git containing files and version history.

**Q:** What is a commit?

**A:** A recorded snapshot of staged changes.

**Q:** What is a branch?

**A:** A movable reference representing a line of development.

**Q:** What is staging?

**A:** Selecting changes to include in the next commit.

**Q:** What does `git status` do?

**A:** Shows the current repository state and changes.

**Q:** What does `git clone` do?

**A:** Creates a local copy of a remote Git repository.

**Q:** What does `git push` do?

**A:** Sends local commits to a remote repository.

**Q:** What does `git fetch` do?

**A:** Downloads remote updates without automatically integrating them into the current branch.

**Q:** What does `git pull` do?

**A:** Fetches remote changes and then integrates them according to the configured pull behavior.

**Q:** What is a merge conflict?

**A:** A situation where Git cannot automatically reconcile conflicting changes.

**Q:** What is `git revert`?

**A:** Creates a new commit that reverses the effect of an earlier commit.

**Q:** What is `git reset`?

**A:** Moves repository references and/or changes staging and working-tree state depending on the selected mode.

**Q:** What is a Pull Request?

**A:** A platform-level workflow for proposing, reviewing, and integrating changes.

**Q:** Why use `.gitignore`?

**A:** To prevent specified untracked files from being included in normal Git workflows.

---

# 62. Knowledge Check

Try these before looking at the answers.

## Beginner

1. What problem does Git solve?
2. What is a commit?
3. What is the staging area?
4. What is a branch?
5. What is the difference between Git and GitHub?

## Intermediate

6. Explain `git fetch` vs `git pull`.
7. Explain merge vs rebase.
8. What is a merge conflict?
9. Why should secrets not be committed?
10. Why use feature branches?

## Advanced

11. Design a Git workflow for an ML team.
12. How would you handle large datasets?
13. How would you recover from an exposed secret?
14. How does Git support CI/CD?
15. Explain how you would trace a production ML model back to source code.

---

# 63. Answer Key

### 1. What problem does Git solve?

Git tracks project changes and provides structured history and collaboration mechanisms.

### 2. What is a commit?

A recorded snapshot of staged project changes.

### 3. What is staging?

Preparing selected changes for the next commit.

### 4. What is a branch?

A reference to a line of development.

### 5. Git vs GitHub?

Git is the version control system; GitHub is a hosted collaboration platform built around Git.

### 6. Fetch vs pull?

Fetch downloads remote updates without integrating them into the current branch; pull generally fetches and integrates.

### 7. Merge vs rebase?

Merge combines histories while preserving them; rebase replays commits onto another base and rewrites commit identities.

### 8. Merge conflict?

Git cannot automatically determine the correct result of overlapping changes.

### 9. Secrets?

They can be exposed to unauthorized users and may remain in repository history even after deletion.

### 10. Feature branches?

They isolate work and support review before integration.

---

# 64. Cheat Sheet

## Core workflow

```text
Edit
 ↓
git status
 ↓
git add
 ↓
git commit
 ↓
git push
```

## Branch workflow

```text
main
 ↓
feature branch
 ↓
commit
 ↓
push
 ↓
Pull Request
 ↓
CI
 ↓
review
 ↓
merge
```

## Remote workflow

```text
git fetch
    ↓
inspect
    ↓
merge / rebase

git pull
    ↓
fetch + integrate
```

## Undo

```text
git revert
→ New commit that reverses an old commit

git reset
→ Move/reset repository state
```

## ML/MLOps

```text
Git
 ↓
Code

DVC / Object Storage
 ↓
Data

MLflow
 ↓
Experiments + Models

Docker
 ↓
Container

Kubernetes
 ↓
Runtime
```

---

# 65. If You Remember Only 5 Things

1. **Git is a distributed version control system.**
2. **Commits provide snapshots of project history; branches provide isolated lines of development.**
3. **The basic workflow is edit → stage → commit → push.**
4. **Pull Requests, CI/CD, and remote hosting platforms build collaboration workflows around Git.**
5. **In ML, use Git for code/configuration and specialized systems for large data, model artifacts, and experiment tracking.**

---

# 66. 1-Minute Interview Explanation

> "Git is a distributed version control system used to track changes and collaborate on software projects. Developers work in local repositories, make changes, stage them, create commits, and synchronize with remote repositories using fetch, pull, and push. Branches allow isolated development, while pull requests provide review and integration workflows on platforms such as GitHub. Git also forms the foundation of many CI/CD pipelines. In ML projects, I would use Git for source code, configuration, tests, and infrastructure while using specialized tools for large datasets, models, and experiment tracking."

---

# 67. 5-Minute Interview Explanation

Git is a distributed version control system that records the history of a software project through commits.

A typical local workflow involves three important areas:

```text
Working Directory
      ↓
Staging Area
      ↓
Repository
```

Developers modify files in the working directory, use `git add` to stage selected changes, and use `git commit` to create a snapshot in repository history.

Branches allow developers to create separate lines of development. A common team workflow is:

```text
main
 ↓
feature branch
 ↓
development
 ↓
commit
 ↓
push
 ↓
Pull Request
 ↓
CI
 ↓
code review
 ↓
merge
```

Git supports synchronization with remote repositories using commands such as `fetch`, `pull`, and `push`.

Git is different from GitHub. Git is the version control technology, while GitHub is a hosted collaboration platform built around Git.

In ML and MLOps, Git is essential for versioning source code, configuration, tests, Dockerfiles, CI/CD definitions, and infrastructure code. However, Git alone is generally not the complete solution for large datasets, model artifacts, and experiment tracking. Those concerns may be handled with tools such as DVC, object storage, MLflow, and container registries.

A production ML system should connect these versions:

```text
Git Commit
    ↓
Code
    ↓
Data Version
    ↓
Training Run
    ↓
Model Version
    ↓
Container Version
    ↓
Production Deployment
```

This traceability makes debugging, reproducibility, auditing, and rollback much easier.

---

# 68. Learning Roadmap

```text
Git Basics
    ↓
Working Directory
    ↓
Staging
    ↓
Commits
    ↓
Branches
    ↓
Merge
    ↓
Merge Conflicts
    ↓
Remote Repositories
    ↓
Fetch / Pull / Push
    ↓
Pull Requests
    ↓
Rebase
    ↓
Tags / Releases
    ↓
Git Hooks
    ↓
GitHub Actions
    ↓
Docker + CI/CD
    ↓
DVC
    ↓
MLflow
    ↓
MLOps
```

## Recommended Project

Build a small **ML project with a professional Git workflow**:

```text
ML Project
   ↓
Initialize Git
   ↓
Create .gitignore
   ↓
Create main branch
   ↓
Feature branch
   ↓
Model development
   ↓
Unit tests
   ↓
Commit
   ↓
Push
   ↓
Pull Request
   ↓
CI
   ↓
Code Review
   ↓
Merge
   ↓
Docker Build
   ↓
MLflow Tracking
   ↓
Deployment
```

### Suggested repository

```text
ml-git-project/
│
├── src/
│   ├── train.py
│   ├── predict.py
│   └── preprocessing.py
│
├── tests/
│   └── test_model.py
│
├── configs/
│   └── config.yaml
│
├── notebooks/
│
├── Dockerfile
├── requirements.txt
├── .gitignore
├── README.md
│
└── .github/
    └── workflows/
        └── ci.yml
```

The goal is to move from simply knowing Git commands to understanding **version control as the foundation for collaborative software engineering and reproducible MLOps**.
