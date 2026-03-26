# Spatial Transcriptomics Analysis of Prostate Cancer

This repository contains Jupyter notebooks for analyzing spatial and single-cell transcriptomics data from prostate cancer samples, examining malignancy progression, cell type dynamics, and treatment response in the tumor microenvironment.

## Data

Two complementary modalities are used:

- **10X Xenium Prostate Cancer Progression** — in situ spatial gene expression profiling across 9 prostate/lung metastasis tissue samples, totaling ~2.09 million cells
- **10X Xenium Post-Treatment** — in situ spatial gene expression profiling across 12 prostate tissue samples, totaling ~2.1 million cells
- **ZmanSeq** — single-cell RNA-seq of tumor-infiltrating immune cells at different time points (12h, 24h, 36h)

Conditions: wild-type control, 6-week post-transplant, 12-week post-transplant, lung normal/metastasis, and treatment arms (JAK inhibitor, anti-PD1, anti-GP120, combination).

## Notebooks

### [`260324_xenium_preprocess.ipynb`](260324_xenium_preprocess.ipynb)

**Xenium data preprocessing and integration**

Loads raw Xenium outputs from 9 samples (`.h5` cell-feature matrices + spatial metadata), converts to SpatialData Zarr format, then concatenates into a single AnnData object. Applies quality control filters (min 30 counts, 20 genes per cell; min 5 cells per gene), CPM normalization + log1p transformation, highly variable gene selection (2,000 HVGs, batch-aware), PCA, UMAP, and Leiden clustering. Outputs the combined `xenium_prostate.h5ad` used by all downstream figure notebooks.

---

### [`260324_figure2.ipynb`](260324_figure2.ipynb)

**Figure 2 — Cell type composition and basal cell malignancy dynamics**

Starting from `xenium_prostate.h5ad`, this notebook:
- Visualizes UMAP and spatial distributions colored by condition and cell type
- Computes and plots cell type proportion shifts across conditions (wild type → 6-week → 12-week → lung metastasis)
- Focuses on basal cells (malignant vs. non-malignant): aggregates pseudo-bulk counts per sample, runs linear regression of TPM across timepoints, and identifies differentially expressed genes
- Generates a z-score heatmap of top trending genes and bar plots of GO term enrichment results (via gseapy)

Produces panels for Figure 2b–j.

---

### [`260324_figure3.ipynb`](260324_figure3.ipynb)

**Figure 3 — Fibroblast and immune cell subtype characterization**

Loads the main `xenium_prostate.h5ad` alongside pre-clustered fibroblast (`xenium_fibroblasts_clustering.h5ad`) and immune (`xenium_immune_clustering.h5ad`) subsets. Updates UMAP coordinates from the pre-computed clusterings and generates high-resolution UMAP and spatial scatter plots with detailed subtype color palettes. Characterizes immune and fibroblast subtypes by marker gene expression.

Produces panels for Figure 3a–b.

---

### [`260324_figure4.ipynb`](260324_figure4.ipynb)

**Figure 4 — Immune cell dynamics (ZmanSeq single-cell data)**

Analyzes `Zmanseq_immune_submit.h5ad` (scRNA-seq of immune cells across treatment timepoints). Extracts timepoint labels from sample names, normalizes expression, and performs GO/pathway enrichment analysis using gseapy to identify functional categories changing over the course of treatment. Visualizes enrichment scores and significance.

Produces panels for Figure 4b–c.

---

### [`260324_figure5.ipynb`](260324_figure5.ipynb)

**Figure 5 — Treatment response across therapeutic conditions**

Loads `xenium_treatment.h5ad` (~2.1M cells) covering four treatment arms: JAK inhibitor, anti-PD1, anti-GP120, and combination therapy. Compares cell type composition and gene expression changes across conditions to characterize transcriptional responses to each treatment.

Produces panels for Figure 5d–e.

---

## Dependencies

```
scanpy
anndata
spatialdata
spatialdata-io
squidpy
gseapy
pandas
numpy
matplotlib
seaborn
scipy
statsmodels
```

## Usage

Run notebooks sequentially starting with the preprocessing notebook, then any figure notebook:

```bash
jupyter notebook
```

Each figure notebook expects preprocessed `.h5ad` files in the data directory. The preprocessing notebook must be run first to generate `xenium_prostate.h5ad`.
