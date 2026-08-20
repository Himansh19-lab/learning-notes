# Cross-Validation — Learn It From First Principles

## 1. Topic Overview

### What is Cross-Validation?

**Cross-validation (CV)** is a model evaluation technique that repeatedly splits available data into training and validation portions so we can estimate how well a machine-learning model is likely to perform on unseen data.

### Simple definition

Instead of trusting a single train/validation split, we train and evaluate the model on **multiple different splits**.

### Technical definition

Cross-validation estimates a model's generalization performance by partitioning the dataset into multiple folds and repeatedly using different folds for validation while training on the remaining folds.

### Why does it matter?

A model can look excellent on one validation split simply because that particular split is unusually easy or difficult.

Cross-validation gives us a more robust estimate.

### Where it fits

```text
Data
 ↓
Train/Test Split
 ↓
Cross-Validation on Training Data
 ↓
Model Selection / Hyperparameter Tuning
 ↓
Final Training
 ↓
Test Set Evaluation
```

---

# 2. The Problem

Suppose we have 100 observations.

We perform:

```text
80 → Training
20 → Validation
```

We train the model and obtain:

```text
Validation Accuracy = 92%
```

Can we conclude that the model will achieve 92% on new data?

Not necessarily.

What if those 20 validation examples happened to be particularly easy?

Or particularly difficult?

A different split might give:

```text
Split 1 → 92%
Split 2 → 86%
Split 3 → 90%
Split 4 → 88%
Split 5 → 91%
```

This tells us more than one number from one split.

---

# 3. Why Do We Need Cross-Validation?

The central problem is:

> **How can we estimate generalization performance without depending too heavily on one arbitrary train/validation split?**

Cross-validation helps by evaluating the model across multiple partitions.

For example, with 5-fold cross-validation:

```text
Fold 1 → Validation
Fold 2 → Training
Fold 3 → Training
Fold 4 → Training
Fold 5 → Training
```

Then:

```text
Fold 2 → Validation
Fold 1 → Training
Fold 3 → Training
Fold 4 → Training
Fold 5 → Training
```

And so on.

Every observation gets an opportunity to be part of the validation set.

---

# 4. Intuition

Imagine five students taking turns being the examiner.

Instead of asking only one examiner:

> "How good is this model?"

we ask five examiners and combine their assessments.

```text
        Dataset
           │
    ┌──────┼──────┐
    ↓      ↓      ↓
  Fold1  Fold2   Fold3 ...
    │      │
    └──────┼──────┘
           ↓
     Multiple Scores
           ↓
      Mean Score
```

The goal is not to create five different models for production.

The goal is to get a better estimate of how the modeling approach behaves on unseen data.

---

# 5. K-Fold Cross-Validation

The most common form is **K-fold cross-validation**.

Suppose:

```text
K = 5
```

We divide the data into five approximately equal folds:

```text
Fold 1
Fold 2
Fold 3
Fold 4
Fold 5
```

Then perform five training/validation cycles.

### Round 1

```text
Validation → Fold 1
Training   → Fold 2 + Fold 3 + Fold 4 + Fold 5
```

### Round 2

```text
Validation → Fold 2
Training   → Fold 1 + Fold 3 + Fold 4 + Fold 5
```

### Round 3

```text
Validation → Fold 3
Training   → Fold 1 + Fold 2 + Fold 4 + Fold 5
```

### Round 4

```text
Validation → Fold 4
Training   → Fold 1 + Fold 2 + Fold 3 + Fold 5
```

### Round 5

```text
Validation → Fold 5
Training   → Fold 1 + Fold 2 + Fold 3 + Fold 4
```

Finally, calculate the average:

```text
CV Score = (Score1 + Score2 + Score3 + Score4 + Score5) / 5
```

---

# 6. Numerical Example

Suppose the five validation accuracies are:

```text
Fold 1 = 0.90
Fold 2 = 0.87
Fold 3 = 0.92
Fold 4 = 0.89
Fold 5 = 0.91
```

