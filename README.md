# EXID - Explainable Intrusion Detection

EXID is a notebook-first machine learning project for intrusion detection on the NSL-KDD dataset.
The workflow covers:

- exploratory data analysis (EDA) and feature engineering
- multi-stage LightGBM model training
- explainability with SHAP

The current version is implemented entirely in Jupyter notebooks.

## Table of Contents

- [Project Scope](#project-scope)
- [Repository Structure](#repository-structure)
- [Environment Setup](#environment-setup)
- [Dataset Requirements](#dataset-requirements)
- [Run the Pipeline](#run-the-pipeline)
- [Model Coverage](#model-coverage)
- [Current Results Snapshot](#current-results-snapshot)
- [Outputs and Artifacts](#outputs-and-artifacts)
- [Reproducibility Notes](#reproducibility-notes)
- [Troubleshooting](#troubleshooting)
- [License](#license)

## Project Scope

Goal: build and interpret an intrusion detection pipeline using NSL-KDD.

Main stages:

1. Prepare train/test data with feature engineering and label processing.
2. Train and evaluate multiple classification variants.
3. Explain model behavior with SHAP global importance analysis.

For the latest measured metrics and interpretation notes, see `RESULTS.md`.

## Repository Structure

```text
EXID/
├── data/
│   ├── KDDTrain+.txt
│   ├── KDDTest+.txt
│   ├── ... (other NSL-KDD files and mirrors)
│   └── processed/                  (generated after notebook 01)
│       ├── X_train.parquet
│       ├── X_test.parquet
│       ├── y_train.parquet
│       └── y_test.parquet
├── notebooks/
│   ├── 01_EDA_Feature_Engineering.ipynb
│   └── 02_Model_Training_and_XAI.ipynb
├── requirements.txt
├── RESULTS.md
├── README.md
└── LICENSE
```

Notebook order is strict:

1. `notebooks/01_EDA_Feature_Engineering.ipynb`
2. `notebooks/02_Model_Training_and_XAI.ipynb`

## Environment Setup

### Prerequisites

- macOS/Linux/Windows with Python 3.10+ recommended
- `pip`
- Jupyter frontend (`jupyter lab` or `jupyter notebook`)

### Create and activate a virtual environment

From repository root:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

On Windows PowerShell:

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
```

### Install dependencies

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

If you do not already have a Jupyter frontend installed, add one:

```bash
pip install jupyterlab
```

Optional: register a named kernel for cleaner notebook selection:

```bash
python -m ipykernel install --user --name exid --display-name "EXID (.venv)"
```

### Launch Jupyter

```bash
jupyter lab
```

## Dataset Requirements

Data source: [NSL-KDD on Kaggle](https://www.kaggle.com/datasets/hassan06/nslkdd/data)

At minimum, place these files under `data/`:

- `KDDTrain+.txt`
- `KDDTest+.txt`

This project currently expects relative notebook paths such as `../data/KDDTrain+.txt` and `../data/KDDTest+.txt`.

Notes:

- Raw dataset files (`data/*.txt`, `data/*.arff`, etc.) are gitignored.
- If your local data folder is empty after cloning, download and place the files manually.

## Run the Pipeline

### 1) EDA + Feature Engineering

Open and run:

- `notebooks/01_EDA_Feature_Engineering.ipynb`

What this notebook does (current version):

- reads raw NSL-KDD train/test files
- analyzes class and mapped-class distributions
- removes zero-variance columns
- removes highly correlated numeric features
- applies one-hot encoding and train/test column alignment
- creates encoded targets for downstream models
- writes parquet outputs to `data/processed/`

Expected outputs:

- `data/processed/X_train.parquet`
- `data/processed/X_test.parquet`
- `data/processed/y_train.parquet`
- `data/processed/y_test.parquet`

### 2) Model Training + XAI

Open and run:

- `notebooks/02_Model_Training_and_XAI.ipynb`

This notebook:

- loads `data/processed/*.parquet`
- trains multiple LightGBM classifiers
- evaluates performance (reports + confusion matrices)
- runs SHAP summary-level explainability plots

If notebook 01 has not been run (or output schema changed), notebook 02 will fail at data loading or feature/label assumptions.

## Model Coverage

Current notebook models:

- **Model 0**: binary classification (`Normal` vs `Anomaly`)
- **Model 1**: specific attack-class classification
- **Model 2**: mapped 5-category classification (`Normal`, `DoS`, `Probe`, `R2L`, `U2R`)
- **Model 2.1**: optimized 5-category variant with `SMOTENC`, stronger class weighting, and threshold overrides

Core libraries used:

- `lightgbm`
- `scikit-learn`
- `imbalanced-learn`
- `shap`
- `pandas`, `numpy`, `pyarrow`

## Current Results Snapshot

Detailed model outcomes are maintained in `RESULTS.md`.

Highlights from the current version:

- Stage 1 binary filtering (`Normal` vs `Anomaly`) is tuned as a strong first-pass detector.
- Stage 2 includes an optimized rare-attack recovery setup (`Model 2.1`) with SMOTE-NC, class weights, and threshold overrides.
- Rare-class detection improves notably for `R2L` and `U2R` recall in the maximized protocol, with expected precision tradeoffs on extremely scarce classes.
- SHAP analysis is used to explain feature drivers across attack families and support analyst interpretability.

Use `RESULTS.md` as the source of truth for exact metrics tables and narrative interpretation.

## Outputs and Artifacts

Generated outputs in the current workflow:

- processed parquet datasets in `data/processed/` (after running notebook 01)

Not currently persisted as standalone files:

- trained model binaries (`.pkl`, `.joblib`, or LightGBM text dumps)
- exported SHAP value files
- separate metrics report files

Most metrics and visual outputs are embedded in notebook outputs.
The curated performance summary is tracked in `RESULTS.md`.

## Reproducibility Notes

- Notebook training uses fixed random states (for major modeling/sampling steps).
- Class imbalance handling is explicit (`class_weight` and SMOTENC usage in advanced model).
- Keep notebook execution order unchanged for reproducible behavior.
- Keep dependency versions aligned with `requirements.txt`.

## Troubleshooting

### `FileNotFoundError` for dataset files

- Confirm raw files are in `data/`.
- Confirm you launched Jupyter from repo root so relative paths resolve correctly.

### `FileNotFoundError` for parquet files

- Run `notebooks/01_EDA_Feature_Engineering.ipynb` first to generate `data/processed/*.parquet`.

### Wrong kernel or missing packages in notebooks

- Activate `.venv`.
- Reinstall dependencies: `pip install -r requirements.txt`.
- Select `EXID (.venv)` kernel (or the active venv kernel) in Jupyter.

### Feature mismatch errors between train/test

- Re-run notebook 01 from top to regenerate aligned features before notebook 02.

## License

This project is licensed under the terms in `LICENSE`.
