# Cell-line → CT mapping and fallback logic

Model B transfers the delivery model to the transcriptomics records by supplying the
`CT` (cell type) feature from the literature `Cell line` field. The mapping and the
fallback rules used are documented here so the transfer is fully reproducible.
(`cellline_to_CT_mapping.csv` in this folder is the lookup table, i.e. Table S3.)

## Step 1 — direct lookup
Each record's `Cell line` string is matched against the `CellLine identifier` column of
`cellline_to_CT_mapping.csv` to obtain the `Assigned CT` (a solid-tumour category:
Breast, Colon, Lung, Skin, Liver, Kidney, Prostate, Brain, Ovary, Pancreas, …).
Matching is **case-insensitive** and **whitespace-stripped**, and common alternate
spellings are included as separate rows (e.g. `MCF7` / `MCF-7`, `HT29` / `HT-29`,
`NCI-H1299` / `H1299`).

## Step 2 — fallback CT
If a `Cell line` string has no entry in the table, the record is assigned
`FALLBACK_CT = "Breast"` — the most frequent CT in the nanotumor delivery database.

## Case normalisation
The source Table S3 contains a few CT labels that differ only in case
(e.g. `Breast`/`breast`, `Prostate`/`prostate`, `Skin`/`skin`, `Brain`/`brain`).
These were learned by the encoder as distinct classes in the original run; the
case-normalised re-analysis (Supplementary Note S2 / `ct_casenorm_DEmax.csv`, produced
by notebook `02`) consolidates them and confirms the results are unchanged.

## Material-class fallback (unknown CT)
If a mapped CT is not present in Model B's fitted CT encoder (encoded as `CT = -1`),
`DEmax_pred` for that record is taken from the **material-class mean** DEmax
(`MaterialClass → DEmax_mean`, i.e. the `DEmax_by_material_class.csv` table that
notebook `02` writes) instead of the Model B prediction.

## Excluded lineages (Table S4)
To keep the transcriptomics cell contexts within the solid-tumour training domain of the
delivery model, the following non-solid-tumour lineages are **excluded** before transfer
(810 records total):

| CellLine | Lineage | Cell type | Records |
|---|---|---|---|
| JURKAT-T | Blood / Immune | T-lymphocyte | 369 |
| HMDM | Blood / Immune | Macrophage | 360 |
| THP-1 | Blood / Immune | Monocyte | 54 |
| VSMC | Vascular | Smooth muscle | 27 |

These exclusions are why the framework's scope is delivery-informed screening within
characterised solid-tumour cell contexts and is not extended to endothelial, immune or
phagocytic lineages.