The mean is:

```text
(0.90 + 0.87 + 0.92 + 0.89 + 0.91) / 5
= 0.898
```

So:

```text
Mean CV Accuracy ≈ 89.8%
```

We can also examine variation between folds.

If the scores are:

```text
0.90
0.87
0.92
0.89
0.91
```

the model is reasonably consistent.

But if they are:

```text
0.99
0.71
0.95
0.68
0.93
```

the average alone hides a major issue: performance is unstable across splits.

---

# 7. Standard K-Fold Cross-Validation

Scikit-learn provides `KFold`.

```python
from sklearn.model_selection import KFold

kf = KFold(
    n_splits=5,
    shuffle=True,
    random_state=42
)
```

Important parameters:

### `n_splits`

Number of folds.

```python
n_splits=5
```

means 5-fold cross-validation.

### `shuffle`

Whether to shuffle observations before splitting.

### `random_state`

Makes the randomized splitting reproducible when shuffling is enabled.

---

# 8. Cross-Validation with Scikit-learn

Example:

```python
from sklearn.datasets import load_iris
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import cross_val_score

X, y = load_iris(return_X_y=True)

model = RandomForestClassifier(
    n_estimators=100,
    random_state=42
)

scores = cross_val_score(
    model,
    X,
    y,
    cv=5,
    scoring="accuracy"
)

print("Scores:", scores)
print("Mean:", scores.mean())
```

Possible output:

```text
Scores: [0.97 0.93 0.97 0.93 0.97]
Mean: 0.954
```

The exact scores depend on the dataset, model, and configuration.

---

# 9. Why Use a Separate Test Set?

A common beginner question is:

> "If cross-validation evaluates the model, why do I need a test set?"

Because the validation data used during model selection is no longer completely independent of the modeling process.

Suppose we try:

```text
Model A → CV score = 0.88
Model B → CV score = 0.91
Model C → CV score = 0.93
```

We choose Model C based partly on those validation results.

Therefore, we want another untouched dataset for final evaluation.

A common workflow is:

```text
Complete Dataset
      │
      ├───────────────┐
      ↓               ↓
 Training Data      Test Data
      │
      ↓
 Cross-Validation
      │
      ↓
 Model Selection
      │
      ↓
 Final Model
      │
      ↓
 Evaluate ONCE
 on Test Data
```

The test set should remain untouched until the final evaluation.

---

# 10. Cross-Validation for Hyperparameter Tuning

Cross-validation is particularly useful when selecting hyperparameters.

Suppose we want to choose:

```text
max_depth
```

Try:

```text
max_depth = 3
max_depth = 5
max_depth = 10
max_depth = None
```

Use CV:

```text
max_depth=3  → CV score 0.86
max_depth=5  → CV score 0.89
max_depth=10 → CV score 0.91
max_depth=None → CV score 0.88
```

We might select:

```text
max_depth = 10
```

because it performed best under the chosen cross-validation procedure.

---

# 11. Grid Search

Scikit-learn provides `GridSearchCV`.

```python
from sklearn.model_selection import GridSearchCV
from sklearn.ensemble import RandomForestClassifier

model = RandomForestClassifier(
    random_state=42
)

param_grid = {
    "n_estimators": [100, 200],
    "max_depth": [5, 10, None]
}

grid_search = GridSearchCV(
    estimator=model,
    param_grid=param_grid,
    cv=5,
    scoring="accuracy"
)

grid_search.fit(X, y)

print(grid_search.best_params_)
print(grid_search.best_score_)
```

Conceptually:

```text
Parameter Combination
          ↓
     Cross-Validation
          ↓
      CV Score
          ↓
Compare all combinations
          ↓
Best Parameters
```

---

# 12. Randomized Search

Grid search can become expensive.

Suppose we have:

```text
10 possible values for A
10 possible values for B
10 possible values for C
10 possible values for D
```

That creates:

```text
10 × 10 × 10 × 10 = 10,000
```

combinations.

With 5-fold CV:

