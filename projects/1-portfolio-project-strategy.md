# Portfolio Strategy & Kaggle API Notes

Reference notes from a working session on building a data science portfolio during a biochemistry → data engineering/data science career transition.

---

## 1. Portfolio Strategy Overview

**Context:** Transitioning from biochemistry into data engineering/data science, building a portfolio to showcase MSDS coursework plus independently-learned tools (Flask, Docker, etc.).

**Core portfolio components discussed:**
- End-to-end ETL pipeline project (already in progress — going well)
- A deployed Flask web application serving predictions or insights
- A Dockerized project demonstrating production-readiness
- A statistical/ML modeling project (this session's focus)

**Key differentiator:** Domain expertise from biochemistry (experimental design, quantitative measurement, analytical rigor) can set a portfolio apart — but the decision here was to start with **classical data science** rather than domain-heavy scientific modeling, to build general DS fundamentals first.

---

## 2. Modeling Project — Getting Unstuck

**Problem identified:** Difficulty finding a good starting dataset, not lack of technique knowledge.

**Chosen focus for first modeling project:** Regression (classical, not domain-specific bio/chem modeling for now).

**Criteria for a good portfolio dataset:**
- Enough complexity to demonstrate real skills (not a toy/tutorial dataset)
- Tells a clear, coherent story or solves a meaningful problem
- Reasonably well-documented (avoid excessive data cleaning/archaeology as the whole project)
- Ideally something genuinely interesting to sustain motivation

**Dataset sources considered:**
| Source | Notes |
|---|---|
| [Kaggle](https://www.kaggle.com/datasets) | Chosen source — DS-specific, good documentation, API access |
| [UCI ML Repository](https://archive.ics.uci.edu/ml/index.php) | Academic standard, well-documented |
| [Data.gov](https://data.gov) | Real-world government data (health, environment, economics) |
| PubChem / Protein Data Bank / GEO | Domain-specific (bio/chem) — set aside for now per preference to avoid hardcore scientific modeling |

**Example regression dataset ideas discussed:** house price prediction (e.g. Ames housing), used car price prediction, insurance cost prediction, bike sharing demand.

---

## 3. Kaggle API — Setup

### Installation
```bash
pip install kaggle
```

### Authentication
1. Go to [Kaggle account settings](https://www.kaggle.com/settings) → "Create New API Token"
2. This downloads a `kaggle.json` credentials file
3. Place and secure it on Ubuntu:

```bash
mkdir -p ~/.kaggle              # Create the config directory if it doesn't exist
mv ~/Downloads/kaggle.json ~/.kaggle/   # Move credentials into place
chmod 600 ~/.kaggle/kaggle.json # Restrict permissions: readable/writable only by you (security best practice for credential files)
```

**Reference:** [Kaggle API GitHub](https://github.com/Kaggle/kaggle-api)

---

## 4. Kaggle API Structure

The API is organized around four main resource types:
1. **Datasets** — static data uploads (primary focus for portfolio work)
2. **Competitions** — contest data with leaderboards
3. **Kernels/Notebooks** — shared code notebooks on Kaggle
4. **Models** — pre-trained models in the Kaggle Model Hub

---

## 5. Core CLI Commands

```bash
# Search for datasets
kaggle datasets list -s "regression" --sort-by hotness
# -s          : search term to filter dataset results
# --sort-by   : ordering method (hotness, votes, updated, active)
# Purpose: quickly surface popular, well-maintained datasets matching a keyword

# Get metadata about a specific dataset before downloading
kaggle datasets metadata <owner>/<dataset-name>
# Returns file sizes, column descriptions, update frequency
# Purpose: sanity-check a dataset before committing to a download

# Download and unzip an entire dataset
kaggle datasets download -d <owner>/<dataset-name> -p ./data --unzip
# -d       : dataset identifier, formatted as owner/dataset-name
# -p       : local path to save files (defaults to current directory)
# --unzip  : automatically extract any compressed archives

# Download a single file from a (possibly large) dataset
kaggle datasets download -d <owner>/<dataset-name> -f <filename> -p ./data
# -f : name of the specific file to pull, avoids downloading the whole archive
```

---

## 6. Python API — Reproducible Workflow

The Python API is preferred over ad hoc CLI use for portfolio projects because it can be scripted and version-controlled.

### Authenticate
```python
from kaggle.api.kaggle_api_extended import KaggleApi

api = KaggleApi()
api.authenticate()  # Reads credentials from ~/.kaggle/kaggle.json
```

### Search datasets
```python
datasets = api.dataset_list(search='regression', sort_by='hotness')
# Returns a list of dataset objects with metadata attributes

for dataset in datasets[:5]:
    print(f"Title: {dataset.title}")
    print(f"Owner: {dataset.ref}")           # This is the owner/dataset-name identifier needed for downloads
    print(f"Size: {dataset.totalBytes / 1e6:.2f} MB")
    print(f"Last updated: {dataset.lastUpdated}")
    print("---")
```

### Download a full dataset
```python
api.dataset_download_files(
    'owner/dataset-name',
    path='./data/raw',   # Destination directory
    unzip=True            # Extract any archive automatically
)
```

### Download a single file (efficient for large datasets)
```python
api.dataset_download_file(
    'owner/dataset-name',
    file_name='train.csv',
    path='./data/raw'
)
```

---

## 7. Reusable Project Scripts

### `data/download_data.py` — scripted, reproducible data acquisition
```python
"""
Script to download dataset from Kaggle
Run once to set up project data
"""
from kaggle.api.kaggle_api_extended import KaggleApi
import os

def download_kaggle_dataset(dataset_ref, data_dir='./data/raw'):
    """
    Download dataset from Kaggle

    Args:
        dataset_ref (str): Kaggle dataset reference (owner/dataset-name)
        data_dir (str): Directory to save data
    """
    os.makedirs(data_dir, exist_ok=True)  # Create directory if it doesn't exist

    api = KaggleApi()
    api.authenticate()

    print(f"Downloading {dataset_ref}...")
    api.dataset_download_files(dataset_ref, path=data_dir, unzip=True)
    print(f"Download complete. Data saved to {data_dir}")

if __name__ == "__main__":
    DATASET_REF = "owner/dataset-name"  # Set the chosen dataset here
    download_kaggle_dataset(DATASET_REF)
```

### Dataset provenance documentation (for README)
```markdown
## Data Source
Dataset: [Dataset Name]
Kaggle URL: https://www.kaggle.com/datasets/owner/dataset-name
Downloaded: YYYY-MM-DD
License: [Check dataset license on Kaggle]

To reproduce:
1. Set up Kaggle API credentials
2. Run: python data/download_data.py
```

### Inspect before downloading
```python
def inspect_dataset(dataset_ref):
    """
    Inspect dataset before downloading.
    Useful for large datasets to preview contents/size first.
    """
    api = KaggleApi()
    api.authenticate()

    dataset = api.dataset_view(dataset_ref)

    print(f"Title: {dataset.title}")
    print(f"Description: {dataset.description}")
    print(f"Size: {dataset.totalBytes / 1e6:.2f} MB")

    files = api.dataset_list_files(dataset_ref).files
    for f in files:
        print(f"  - {f.name} ({f.totalBytes / 1e6:.2f} MB)")

    return dataset

inspect_dataset('owner/dataset-name')
```

### Search/filter helper for candidate datasets
```python
def find_regression_datasets(min_size_mb=1, max_size_mb=500):
    """
    Search for suitable regression datasets, filtered by size
    to avoid both toy datasets and unwieldy large downloads.

    Args:
        min_size_mb: Minimum dataset size in MB
        max_size_mb: Maximum dataset size in MB
    """
    api = KaggleApi()
    api.authenticate()

    datasets = api.dataset_list(search='regression', sort_by='votes')

    suitable = []
    for ds in datasets:
        size_mb = ds.totalBytes / 1e6
        if min_size_mb <= size_mb <= max_size_mb:
            suitable.append({
                'ref': ds.ref,
                'title': ds.title,
                'size_mb': round(size_mb, 2),
                'votes': ds.voteCount,
                'url': f"https://www.kaggle.com/datasets/{ds.ref}"
            })

    return suitable[:10]  # Top 10 results

candidates = find_regression_datasets()
for i, ds in enumerate(candidates, 1):
    print(f"{i}. {ds['title']}")
    print(f"   Size: {ds['size_mb']} MB | Votes: {ds['votes']}")
    print(f"   URL: {ds['url']}\n")
```

---

## 8. Best Practices Recap

- **Script data acquisition** — don't manually click-download; use `download_data.py` style scripts for reproducibility
- **Document data provenance** in a README (source URL, download date, license)
- **Download once, version metadata not raw data** — add `data/` to `.gitignore`, don't commit large raw datasets to git
- **Inspect before downloading** — use `dataset_view` / `dataset_list_files` to check size and contents first
- **Respect API rate limits** — generally generous for personal/portfolio use, but avoid re-downloading on every run

---

## 9. Reference Links
- Kaggle API GitHub (full docs): https://github.com/Kaggle/kaggle-api
- Kaggle account settings (API token creation): https://www.kaggle.com/settings
- UCI Machine Learning Repository: https://archive.ics.uci.edu/ml/index.php
- Data.gov: https://data.gov

---

## 10. Open Next Step
Still to do: search for and select a specific regression dataset via the API, then set up the full project structure (directory layout, environment, reproducible download script, EDA notebook).