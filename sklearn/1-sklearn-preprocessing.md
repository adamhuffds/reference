# scikit-learn Reference — Preprocessing

Docs: https://scikit-learn.org/stable/modules/preprocessing.html

Prerequisite: `sklearn-basics-workflow.md`

## Why Preprocessing Matters

Most models are sensitive to feature scale (distance-based models like
k-NN/SVM, gradient-descent-based models like linear/logistic regression and
neural nets) or require numeric-only input (nearly everything) — raw,
uncleaned data rarely feeds directly into a model without these steps first.
Tree-based models (`sklearn-models-overview.md`) are a notable exception —
they're scale-invariant and don't strictly need feature scaling.

## Scaling Numeric Features

```python
from sklearn.preprocessing import StandardScaler, MinMaxScaler, RobustScaler

scaler = StandardScaler()               # mean=0, std=1 — the most common default choice
X_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)    # IMPORTANT: use .transform() on test data, NOT .fit_transform()
```
**Critical rule: fit the scaler only on training data, then use `.transform()`
(not `.fit_transform()`) on the test set.** Fitting on the full dataset
(including test data) leaks information about the test set's distribution
into training — a subtle but serious form of data leakage that inflates
apparent model performance.

```python
MinMaxScaler()      # scales to a fixed range, default [0, 1] — sensitive to outliers
RobustScaler()          # uses median/IQR instead of mean/std — more robust to outliers
```
`StandardScaler` is the reasonable default; switch to `RobustScaler`
specifically when your data has significant outliers you don't want
dominating the scaling.

## Encoding Categorical Features

```python
from sklearn.preprocessing import OneHotEncoder, OrdinalEncoder, LabelEncoder

# One-hot encoding — for NOMINAL categories with no inherent order (e.g. city, color)
encoder = OneHotEncoder(sparse_output=False, handle_unknown="ignore")
X_encoded = encoder.fit_transform(X_train[["category_col"]])
```
- `handle_unknown="ignore"` — prevents an error if the test set contains a
  category value never seen during training; instead encodes it as all
  zeros. Worth setting deliberately rather than letting an unseen category
  crash a production pipeline.

```python
# Ordinal encoding — for ORDINAL categories with a meaningful order (e.g. 'low'/'medium'/'high')
encoder = OrdinalEncoder(categories=[["low", "medium", "high"]])
X_encoded = encoder.fit_transform(X_train[["size_col"]])
```
Using `OneHotEncoder` on an ordinal feature (or `OrdinalEncoder` on a
nominal one) is a common modeling mistake — one-hot encoding an ordinal
feature throws away the meaningful order information; ordinal-encoding a
nominal feature invents a false order the model will try to learn from.

```python
# LabelEncoder — for encoding the TARGET (y), not features
le = LabelEncoder()
y_encoded = le.fit_transform(y_train)          # e.g. ['cat','dog','cat'] -> [0, 1, 0]
```
`LabelEncoder` is specifically intended for the target variable in
classification, not for feature columns — use `OneHotEncoder`/
`OrdinalEncoder` for features instead.

## Handling Missing Values

```python
from sklearn.impute import SimpleImputer

imputer = SimpleImputer(strategy="mean")           # numeric: fill with column mean
imputer = SimpleImputer(strategy="median")            # numeric: fill with column median (more robust to outliers)
imputer = SimpleImputer(strategy="most_frequent")         # works for numeric OR categorical
imputer = SimpleImputer(strategy="constant", fill_value=0)   # fill with a fixed value

X_imputed = imputer.fit_transform(X_train)
```
Same fit-on-train-only, transform-on-test rule applies here as with
scaling — fit the imputer's statistics (mean, median, etc.) only on
training data.

## Applying Different Preprocessing to Different Columns

```python
from sklearn.compose import ColumnTransformer
from sklearn.pipeline import Pipeline

numeric_features = ["age", "income"]
categorical_features = ["city", "education"]

preprocessor = ColumnTransformer(transformers=[
    ("num", StandardScaler(), numeric_features),
    ("cat", OneHotEncoder(handle_unknown="ignore"), categorical_features),
])

X_processed = preprocessor.fit_transform(X_train)
```
`ColumnTransformer` is the standard tool for real-world datasets that mix
numeric and categorical columns needing different treatment — applying a
single `StandardScaler` to the whole DataFrame would fail (or produce
nonsense) on non-numeric columns. See `sklearn-pipelines-model-selection.md`
for combining this with a model into a full `Pipeline`.

## Feature Engineering Helpers

```python
from sklearn.preprocessing import PolynomialFeatures

poly = PolynomialFeatures(degree=2, include_bias=False)
X_poly = poly.fit_transform(X_train)      # adds squared terms and pairwise interaction terms
```
Useful for letting a linear model capture non-linear relationships without
switching to a fundamentally different model type — but feature count grows
quickly with `degree` and number of input columns, worth watching for on
wide datasets.

```python
from sklearn.preprocessing import Binarizer

Binarizer(threshold=0.5).fit_transform(X_train)     # convert continuous values to 0/1 based on a threshold
```

## Target Transformation (For Skewed Regression Targets)

```python
import numpy as np

y_train_log = np.log1p(y_train)          # log(1+x) transform — handles zero values, common for skewed targets (e.g. price)
predictions = np.expm1(model.predict(X_test))    # inverse transform predictions back to the original scale
```
`log1p`/`expm1` (rather than plain `log`/`exp`) are used specifically
because they handle a value of exactly 0 gracefully (`log(0)` is undefined,
but `log1p(0) = 0`) — common when a target variable like sales or price can
legitimately be zero.