```text
10,000 × 5 = 50,000
```

training runs.

Randomized search evaluates a selected number of randomly sampled configurations.

```python
from sklearn.model_selection import RandomizedSearchCV

search = RandomizedSearchCV(
    estimator=model,
    param_distributions=param_grid,
    n_iter=10,
    cv=5,
    scoring="accuracy",
    random_state=42
)
```

---

# 13. Stratified K-Fold Cross-Validation

For classification, class imbalance can make ordinary K-fold splitting problematic.

Suppose:

```text
Class 0 → 95%
Class 1 → 5%
```

We want each fold to have approximately similar class proportions.

That is what **StratifiedKFold** helps with.

```python
from sklearn.model_selection import StratifiedKFold

skf = StratifiedKFold(
    n_splits=5,
    shuffle=True,
    random_state=42
)
```

Conceptually:

```text
Original Data
Class 0 → 95%
Class 1 → 5%

Fold 1 → approximately 95/5
Fold 2 → approximately 95/5
Fold 3 → approximately 95/5
Fold 4 → approximately 95/5
Fold 5 → approximately 95/5
```

For classification problems, stratification is often a sensible default when class proportions matter.

---

# 14. Regression Cross-Validation

For regression, we generally cannot use `StratifiedKFold` directly because the target is continuous.

Standard K-fold is commonly used:

```python
from sklearn.model_selection import KFold

kf = KFold(
    n_splits=5,
    shuffle=True,
    random_state=42
)
```

The important consideration is whether the splitting strategy matches the structure of the data.

---

# 15. Leave-One-Out Cross-Validation

Another approach is **Leave-One-Out Cross-Validation (LOOCV)**.

Suppose there are 100 observations.

LOOCV creates:

```text
Round 1 → train on 99, validate on 1
Round 2 → train on 99, validate on 1
...
Round 100 → train on 99, validate on 1
```

So:

```text
Number of training runs = Number of observations
```

### Advantage

Almost all data is used for training in each iteration.

### Disadvantage

It can be computationally expensive.

For large datasets:

```text
100,000 observations
→ potentially 100,000 training runs
```

That is usually impractical.

---

# 16. Repeated K-Fold

Ordinary K-fold depends on the particular partition.

Repeated K-fold runs K-fold multiple times using different random partitions.

Conceptually:

```text
5-fold CV
    ↓
Repeat
    ↓
Another 5-fold CV
    ↓
Repeat
    ↓
Another 5-fold CV
```

This can provide a more stable estimate when the dataset is relatively small.

---

# 17. Time-Series Cross-Validation

Standard random cross-validation can be wrong for time-series data.

Suppose:

```text
2021 → 2022 → 2023 → 2024 → 2025
```

We cannot randomly mix future observations into training if our real-world task is predicting the future.

Instead, use a time-aware strategy.

Example:

```text
Train: 2021
Validate: 2022

Train: 2021 + 2022
Validate: 2023

Train: 2021 + 2022 + 2023
Validate: 2024

Train: 2021 + 2022 + 2023 + 2024
Validate: 2025
```

In Scikit-learn:

```python
from sklearn.model_selection import TimeSeriesSplit

tscv = TimeSeriesSplit(n_splits=5)
```

### Key rule

> **Never allow future information to leak into the past.**

---

# 18. Group K-Fold

Sometimes multiple observations belong to the same entity.

Example:

```text
Patient A → 10 records
Patient B → 8 records
Patient C → 12 records
```

If records from the same patient appear in both training and validation sets, the model may effectively see information about that patient during training.

That can produce overly optimistic results.

Use `GroupKFold` when groups must stay together.

```python
from sklearn.model_selection import GroupKFold

gkf = GroupKFold(n_splits=5)
```

Conceptually:

```text
Patient A → Training
Patient B → Training
Patient C → Validation
```

rather than splitting individual rows independently.

---

# 19. The Most Important Issue: Data Leakage

Cross-validation does **not** automatically prevent data leakage.

Suppose we perform scaling before cross-validation:

