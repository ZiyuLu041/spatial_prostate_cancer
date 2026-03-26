# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a spatial transcriptomics research project analyzing prostate cancer malignancy and treatment response using two complementary data modalities:
- **10X Xenium**: High-resolution in situ spatial gene expression (~2.1M cells across 9 tissue samples)
- **ZmanSeq**: Single-cell transcriptomics with treatment timepoints (CD8+ T cells, neutrophils, macrophages, etc.)

The notebooks in this `submit/` directory generate publication figures for manuscript submission.

## Running Notebooks

There is no build system. Notebooks are run interactively via Jupyter:

```bash
jupyter notebook
# or
jupyter lab
```

Each notebook is self-contained. Run cells sequentially from top to bottom.

## Data Paths

- Raw/processed data: `/ru-auth/local/home/zlu/aws/spatial/data/` (H5AD files, ~14 GB)
- Results (DEGs, enrichment): `./results/`
- Figures output: `./figures/` and `./plots/`

Xenium raw outputs follow this structure per sample:
```
<sample_dir>/
  cell_feature_matrix/   # matrix.mtx.gz, barcodes.tsv.gz, features.tsv.gz
  cells.parquet          # cell metadata with x_centroid, y_centroid
  cells.zarr.zip         # spatial data
  transcripts.zarr.zip
  analysis/umap/gene_expression_2_components/projection.csv
  analysis/clustering/
```

## Notebook Contents

| Notebook | Content |
|----------|---------|
| `260324_xenium_preprocess.ipynb` | Raw Xenium loading, QC (min_counts=30, min_genes=20, min_cells=5), normalization, PCA, Leiden clustering |
| `260324_figure2.ipynb` | Cell type proportions by condition, spatial visualization, basal cell malignancy signatures |
| `260324_figure3.ipynb` | Additional cell type characterization (immune/fibroblast) |
| `260324_figure4.ipynb` | ZmanSeq immune cell analysis (CD8+ T cells, GO term enrichment via gseapy) |
| `260324_figure5.ipynb` | Multi-sample spatial organization, cell-cell interaction analysis |

## Key Libraries

- `scanpy` / `anndata` — core single-cell analysis framework; data stored as `adata` (AnnData objects)
- `spatialdata` / `spatialdata_io` — spatial data I/O
- `squidpy` — spatial statistics and neighborhood analysis
- `gseapy` — GO term enrichment analysis
- `scipy`, `statsmodels` — statistical tests, FDR correction (Benjamini-Hochberg)

## Analysis Patterns

**Standard preprocessing pipeline:**
```python
sc.pp.filter_cells(adata, min_counts=30, min_genes=20)
sc.pp.filter_genes(adata, min_cells=5)
sc.pp.normalize_total(adata)
sc.pp.log1p(adata)
sc.pp.pca(adata)
sc.pp.neighbors(adata)
sc.tl.leiden(adata)
sc.tl.umap(adata)
```

**Multi-sample concatenation:**
```python
combined = ad.concat([adata1, adata2, ...], label="sample", keys=sample_names)
```

**Spatial scatter plot convention:** Always `invert_yaxis()` to match tissue orientation.

## Biological Context

- **Samples**: 9 Xenium samples (IDs: H2024-1069-18/19/29A/29B/40, H2024-959-2/3, H2024-1100-1/2)
- **Conditions**: Wild-type control, 6-week post-transplant, 12-week post-transplant, lung (normal/metastasis)
- **Key cell types**: Basal cells (malignant vs. non-malignant), luminal cells, immune cells (CD4, CD8, macrophages, neutrophils), fibroblasts
- **Malignancy scoring**: Based on gene expression signatures to distinguish malignant progression
