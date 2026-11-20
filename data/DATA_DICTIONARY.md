# Data dictionary

Three data objects are deposited. All modelling uses `SEED = 42`.

---

## 1. `AI_nano.xlsx` — nanotumor delivery dataset (Stage 1)

Sheet `Model_dataset`, 378 rows × 25 columns; 288 rows remain after dropping records with
missing values in the 10 model features or the target. Curated/derived from the published
nanoparticle tumour-delivery meta-analysis of Lin et al. 2022 and Cheng et al. 2020
(references [17] and [21] in the manuscript).

**Model A input features (10):**

| Column | Meaning | Type |
|---|---|---|
| Type | Nanoparticle type (organic / inorganic family) | categorical |
| TS | Targeting strategy (active / passive) | categorical |
| HD | Hydrodynamic diameter (nm); modelled as log-HD | continuous |
| Zeta | Zeta potential (mV) | continuous |
| Charge | Surface charge sign | ordinal |
| Shape | Nanoparticle shape | categorical |
| TM | Tumour model | categorical |
| CT | Cell type / cancer type (nanotumor category) | categorical |
| TSz | Tumour size (cm) | continuous |
| TW | Tumour weight (g) | continuous |

**Model B** uses the transferable 4-feature subset: `HD, Zeta, Shape, CT`.

**Targets / other columns:**

| Column | Meaning |
|---|---|
| Demax | **Delivery efficiency** — maximum % injected dose reaching the tumour (the Stage-1 target). Range 0.008–30.24 for the retained set; full 0.008–56.89 across all delivery metrics. |
| DE24, DE168 | Delivery efficiency at 24 h and 168 h (%ID) |
| KTRESrelease, KTRESmax, KTRES50, KTRESn | Tumour release-kinetic parameters (additional regression targets in Table S2) |
| QTC, VTC, BVT, PT, PATC | PBPK-derived tumour parameters |
| Dose | Administered dose (used only to confirm Demax is dose-independent; Pearson r = −0.003) |
| BW | Animal body weight |
| ID | Record identifier |

`Demax` is a **tissue-level delivery-efficiency** metric (%ID), **not** cellular uptake.

---

## 2. `Literature_transcriptomics_review.xlsx` — transcriptomics endpoint dataset (Stage 2)

- Sheet `Processed and Rel Dataset` — 245 data rows (a two-row grouped header). Columns
  include nanoparticle physico-chemical properties (Core size, Shape, Surface coatings,
  Zeta potential, specific surface area), exposure conditions (dose, duration), in-vitro
  model characteristics (Tissue, **Cell type**, **Cell line**), and the **nine binary
  toxicity endpoint calls** (triggered / no-effect): Angiogenesis; Adhesion/ECM; Cell
  cycle & proliferation; Cell death & apoptosis; Cytoskeleton organisation; DNA damage &
  repair; Inflammation & immune; Metal-ion responses; UPR/ER stress.
- Sheet `Literature Transcriptomics` — the per-study curation/provenance record
  (DEGs, pathway interpretation, validation, data availability), kept as documentation.

The endpoints are **curated binary pathway-activation labels**, not gene-expression
values; the modelling is an endpoint-classification task over these labels.

> For a FAIR, machine-readable version, use the flattened single-header CSV export
> (`transcriptomics_endpoints.csv`) if provided; the `.xlsx` above is the original.

---

## 3. `cellline_to_CT_mapping.csv` — Table S3 (cell-line → CT lookup)

| Column | Meaning |
|---|---|
| CellLine identifier | Cell-line string as it appears in the literature (incl. alternate spellings) |
| Assigned CT | Solid-tumour category assigned to Model B's `CT` feature |
| Cell line description | Brief description |
| Reference/Source | Repository / catalogue identifier (e.g. ATCC) |

The matching rules, the `"Breast"` fallback, case normalisation, the material-class
fallback for unknown CT, and the excluded non-solid-tumour lineages (Table S4) are
described in `MAPPING_AND_FALLBACK_NOTES.md`.