```python
scaler.fit_transform(X)
cross_val_score(model, X_scaled, y, cv=5)
```

The scaler has seen the entire dataset, including observations that later become validation data.

That can leak information across folds.

A safer approach is to put preprocessing inside a Pipeline:

```python
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import cross_val_score

pipeline = Pipeline([
    ("scaler", StandardScaler()),
    ("model", LogisticRegression())
])

scores = cross_val_score(
    pipeline,
    X,
    y,
    cv=5,
    scoring="accuracy"
)
```

Now, for each fold, preprocessing is fitted only on that fold's training data.

This principle is extremely important:

> **Anything that learns from data should generally be fitted inside each training fold, not on the complete dataset before cross-validation.**

---

# 20. Feature Selection and Cross-Validation

The same issue applies to feature selection.

Incorrect pattern:

```text
Complete Dataset
      ↓
Feature Selection
      ↓
Cross-Validation
```

The feature-selection process has already seen the validation observations.

Better:

```text
Fold Training Data
      ↓
Feature Selection
      ↓
Model Training
      ↓
Fold Validation Data
```

Use a Pipeline when possible.

---

# 21. Standard Cross-Validation Workflow

A robust workflow is:

```text
Raw Dataset
     ↓
Separate Final Test Set
     ↓
Training Dataset
     ↓
Choose CV Strategy
     ↓
Pipeline
 ┌─────────────────┐
 │ Preprocessing    │
 │ Feature Selection│
 │ Model            │
 └─────────────────┘
     ↓
Cross-Validation
     ↓
Hyperparameter Tuning
     ↓
Select Model
     ↓
Fit Final Model
     ↓
Evaluate on Untouched Test Set
```

---

# 22. Common Mistakes

## Mistake 1: Using the test set repeatedly

Bad:

```text
Train
 ↓
Test
 ↓
Change model
 ↓
Test again
 ↓
Change model
 ↓
Test again
```

Eventually, the test set becomes part of model selection.

### Better

```text
Train
 ↓
Cross-validation
 ↓
Select model
 ↓
Final test evaluation
```

---

## Mistake 2: Preprocessing before CV

Bad:

```text
Scale entire dataset
       ↓
Cross-validation
```

Better:

```text
Pipeline
  ↓
Scaling inside each fold
  ↓
Model
```

---

## Mistake 3: Ignoring class imbalance

For classification problems with imbalanced classes, consider stratification.

---

## Mistake 4: Random CV for time series

Random splitting can allow future information to influence training.

Use a time-aware strategy.

---

## Mistake 5: Ignoring groups

If multiple rows belong to the same user, patient, customer, device, or other entity, ordinary K-fold can create leakage.

Use group-aware splitting when appropriate.

---

## Mistake 6: Choosing K blindly

There is no universally correct value of K.

Common choices include:

```text
K = 5
K = 10
```

The appropriate choice depends on:

- Dataset size
- Computational budget
- Variance/bias considerations
- Data structure

---

# 23. Bias and Variance Intuition

The choice of K affects the relationship between the amount of training data per fold and the variability of the estimate.

For example:

### 5-fold

Approximately:

```text
80% training
20% validation
```

per iteration.

### 10-fold

Approximately:

```text
90% training
10% validation
```

per iteration.

### LOOCV

Approximately:

```text
99%+ training
~1 observation validation
```

per iteration.

The choice is a practical trade-off between computational cost and the characteristics of the performance estimate.

---

# 24. Cross-Validation Does Not Mean "Train Once"

With 5-fold CV:

```text
5 folds
×
1 training process per fold
=
5 model-training executions
```

If you perform:

```text
20 hyperparameter combinations
×
5 folds
```

you potentially perform:

```text
100 training executions
```

This is why hyperparameter tuning can become computationally expensive.

---

# 25. Nested Cross-Validation

When model selection and performance estimation need to be carefully separated, **nested cross-validation** can be used.

Conceptually:

