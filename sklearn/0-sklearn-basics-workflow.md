# scikit-learn Reference — Basics & Workflow

Docs: https://scikit-learn.org/stable/

```python
import sklearn
```
Individual estimators/functions are imported from their specific submodule
(e.g. `from sklearn.linear_model import LinearRegression`) — there's no
single flat `sklearn.SomeModel` import, unlike numpy/pandas' single-alias
convention.

## The Core API Pattern

Nearly every scikit-learn model/transformer follows the same three-method
pattern, which is the single most important thing to internalize about the
library:

```python
model.fit(X, y)              # learn from training data
model.predict(X_new)            # make predictions on new data
model.score(X, y)                  # evaluate performance (metric depends on the estimator type)
```
For transformers (scalers, encoders — see `sklearn-preprocessing.md`):
```python
transformer.fit(X)              # learn parameters from the data (e.g. mean/std for a scaler)
transformer.transform(X)           # apply the learned transformation
transformer.fit_transform(X)          # both steps combined — common shorthand for the training set
```
`X` (capital, since it's conventionally a 2D array/DataFrame) is the feature
matrix; `y` (lowercase, conventionally a 1D array/Series) is the target.
This capitalization convention is followed throughout the ecosystem's
examples and worth adopting for consistency with docs/tutorials.

## Train/Test Split

```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)
```
- `test_size=0.2` — 20% of data held out for testing, 80% for training
- `random_state=42` — seeds the shuffle for reproducibility, same idea as
  NumPy's random seeding (see `numpy-linear-algebra-random.md`)
- `stratify=y` (optional) — preserves the target class proportions in both
  splits, important for imbalanced classification datasets:
  ```python
  train_test_split(X, y, test_size=0.2, stratify=y, random_state=42)
  ```

## A Minimal End-to-End Example

```python
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression
from sklearn.metrics import mean_squared_error, r2_score

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

model = LinearRegression()
model.fit(X_train, y_train)

predictions = model.predict(X_test)

print("R²:", r2_score(y_test, predictions))
print("RMSE:", mean_squared_error(y_test, predictions, squared=False))
```
See `sklearn-models-overview.md` for other model types and
`sklearn-pipelines-model-selection.md` for evaluation metrics in depth.

## Inspecting a Fitted Model

```python
model.coef_             # learned coefficients (linear models)
model.intercept_           # learned intercept (linear models)
model.feature_importances_    # feature importance scores (tree-based models — see sklearn-models-overview.md)
model.classes_                   # the class labels the model learned (classifiers)
```
The trailing underscore (`coef_`, `intercept_`, etc.) is a scikit-learn
naming convention specifically marking attributes that only exist **after**
`.fit()` has been called — accessing them before fitting raises a
`NotFittedError`.

## X as a DataFrame vs. NumPy Array

```python
model.fit(X_train, y_train)          # works whether X_train is a DataFrame or a numpy array
```
Modern scikit-learn accepts pandas DataFrames directly and, since version
1.0+, can preserve feature names through the pipeline (accessible via
`model.feature_names_in_` after fitting) — worth using DataFrames rather
than converting to raw numpy arrays, since it makes debugging/inspection
easier and avoids losing track of which column is which.

## Setting Global Config

```python
from sklearn import set_config
set_config(transform_output="pandas")     # transformers return DataFrames instead of raw numpy arrays
```
This is a relatively recent (scikit-learn 1.2+) quality-of-life setting —
worth enabling globally at the top of a notebook/script if you're working
with pandas throughout, since it keeps column names attached after
`ColumnTransformer`/scaler operations rather than losing them to a plain
numpy array (see `sklearn-preprocessing.md`).

## Saving & Loading a Trained Model

```python
import joblib

joblib.dump(model, "model.joblib")        # save
loaded_model = joblib.load("model.joblib")   # load later, ready to .predict() without retraining
```
`joblib` (not Python's built-in `pickle`) is the current recommended
serialization approach for scikit-learn models specifically — it's more
efficient for objects containing large numpy arrays, which most fitted
models are. Docs: https://scikit-learn.org/stable/model_persistence.html