# Bridging nanotumor pharmacokinetics and transcriptomic toxicity responses — code and data

This repository contains the datasets, trained model artifacts, and analysis code for the
two-stage machine-learning framework described in the manuscript *"Bridging Nanotumor
Pharmacokinetics and Transcriptomic Toxicity Responses via Predicted Tumor Delivery
Efficiency: A Machine Learning Framework for Nanosafety Endpoint Classification"*
(NanoImpact).

All modelling uses a fixed random seed (`SEED = 42`).

## What the framework does

* **Stage 1 — delivery model.** A bagged gradient-boosted tree ensemble (XGBoost in a
10-estimator `BaggingRegressor`) predicts tumour **delivery efficiency (DEmax)** from
nanoparticle physicochemical and tumour descriptors. Model A uses 10 features; Model B
is a transferable 4-feature variant (`HD, Zeta, Shape, CT`).
* **Stage 2 — endpoint classification.** Predicted delivery efficiency is transferred to a
curated transcriptomics dataset and used, with physicochemical descriptors, to classify
nine binary toxicity endpoints; associations are tested with Mann–Whitney tests
(rank-biserial effect sizes), multiplicity correction, a pooled GEE interaction test,
a DEmax\_pred ablation, and group-aware cross-validation.

## Repository layout

```
.
├── README.md
├── requirements.txt
├── LICENSE
├── code/
│   ├── 01\_delivery\_model\_comparison\_and\_selection.ipynb
│   └── 02\_endpoint\_classification\_and\_statistics.ipynb
├── data/
│   ├── AI\_nano.xlsx                      # Stage-1 delivery dataset (378×25)
│   ├── Literature\_transcriptomics\_review.xlsx   # Stage-2 endpoint dataset
│   ├── cellline\_to\_CT\_mapping.csv        # Table S3 lookup
│   ├── MAPPING\_AND\_FALLBACK\_NOTES.md     # mapping + fallback + exclusions
│   └── DATA\_DICTIONARY.md
└── models/                               # produced by notebook 02 (see below)
    ├── model\_Demax\_full.joblib
    ├── model\_Demax\_full\_gridsearch.joblib
    ├── scaler\_X\_full.joblib
    ├── scaler\_Y\_full.joblib
    ├── model\_Demax\_B.joblib
    ├── scaler\_X\_B.joblib
    └── scaler\_Y\_B.joblib
```

## How to run

1. Install dependencies: `pip install -r requirements.txt` (Python 3.10+).
For an exact reproduction, pin the versions from the environment that produced the
results (`pip freeze`), as noted in `requirements.txt`.
2. Set the data location. Each notebook begins with a config cell:

```python
   DATA\_DIR = '/content/drive/MyDrive/datasets'   # Colab default
   # local run:  DATA\_DIR = './data'  (and skip the drive.mount cell)
   ```

   Point `DATA\_DIR` at the folder holding `AI\_nano.xlsx` and
`Literature\_transcriptomics\_review.xlsx`.

3. Run the notebooks (the two are independent; this is the logical reading order):

   * `code/01\_…` reproduces the five-model comparison across the pharmacokinetic targets
(manuscript Table S2), hyperparameter tuning, the dose-independence check, and the
leakage-free cross-validation re-run. Outputs `Table2\_multitarget\_results.csv`,
`hyperparameter\_summary.csv`.
   * `code/02\_…` is the full pipeline: trains and **saves** the delivery models, applies the
cell-line → CT mapping, generates `DEmax\_pred`, trains the nine endpoint classifiers,
and runs the endpoint statistics. It writes all intermediate CSVs it needs (no
external CSV inputs).

## Regenerating the model artifacts (`models/`)

The `.joblib` files are **created when `code/02\_…` runs** — they are not stored by hand.
To regenerate them: open `02\_endpoint\_classification\_and\_statistics.ipynb`, set `DATA\_DIR`,
**Run all**; the `joblib.dump(...)` cells write the seven artifacts listed above to the
working directory. Download them into `models/`. The scalers (`scaler\_X\_\*`, `scaler\_Y\_\*`)
are required — the models predict on standardised inputs and are unusable without them.
The nine endpoint classifiers are retrained in-notebook under `SEED = 42` rather than saved.

## Data provenance

**Delivery dataset (`AI\_nano.xlsx`).** Curated/derived from the published nanoparticle
tumour-delivery studies of Lin et al. 2022 (*Int. J. Nanomedicine* 17:1365–1379) and
Cheng et al. 2020 (*ACS Nano* 14:3075–3095).

**Transcriptomics endpoint dataset (`Literature\_transcriptomics\_review.xlsx` /
`transcriptomics\_endpoints.csv`).** Curated by the authors from published nanoparticle
transcriptomics (microarray / RNA-seq) toxicity studies. Each record is a binary
pathway-activation call (`triggered` / `no\_effect`) for nine toxicity endpoints, abstracted
from the original studies — the dataset contains these curated endpoint labels, not raw
gene-expression values. The underlying per-study sources, differentially expressed genes,
pathway interpretation, external validation and data-availability notes are recorded in the
`Literature Transcriptomics` sheet of `Literature\_transcriptomics\_review.xlsx`, which serves
as the provenance record for this dataset.

See `data/DATA\_DICTIONARY.md` for full column definitions and
`data/MAPPING\_AND\_FALLBACK\_NOTES.md` for the cell-line → CT mapping and exclusions.

## License

Released under the MIT License (see `LICENSE`).

## Citation

Repository: https://github.com/monrudee-li/nanotumor-demax-framework