```text
Outer CV
│
├── Training portion
│      ↓
│   Inner CV
│      ↓
│   Hyperparameter tuning
│
└── Outer validation
       ↓
  Unbiased performance estimate
```

### Inner loop

Used for:

- Hyperparameter tuning
- Model selection

### Outer loop

Used for:

- Estimating generalization performance

Nested CV is particularly useful when rigorous evaluation is important and the dataset is limited.

---

# 26. Choosing the Correct CV Strategy

| Data Situation | Recommended Strategy |
|---|---|
| General classification | Stratified K-Fold |
| General regression | K-Fold |
| Time-dependent data | TimeSeriesSplit |
| Multiple observations per entity | GroupKFold |
| Very small dataset | Consider K-Fold / repeated CV / LOOCV depending on cost |
| Hyperparameter tuning | K-Fold or appropriate specialized splitter |
| Rigorous model-selection evaluation | Nested CV |

The correct choice depends on the data-generating process.

---

# 27. Complete Example with Pipeline and GridSearchCV

```python
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split, GridSearchCV, StratifiedKFold
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression

# Load data
X, y = load_iris(return_X_y=True)

# Hold out final test set
X_train, X_test, y_train, y_test = train_test_split(
    X,
    y,
    test_size=0.2,
    stratify=y,
    random_state=42
)

# Pipeline
pipeline = Pipeline([
    ("scaler", StandardScaler()),
    ("model", LogisticRegression(max_iter=1000))
])

# Hyperparameter search space
param_grid = {
    "model__C": [0.01, 0.1, 1, 10, 100]
}

# Cross-validation strategy
cv = StratifiedKFold(
    n_splits=5,
    shuffle=True,
    random_state=42
)

# Search
search = GridSearchCV(
    estimator=pipeline,
    param_grid=param_grid,
    cv=cv,
    scoring="accuracy"
)

search.fit(X_train, y_train)

print("Best parameters:", search.best_params_)
print("Best CV score:", search.best_score_)

# Final test evaluation
test_score = search.score(X_test, y_test)

print("Final test score:", test_score)
```

Notice the important structure:

```text
20% → untouched test set

80% → cross-validation
         ↓
     preprocessing
         ↓
       model
         ↓
   hyperparameter tuning
```

---

# 28. When Should You Use Cross-Validation?

### Use it when:

- You need a reliable estimate for model selection.
- The dataset is not extremely large.
- You are tuning hyperparameters.
- You want to compare models fairly.
- A single validation split may be unstable.

### Be careful when:

- Training is extremely expensive.
- Data has temporal structure.
- Observations are grouped.
- Data is highly dependent.
- Leakage is possible.

Cross-validation should reflect the way the model will actually encounter unseen data.

---

# 29. Production Perspective

Cross-validation is primarily a **model-development and evaluation technique**.

It is not normally something you run on every production prediction.

A production workflow might look like:

```text
Development
    ↓
Cross-Validation
    ↓
Model Selection
    ↓
Final Training
    ↓
Model Registry
    ↓
Deployment
    ↓
Production Monitoring
```

Once deployed, monitor things such as:

- Prediction quality
- Data drift
- Model drift
- Latency
- Errors
- Business metrics

Then retrain and reevaluate when appropriate.

---

# 30. Interview Questions

## Beginner

1. What is cross-validation?
2. Why do we use cross-validation?
3. What is K-fold cross-validation?
4. What is a fold?
5. What is a CV score?
6. Why do we average CV scores?
7. What is StratifiedKFold?
8. What is the purpose of `shuffle=True`?
9. What is the difference between validation and test data?
10. Why should the test set remain untouched?

## Intermediate

11. Why use cross-validation for hyperparameter tuning?
12. What is the difference between KFold and StratifiedKFold?
13. What is GroupKFold?
14. Why is random K-fold problematic for time series?
15. What is TimeSeriesSplit?
16. What is LOOCV?
17. What is RepeatedKFold?
18. What is GridSearchCV?
19. Why should preprocessing be inside a Pipeline?
20. What is data leakage during cross-validation?

