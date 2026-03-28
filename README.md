# Multimodal Spatial Transcriptomics Analysis: Integrating Gene Expression, Imaging, and Graph Learning for Cell-Type Identification

This repository contains a complete pipeline for analyzing **spatial transcriptomics data** using **gene expression**, **morphological imaging features**, and **graph-based deep learning**. The workflow integrates multimodal data at the single-cell level to identify biologically meaningful cell populations and spatial tissue structure.

---

## About

This project provides a reproducible framework for **multimodal single-cell analysis** by combining:

* Targeted spatial transcriptomics (gene expression)
* CNN-extracted morphological features from microscopy images
* Spatial graph-based representation learning

**Key goals:**

* Perform robust quality control tailored for spatial transcriptomics
* Integrate imaging and transcriptomic data at the single-cell level
* Learn spatially informed embeddings using graph neural networks
* Identify cell populations and annotate biological cell types
* Interpret clusters using marker genes and pathway enrichment

---

## Repository Structure

```
project/
│── README.md
│── requirements.txt
│── .gitignore
│
│── notebooks/
│ ├── 1_Data_loading_preprocessing_hvg.ipynb
│ ├── 2_CNN_feature_extraction.ipynb
│ ├── 3_Graph_model_construction.ipynb
│ └── 4_Clustering_annotation_and_pathway_analysis.ipynb
│
│── models/
│ └── CNN_embeddings.npy
│
└── dataset/
│ ├── raw/
│ ├── HumanOvarianCancerPatient2Slice2_cell_by_gene.csv
│ ├── HumanOvarianCancerPatient2Slice2_cell_metadata.csv
│ ├── processed/
│ ├── adata_500.h5ad
│ ├── adata_hvg_with_CNN_embeddings.h5ad
│ └── adata_with_graph_embeddings.h5ad
│ └── images/
```

**Notes:**

1. `notebooks/` contains step-by-step analysis in logical order.
2. `dataset/` includes raw data, processed AnnData files, and imaging data.
3. `models/` contains pretrained CNN weights.
4. `scripts/` includes reusable modules for image processing, modeling, and utilities.
5. `requirements.txt` ensures reproducibility.

---

## Data Loading and Quality Control

Gene expression and metadata were loaded from CSV files and stored in an **AnnData** object.

* Initial dataset:

  * **71,381 cells**
  * **550 genes**

### Quality Checks Performed:

* Verified gene symbols (e.g., CD4, TBX21, GZMB)
* Confirmed absence of:

  * Mitochondrial genes
  * Ribosomal genes
  * Hemoglobin genes

### QC Metrics:

* Mean genes per cell: ~104
* Mean counts per cell: ~317

### Filtering Strategy:

* Removed cells with < 40 genes
* Removed genes expressed in < 20 cells
* MAD-based outlier removal (threshold = 6)

### After QC:

* **63,072 cells retained**
* **550 genes**

---

## Removal of Control Probes

* Identified **50 blank/control probes**
* Removed from dataset

### Final Dataset:

* **63,072 cells**
* **500 genes**

---

## Doublet Detection

Performed using **Scrublet**:

* Expected doublet rate: 4%
* Detected doublets: **0%**

This is consistent with spatial transcriptomics data, where doublets are rare.

---

## Normalization and Feature Selection

* Total-count normalization (`target_sum = 1e4`)
* Log transformation (`log1p`)
* Highly Variable Gene (HVG) selection (Seurat v3)

### Selected:

* **150 HVGs**

---

## Dimensionality Reduction and Clustering

* PCA (ARPACK solver)
* KNN graph:

  * 30 PCs
  * 15 neighbors
* UMAP visualization
* Leiden clustering (resolution = 0.5)

---

## Integration of Imaging Data (CNN Features)

### Image Processing:

* Extracted **64×64 patches** per cell
* Used **7 Z-stack images**
* Averaged into a single representation

### CNN Model:

* **ResNet-18 (modified for grayscale input)**
* Final output: **512-dimensional embedding per cell**

### Output:

* Shape: **(63,072, 512)**
* Stored in: `obsm['X_cnn']`

---

## Multimodal Feature Integration

* Gene expression (150 HVGs)
* CNN features (512 dims)

### Combined Feature Matrix:

* Shape: **(63,072, 662)**

---

## Spatial Graph Construction

* Built using cell coordinates
* k = 10 nearest neighbors

### Graph:

* **63,072 nodes**
* **567,648 edges**

---

## Graph Transformer Model

Implemented using **PyTorch Geometric**:

### Architecture:

* 2 × TransformerConv layers (multi-head attention)
* ReLU activations
* Linear projection

### Output:

* **128-dimensional embeddings**

---

## Model Training

* Unsupervised reconstruction task
* Loss: Mean Squared Error (MSE)
* Optimizer: Adam (lr = 0.001)

### Training:

* 200 epochs
* Final loss: **0.0561**

---

## Embedding Storage

* Final embeddings stored in:

  * `obsm["X_graph"]`
* Shape: **(63,072, 128)**

---

## Clustering Using Graph Embeddings

* KNN graph built using `X_graph`
* Leiden clustering (resolution = 0.5)
* UMAP visualization of learned structure

---

## Differential Gene Expression

* Wilcoxon rank-sum test
* Compared clusters vs all cells
* Included:

  * Top marker genes
  * Expression percentages (`pts=True`)

---

## Marker Gene Filtering

Criteria:

* Adjusted p-value < 0.05

* Log fold change > 0.25

* Expression > 5%

* Top **100 genes per cluster** retained

---

## Pathway Enrichment Analysis

Performed using **GSEApy (Enrichr)**:

### Databases:

* GO Biological Processes (2021)
* KEGG Pathways (2021)

### Selection Criteria:

* Adjusted p-value
* Combined enrichment score

---

## Automated Cell Type Annotation

Clusters were annotated using:

* Marker genes
* Enriched pathways

### Identified Cell Types:

* Epithelial tumor cells
* Proliferating tumor cells
* Metabolic tumor cells
* Cancer-associated fibroblasts (CAFs)
* Endothelial cells
* T/NK cells
* Myeloid cells
* Inflamed microenvironment

---

## Results Summary

* **20 clusters identified**
* Clear separation of:

  * Tumor populations (heterogeneous)
  * Stromal cells (CAFs)
  * Immune cells (T/NK, myeloid)
  * Endothelial structures

---

## Final Outputs

The final AnnData file includes:

* Filtered gene expression
* Spatial metadata
* CNN embeddings (`X_cnn`)
* Graph embeddings (`X_graph`)
* Cluster labels
* Cell-type annotations

### Saved as:

```
adata_with_graph_embeddings.h5ad
```

---

## Visualizations

* UMAP (clusters & cell types)
* Spatial plots (tissue organization)
* PCA variance plots
* QC plots (violin, histogram, scatter)

---
