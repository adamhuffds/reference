# scikit-learn Reference — Models Overview

Docs: https://scikit-learn.org/stable/supervised_learning.html

Prerequisite: `sklearn-basics-workflow.md`, `sklearn-preprocessing.md`

All models below follow the same `.fit(X, y)` / `.predict(X)` pattern from
`sklearn-basics-workflow.md` — this file focuses on what each model is,
when to reach for it, and its distinctive parameters.

## Regression Models

```python
from sklearn.linear_model import LinearRegression, Ridge, Lasso, ElasticNet

LinearRegression()      # baseline — no regularization, fast, interpretable, sensitive to multicollinearity
Ridge(alpha=1.0)           # L2 regularization — shrinks coefficients, handles multicollinearity better
Lasso(alpha=1.0)              # L1 regularization — can shrink coefficients to EXACTLY zero (built-in feature selection)
ElasticNet(alpha=1.0, l1_ratio=0.5)   # combines L1 + L2 — l1_ratio controls the mix
```
`alpha` controls regularization strength — higher values = more shrinkage
(simpler model, less overfitting risk, but more bias). Tune it via
cross-validation (see `sklearn-pipelines-model-selection.md`) rather than
guessing.

```python
from sklearn.tree import DecisionTreeRegressor
from sklearn.ensemble import RandomForestRegressor, GradientBoostingRegressor

DecisionTreeRegressor(max_depth=5)               # single tree — interpretable, prone to overfitting if unconstrained
RandomForestRegressor(n_estimators=100)             # ensemble of trees — generally strong default, less overfitting
GradientBoostingRegressor(n_estimators=100)            # sequential ensemble — often higher accuracy, slower to train,
                                                           # more sensitive to hyperparameter tuning than RandomForest
```

## Classification Models

```python
from sklearn.linear_model import LogisticRegression

LogisticRegression(max_iter=1000)
```
Despite the name, `LogisticRegression` is a **classification** algorithm,
not regression — a common naming confusion for newcomers. `max_iter` often
needs raising above the default (100) to reach convergence on real datasets
— a `ConvergenceWarning` is your signal to increase it or scale the
features first (see `sklearn-preprocessing.md`).

```python
from sklearn.tree import DecisionTreeClassifier
from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier

DecisionTreeClassifier(max_depth=5)
RandomForestClassifier(n_estimators=100)
GradientBoostingClassifier(n_estimators=100)
```
Same trade-offs as the regression versions above — RandomForest is a strong
general-purpose default; GradientBoosting often edges out RandomForest on
accuracy with more careful tuning.

```python
from sklearn.svm import SVC
from sklearn.neighbors import KNeighborsClassifier
from sklearn.naive_bayes import GaussianNB

SVC(kernel="rbf", C=1.0)              # Support Vector Classifier — strong on smaller/high-dimensional datasets,
                                          # requires feature scaling (see sklearn-preprocessing.md)
KNeighborsClassifier(n_neighbors=5)      # simple, no real "training" step, but slow at prediction time on large data,
                                             # also requires feature scaling since it's distance-based
GaussianNB()                                   # fast, works well on small data / text classification baselines,
                                                   # assumes feature independence (often violated, still works OK in practice)
```

## Tree-Based Models & Feature Scaling

Worth calling out explicitly: **tree-based models (Decision Tree, Random
Forest, Gradient Boosting) do not require feature scaling** — they split on
raw thresholds regardless of scale. Distance-based models (SVM, k-NN) and
linear/logistic regression **do** require scaling. Skipping scaling for the
latter group is a common, quietly-degrading mistake — the model still runs
without error, just performs worse than it should.

## Clustering (Unsupervised)

```python
from sklearn.cluster import KMeans, DBSCAN

kmeans = KMeans(n_clusters=3, random_state=42, n_init="auto")
kmeans.fit(X)
labels = kmeans.labels_               # cluster assignment per row
kmeans.cluster_centers_                  # coordinates of each cluster's center

dbscan = DBSCAN(eps=0.5, min_samples=5)     # density-based — doesn't require specifying cluster count upfront,
                                                 # can also flag points as noise/outliers (label -1)
dbscan.fit(X)
```
Clustering has no `y` — `.fit(X)` only, since there's no target to predict
against. `KMeans` requires choosing `n_clusters` ahead of time (see the
elbow method / silhouette score in `sklearn-pipelines-model-selection.md`
for picking it); `DBSCAN` doesn't, but is more sensitive to its `eps`/
`min_samples` parameters.

## Dimensionality Reduction

```python
from sklearn.decomposition import PCA

pca = PCA(n_components=2)             # reduce to 2 dimensions, e.g. for visualization
X_reduced = pca.fit_transform(X_scaled)   # ALWAYS scale features before PCA — it's variance-sensitive

pca.explained_variance_ratio_            # how much variance each component captures
```
PCA is variance-based, so unscaled features with larger numeric ranges will
dominate the components regardless of their actual importance — always run
`StandardScaler` before PCA (ties back to `sklearn-preprocessing.md`).

## Choosing a Model: Quick Heuristics

- **Start simple** — linear/logistic regression as a baseline before
  reaching for anything more complex; if a simple model performs nearly as
  well, the added complexity of an ensemble may not be worth it
- **Tabular data, unsure where to start** — `RandomForestRegressor`/
  `RandomForestClassifier` is a strong, low-maintenance default
- **Need interpretability** — linear models or a single shallow
  `DecisionTree` over an ensemble, since ensemble predictions are harder to
  explain feature-by-feature
- **Small dataset, high dimensionality** — `SVC` or regularized linear
  models (`Ridge`/`Lasso`) tend to generalize better than tree ensembles
- **Squeezing out accuracy, willing to tune** — gradient boosting (or
  `xgboost`/`lightgbm`, outside scikit-learn itself but following the same
  `.fit()`/`.predict()` API convention)