## Advanced

21. Explain nested cross-validation.
22. How would you choose the number of folds?
23. What happens when the dataset is highly imbalanced?
24. How would you perform CV for grouped medical data?
25. How would you perform CV for financial time-series data?
26. How can cross-validation become computationally expensive?
27. How would you distribute CV training jobs?
28. What is the relationship between CV and model selection bias?
29. Why might a high CV score still fail in production?
30. Design a CV strategy for a recommendation system where users generate many observations.

---

# 31. Strong Interview Answer

### Question

> What is cross-validation?

### Short answer

> "Cross-validation is a model evaluation technique where the training data is divided into multiple folds. The model is trained on some folds and validated on the remaining fold, repeating the process so that every fold is used for validation. The resulting scores are aggregated to estimate generalization performance and help with model selection or hyperparameter tuning."

### Stronger answer

> "I would first choose the cross-validation strategy based on the data structure. For standard classification I might use StratifiedKFold, while for time series I would use a time-aware split and for grouped observations I would use GroupKFold. I would also place preprocessing and feature selection inside a Pipeline to prevent leakage. I would use CV on the training set for model selection and reserve a completely untouched test set for the final evaluation."

---

# 32. Flashcards

**Q:** What is cross-validation?

**A:** A technique for repeatedly training and validating a model on different partitions of data to estimate generalization performance.

**Q:** What is K-fold CV?

**A:** A method that divides data into K folds and uses each fold as validation once.

**Q:** What is a parameter?

**A:** A value selected or configured for the model or training process.

**Q:** What is a metric?

**A:** A measured quantity used to evaluate model performance.

**Q:** Why use StratifiedKFold?

**A:** To approximately preserve class proportions across folds in classification problems.

**Q:** Why use TimeSeriesSplit?

**A:** To respect temporal ordering and avoid training on future observations when evaluating past-to-future prediction.

**Q:** Why use GroupKFold?

**A:** To keep observations belonging to the same group from being split across training and validation folds.

**Q:** What is data leakage?

**A:** Unintended use of information that would not legitimately be available when making predictions.

**Q:** Why put scaling inside a Pipeline?

**A:** So the scaler is fitted only on the training portion of each fold.

**Q:** Why keep a test set?

**A:** To obtain a final evaluation on data that was not used for model selection.

**Q:** What is LOOCV?

**A:** Cross-validation where one observation is left out for validation at a time.

**Q:** What is nested CV?

**A:** An outer CV loop for performance estimation containing an inner CV loop for model selection or hyperparameter tuning.

---

# 33. Knowledge Check

Try these before reading an answer.

### Beginner

1. Explain cross-validation in your own words.
2. Why might a single train/validation split be unreliable?
3. What happens in 5-fold CV?
4. What is the purpose of a test set?
5. What does stratification accomplish?

### Intermediate

6. Why should scaling happen inside a Pipeline?
7. When would you use GroupKFold?
8. Why is random K-fold dangerous for time-series data?
9. Why does GridSearchCV require multiple model fits?
10. What is the difference between validation data and the final test data?

### Advanced

11. Explain nested cross-validation.
12. Design a CV strategy for customer-level transaction data.
13. Design a CV strategy for stock-price prediction.
14. Explain how leakage can make CV scores artificially high.
15. Explain why CV score alone does not guarantee production performance.

---

# 34. Answer Key

### 1. Explain cross-validation.

Cross-validation repeatedly divides the available training data into training and validation portions to estimate how the modeling approach generalizes to unseen data.

### 2. Why can one split be unreliable?

A single split can accidentally produce an unusually easy or difficult validation sample.

### 3. What happens in 5-fold CV?

The data is divided into five folds. Five training/validation cycles are performed, with each fold serving as validation once.

### 4. Why keep a test set?

The test set provides a final evaluation on data that was not used for model selection.

### 5. What does stratification accomplish?

It attempts to preserve class proportions across classification folds.

### 6. Why put scaling inside a Pipeline?

To prevent the scaler from learning from validation data.

