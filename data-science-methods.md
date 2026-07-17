# Data Science & AI/ML Technique Map

A working reference of techniques organized from foundational to advanced. Meant as a
scaffold — check things off, add notes/links as you go deeper on each.

---

## 1. Foundations: Data Handling & Statistics

**Data wrangling**
- Cleaning: missing data (deletion, mean/median/mode imputation, KNN imputation, MICE),
  outlier detection (IQR, z-score, isolation forest), deduplication
- Reshaping: pivot/melt (wide ↔ long), joins/merges, groupby-aggregate
- Encoding: one-hot, ordinal, target/mean encoding, hashing trick
- Scaling/normalization: min-max, standardization (z-score), robust scaling, log/Box-Cox
  transforms

**Descriptive statistics**
- Central tendency & spread: mean, median, mode, variance, std dev, skewness, kurtosis
- Distributions: normal, binomial, Poisson, exponential, uniform — knowing which process
  generates which shape
- Correlation vs. covariance (Pearson, Spearman, Kendall's tau)

**Inferential statistics**
- Hypothesis testing: t-tests (one-sample, two-sample, paired), chi-square, ANOVA/MANOVA
- Confidence intervals, p-values, effect size (Cohen's d) — and their common misuses
- Power analysis / sample size determination
- Multiple comparisons correction (Bonferroni, Benjamini-Hochberg/FDR) — very relevant
  for any -omics or high-throughput screening work
- Non-parametric alternatives: Mann-Whitney U, Kruskal-Wallis, Wilcoxon signed-rank

**Experimental design**
- A/B testing, randomized controlled trials, factorial designs
- Confounding, causal vs. correlational reasoning, Simpson's paradox
- Bootstrapping and permutation testing

---

## 2. Core Machine Learning

**Supervised learning — regression**
- Linear regression (OLS), regularized variants: Ridge (L2), Lasso (L1), Elastic Net
- Polynomial regression, spline/GAM-based nonlinear regression
- Regression trees

**Supervised learning — classification**
- Logistic regression
- k-Nearest Neighbors (KNN)
- Naive Bayes (Gaussian, Multinomial, Bernoulli)
- Support Vector Machines (linear & kernel: RBF, polynomial)
- Decision trees

**Ensemble methods**
- Bagging: Random Forest
- Boosting: AdaBoost, Gradient Boosting, XGBoost, LightGBM, CatBoost
- Stacking / blending

**Unsupervised learning**
- Clustering: k-means, hierarchical (agglomerative), DBSCAN, Gaussian Mixture Models
- Dimensionality reduction: PCA, t-SNE, UMAP, Factor Analysis
- Association rule mining: Apriori, FP-Growth

**Model evaluation & validation**
- Train/val/test splits, k-fold and stratified cross-validation, leave-one-out
- Classification metrics: accuracy, precision, recall, F1, ROC-AUC, PR-AUC, confusion matrix
- Regression metrics: MSE, RMSE, MAE, R², adjusted R²
- Bias-variance tradeoff, overfitting/underfitting diagnostics (learning curves)
- Hyperparameter tuning: grid search, random search, Bayesian optimization (Optuna,
  Hyperopt)

**Feature engineering & selection**
- Filter methods (correlation, chi-square, mutual information)
- Wrapper methods (recursive feature elimination)
- Embedded methods (Lasso coefficients, tree feature importances)
- SHAP / permutation importance for interpretability
- Domain-driven feature construction (this is where your protein/assay domain knowledge
  becomes a real modeling advantage over generic pipelines)

---

## 3. Deep Learning

**Fundamentals**
- Perceptron → Multi-Layer Perceptron (MLP)
- Backpropagation, gradient descent variants (SGD, Momentum, RMSProp, Adam)
- Activation functions: ReLU/leaky ReLU, sigmoid, tanh, GELU, softmax
- Regularization: dropout, batch norm/layer norm, weight decay, early stopping

**Architectures**
- Convolutional Neural Networks (CNNs) — image data, also 1D CNNs for sequence/signal data
- Recurrent architectures: RNN, LSTM, GRU — largely superseded but still conceptually
  foundational
- Transformers: self-attention, multi-head attention, positional encoding — the backbone
  of essentially everything current in NLP and increasingly in structural biology
  (AlphaFold's Evoformer is attention-based)
- Autoencoders (vanilla, denoising, variational/VAE)
- Graph Neural Networks (GNNs) — highly relevant for molecular/protein structure data

**Generative models**
- GANs (Generative Adversarial Networks)
- Variational Autoencoders (VAEs)
- Diffusion models (the current state of the art for image/structure generation —
  conceptually related to how RFdiffusion works for protein design)

---

## 4. NLP & Language Models

- Classical: bag-of-words, TF-IDF, n-grams
- Word embeddings: Word2Vec, GloVe, FastText
- Contextual embeddings & transformer LMs: BERT (encoder-only), GPT-family (decoder-only),
  T5 (encoder-decoder)
- Fine-tuning approaches: full fine-tuning, LoRA/QLoRA (parameter-efficient fine-tuning),
  prompt tuning
- Retrieval-Augmented Generation (RAG) — vector embeddings + similarity search (FAISS,
  pgvector) feeding an LLM
- Evaluation: perplexity, BLEU/ROUGE, human eval, LLM-as-judge

---

## 5. Time Series & Sequential Data

- Classical: ARIMA/SARIMA, exponential smoothing (Holt-Winters)
- Decomposition: trend/seasonality/residual (STL)
- Stationarity testing (ADF, KPSS), autocorrelation (ACF/PACF)
- Modern: Prophet, LSTM/GRU for sequences, Temporal Fusion Transformers
- Anomaly detection in time series (control charts, isolation forest, seasonal-hybrid ESD)

---

## 6. Specialized Techniques Relevant to Your Domain

Given your structural biology / protein science background, worth tracking separately:

- **Sequence & structure modeling**: protein language models (ESM, ProtT5), AlphaFold/
  AlphaFold-Multimer confidence metrics (pLDDT, PAE), RoseTTAFold
- **Generative protein design**: RFdiffusion, ProteinMPNN (inverse folding)
- **QSAR/cheminformatics**: molecular descriptors, fingerprints (Morgan/ECFP), docking
  scores as features — you've already built this pipeline once (D2 receptor project)
- **Graph-based molecular ML**: GNNs over molecular graphs, message-passing networks
- **Survival analysis**: Kaplan-Meier, Cox proportional hazards — common in assay/stability
  time-to-event data

---

## 7. MLOps & Production (Data Engineering side)

- Experiment tracking: MLflow, Weights & Biases
- Pipeline orchestration: Airflow, Prefect, Snakemake (you've already used Snakemake for
  RNA-seq planning)
- Model serving: FastAPI/Flask endpoints, batch vs. real-time inference
- Data/model versioning: DVC
- Monitoring: data drift, concept drift, model performance decay
- Containerization/reproducibility: Docker, conda-lock/environment.yml

---

## 8. Explainability & Responsible ML

- SHAP, LIME — local and global feature attribution
- Partial dependence plots / ICE plots
- Fairness metrics (demographic parity, equalized odds) — less central to your current
  work, but standard in the broader field
- Uncertainty quantification: conformal prediction, Bayesian approaches, ensemble variance

---

## How to use this

A reasonable way to work through it without getting overwhelmed:
1. Mark what you already have hands-on experience with (you're solidly through most of
   Section 2, parts of 5 and 7 already from your portfolio projects).
2. Pick one item per section to go deep on next, rather than trying to broaden everywhere
   at once.
3. Add a "docs/notes" column per technique as you study it — link to the official docs,
   plus one dataset/mini-project where you applied it.

Want me to turn this into a tracked spreadsheet (progress status, docs links, project
tie-ins) instead of a static markdown file? That'd make it easier to maintain as you go.