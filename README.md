# Bridging nanotumor pharmacokinetics and transcriptomic toxicity responses — code and data

This repository contains the datasets, trained model artifacts, and analysis code for the
two-stage machine-learning framework described in the manuscript *"Bridging Nanotumor
Pharmacokinetics and Transcriptomic Toxicity Responses via Predicted Tumor Delivery
Efficiency: A Machine Learning Framework for Nanosafety Endpoint Classification"*
(NanoImpact).

All modelling uses a fixed random seed (`SEED = 42`).

## What the framework does
- **Stage 1 — delivery model.** A bagged gradient-boosted tree ensemble (XGBoost in a
  10-estimator `BaggingRegressor`) predicts tumour **delivery efficiency (DEmax)** from
  nanoparticle physicochemical and tumour descriptors. Model A uses 10 features; Model B
  is a transferable 4-feature variant (`HD, Zeta, Shape, CT`).
- **Stage 2 — endpoint classification.** Predicted delivery efficiency is transferred to a
  curated transcriptomics dataset and used, with physicochemical descriptors, to classify
  nine binary toxicity endpoints; associations are tested with Mann–Whitney tests
  (rank-biserial effect sizes), multiplicity correction, a pooled GEE interaction test,
  a DEmax_pred ablation, and group-aware cross-validation.

## Repository layout
```
.
├── README.md
├── requirements.txt
├── LICENSE
├── code/
│   ├── 01_delivery_model_comparison_and_selection.ipynb
│   └── 02_endpoint_classification_and_statistics.ipynb
├── data/
│   ├── AI_nano.xlsx                      # Stage-1 delivery dataset (378×25)
│   ├── Literature_transcriptomics_review.xlsx   # Stage-2 endpoint dataset
│   ├── cellline_to_CT_mapping.csv        # Table S3 lookup
│   ├── MAPPING_AND_FALLBACK_NOTES.md     # mapping + fallback + exclusions
│   └── DATA_DICTIONARY.md
└── models/                               # produced by notebook 02 (see below)
    ├── model_Demax_full.joblib
    ├── model_Demax_full_gridsearch.joblib
    ├── scaler_X_full.joblib
    ├── scaler_Y_full.joblib
    ├── model_Demax_B.joblib
    ├── scaler_X_B.joblib
    └── scaler_Y_B.joblib
```

## How to run
1. Install dependencies: `pip install -r requirements.txt` (Python 3.10+).
   For an exact reproduction, pin the versions from the environment that produced the
   results (`pip freeze`), as noted in `requirements.txt`.
2. Set the data location. Each notebook begins with a config cell:
   ```python
   DATA_DIR = '/content/drive/MyDrive/datasets'   # Colab default
   # local run:  DATA_DIR = './data'  (and skip the drive.mount cell)
   ```
   Point `DATA_DIR` at the folder holding `AI_nano.xlsx` and
   `Literature_transcriptomics_review.xlsx`.
3. Run the notebooks (the two are independent; this is the logical reading order):
   - `code/01_…` reproduces the five-model comparison across the pharmacokinetic targets
     (manuscript Table S2), hyperparameter tuning, the dose-independence check, and the
     leakage-free cross-validation re-run. Outputs `Table2_multitarget_results.csv`,
     `hyperparameter_summary.csv`.
   - `code/02_…` is the full pipeline: trains and **saves** the delivery models, applies the
     cell-line → CT mapping, generates `DEmax_pred`, trains the nine endpoint classifiers,
     and runs the endpoint statistics. It writes all intermediate CSVs it needs (no
     external CSV inputs).

## Regenerating the model artifacts (`models/`)
The `.joblib` files are **created when `code/02_…` runs** — they are not stored by hand.
To regenerate them: open `02_endpoint_classification_and_statistics.ipynb`, set `DATA_DIR`,
**Run all**; the `joblib.dump(...)` cells write the seven artifacts listed above to the
working directory. Download them into `models/`. The scalers (`scaler_X_*`, `scaler_Y_*`)
are required — the models predict on standardised inputs and are unusable without them.
The nine endpoint classifiers are retrained in-notebook under `SEED = 42` rather than saved.

## Data provenance
**Delivery dataset (`AI_nano.xlsx`).** Curated/derived from the published nanoparticle
tumour-delivery studies of Lin et al. 2022 (*Int. J. Nanomedicine* 17:1365–1379) and
Cheng et al. 2020 (*ACS Nano* 14:3075–3095).

**Transcriptomics endpoint dataset (`Literature_transcriptomics_review.xlsx` /
`transcriptomics_endpoints.csv`).** Curated by the authors from published nanoparticle
transcriptomics (microarray / RNA-seq) toxicity studies. Each record is a binary
pathway-activation call (`triggered` / `no_effect`) for nine toxicity endpoints, abstracted
from the original studies — the dataset contains these curated endpoint labels, not raw
gene-expression values. The underlying per-study sources, differentially expressed genes,
pathway interpretation, external validation and data-availability notes are recorded in the
`Literature Transcriptomics` sheet of `Literature_transcriptomics_review.xlsx`, which serves
as the provenance record for this dataset.

See `data/DATA_DICTIONARY.md` for full column definitions and
`data/MAPPING_AND_FALLBACK_NOTES.md` for the cell-line → CT mapping and exclusions.

## License
Released under the MIT License (see `LICENSE`).

## Citation
If you use this material, please cite the manuscript and this archive:
- Repository: https://github.com/monrudee-li/nanotumor-demax-framework
- Archive: Zenodo DOI (added on release)