### 7. When use GroupKFold?

When observations are associated with groups that must not be split between training and validation.

### 8. Why avoid random K-fold for time series?

Because random splitting can allow future information to influence training.

### 9. Why does GridSearchCV require many model fits?

Each hyperparameter configuration must be evaluated across the CV folds.

### 10. Validation vs test data?

Validation data influences model selection; the final test set should remain untouched until final evaluation.

---

# 35. Cheat Sheet

## Core concepts

```text
Fold
→ One partition of the dataset.

K-Fold
→ Split into K folds and rotate the validation fold.

CV Score
→ Performance measured across validation folds.

StratifiedKFold
→ Preserves class proportions.

GroupKFold
→ Keeps groups together.

TimeSeriesSplit
→ Respects chronological order.

LOOCV
→ One observation held out per iteration.

Nested CV
→ Inner CV for selection, outer CV for evaluation.
```

## Most important rule

```text
TEST SET
↓
Never use it repeatedly for model selection.

TRAINING SET
↓
Use cross-validation for model selection and tuning.
```

## Leakage prevention

```text
Preprocessing
Feature Selection
Encoding
Imputation
Feature Engineering

        ↓

Put learned transformations inside the CV Pipeline.
```

---

# 36. Final Takeaways

## If You Remember Only 5 Things

1. **Cross-validation evaluates a modeling approach across multiple data splits.**
2. **K-fold CV rotates which fold is used for validation.**
3. **Choose the CV strategy based on your data structure.**
4. **Keep preprocessing and learned transformations inside the CV pipeline to prevent leakage.**
5. **Use CV for model selection and keep a final test set untouched for final evaluation.**

---

## 1-Minute Interview Explanation

> "Cross-validation is a technique for estimating model generalization by repeatedly splitting the training data into different training and validation folds. In K-fold cross-validation, the data is divided into K folds and each fold is used as validation once. For classification I may use StratifiedKFold, while time-series and grouped data require specialized splitting strategies. I would use cross-validation for model selection and hyperparameter tuning, while keeping a separate test set untouched for final evaluation. I would also put preprocessing inside a Pipeline to avoid data leakage."

---

## 5-Minute Interview Explanation

Cross-validation is a model evaluation and selection technique designed to reduce dependence on a single arbitrary validation split.

In K-fold cross-validation, the training data is divided into K folds. The model is trained on K-1 folds and validated on the remaining fold. This is repeated K times so every fold serves as validation once. The resulting scores can be summarized using the mean and, when useful, measures of variability.

The choice of splitter should reflect the data. StratifiedKFold is useful for classification when preserving class proportions matters. GroupKFold is useful when observations belong to groups such as customers or patients. TimeSeriesSplit is appropriate when chronological ordering must be preserved.

Cross-validation is commonly used for model comparison and hyperparameter tuning. However, it does not automatically prevent leakage. Preprocessing, feature selection, imputation, and other transformations that learn from data should generally be performed inside a Pipeline so they are fitted independently within each training fold.

A final test set should remain untouched during model selection. After selecting the modeling approach, the final model can be trained using the appropriate training data and evaluated once on the held-out test set.

---

# 37. Learning Roadmap

```text
Train/Test Split
       ↓
Cross-Validation
       ↓
K-Fold
       ↓
Stratified K-Fold
       ↓
Pipelines
       ↓
GridSearchCV
       ↓
RandomizedSearchCV
       ↓
Time-Series CV
       ↓
Group CV
       ↓
Nested CV
       ↓
Model Evaluation
       ↓
MLflow Experiment Tracking
       ↓
Production MLOps
```

### Recommended next project

Build a **Customer Churn Prediction** project that includes:

```text
Dataset
   ↓
Train/Test Split
   ↓
Pipeline
   ↓
Stratified K-Fold
   ↓
GridSearchCV
   ↓
Model Selection
   ↓
Final Test Evaluation
   ↓
MLflow Tracking
```

This connects cross-validation directly to practical MLOps.
