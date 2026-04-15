# Mnemiopsis Cathepsin Bioinformatics Pipeline

A Google Colab-based pipeline for analyzing protein sequences from *Mnemiopsis leidyi* (comb jelly). Given one or more FASTA files, the notebook computes physicochemical properties, performs pairwise sequence alignment, and visualizes protein relationships via dendrogram and heatmaps.

---

## Table of Contents

- [Example: M.leidyi](#overview)
- [Requirements](#requirements)
- [Input Data](#input-data)
- [Pipeline Stages](#pipeline-stages)
  - [1. Install & Import Packages](#1-install--import-packages)
  - [2. Input or Fetch Data](#2-input-or-fetch-data)
  - [3. Store FASTA File](#3-store-fasta-file)
  - [4. Calculate Physicochemical Properties](#4-calculate-physicochemical-properties)
  - [5. Create a Property Dictionary](#5-create-a-property-dictionary)
  - [6. Sequence Similarity](#6-sequence-similarity)
  - [7. Store Sequence Similarity](#7-store-sequence-similarity)
  - [8. Relationship Between Proteins — Dendrogram](#8-relationship-between-proteins--dendrogram)
  - [9. Relationship Outputs — Heatmaps](#9-relationship-outputs--heatmaps)
- [Output Summary](#output-summary)
- [Notes & Limitations](#notes--limitations)

---

## Example: M.leidyi

This pipeline takes protein sequences in FASTA format and produces:

- Per-protein physicochemical properties (molecular weight, isoelectric point, net charge, extinction coefficient, amino acid composition)
- An all-vs-all pairwise sequence similarity table using global alignment
- A hierarchical clustering dendrogram based on physicochemical features
- Three pairwise distance heatmaps: physicochemical, sequence-based, and combined

The example dataset contains six cathepsin-like proteins from *Mnemiopsis leidyi* (gene IDs: g-680, g-11155, g-15657, g-4430, g-15835, g-15451).

---

## Requirements

All dependencies are installed within the notebook. The pipeline runs on **Google Colab** (Python 3.12+).

| Package | Purpose |
|---|---|
| `biopython` (≥1.87) | FASTA parsing, protein analysis, pairwise alignment |
| `pandas` | Tabular data storage and display |
| `numpy` | Numerical operations |
| `matplotlib` | Dendrogram and plot rendering |
| `seaborn` | Heatmap visualization |
| `scipy` | Hierarchical clustering, pairwise distance calculation |
| `scikit-learn` | Feature scaling (StandardScaler, MinMaxScaler) |
| `google.colab` | File upload interface |

Install command (run automatically in the notebook):
```bash
!pip install biopython
```

---

## Input Data

**Format:** FASTA (`.fasta` or `.fa`), containing one or more protein sequences.

**Example file:** `Mnemiopsis_cathepsin_subset.fasta` (~4,580 bytes, 6 sequences)

Each record should follow standard FASTA format:
```
>sequence_id optional description
MKVLILCLVVVTITVS...
```

Stop codons (`*`) are stripped automatically before analysis.

---

## Pipeline Stages

### 1. Install & Import Packages

Installs `biopython` and imports the core libraries needed throughout the notebook:

```python
import google.colab.files as files
from pathlib import Path
```

### 2. Input or Fetch Data

Prompts the user to upload one or more FASTA files via the Colab file dialog:

```python
uploaded = files.upload()
```

All uploaded filenames are stored in the `uploaded` dictionary for downstream use.

### 3. Store FASTA File

Reads each uploaded file into memory and stores the raw text content in `fasta_data_dict`, a dictionary keyed by filename:

```python
fasta_data_dict[file_path] = f.read()
```

This step confirms the number of characters loaded per file and is required before any analysis steps.

### 4. Calculate Physicochemical Properties

Uses `Bio.SeqUtils.ProtParam.ProteinAnalysis` to compute the following for each sequence:

| Property | Description |
|---|---|
| **Length** | Number of amino acids |
| **Molecular Weight** | In Daltons (Da) |
| **Amino Acid Composition** | Per-residue counts for all 20 standard amino acids |
| **Isoelectric Point (pI)** | pH at which the protein carries no net charge |
| **Net Charge at pH 7.0** | Estimated charge under physiological conditions |
| **Extinction Coefficient** | Molar extinction (M⁻¹ cm⁻¹) for both reduced and oxidized cysteine states |

Results are printed to the console for each sequence.

### 5. Create a Property Dictionary

Aggregates the per-protein properties into a `pandas` DataFrame (`properties_df`) with one row per protein:

| Column | Description |
|---|---|
| `Protein_ID` | Sequence identifier from FASTA header |
| `Length` | Amino acid count |
| `Molecular_Weight_Da` | Molecular weight (Da) |
| `Isoelectric_Point` | pI value |
| `Net_Charge_pH7` | Net charge at pH 7.0 |
| `Extinction_Coeff_Reduced` | Extinction coefficient (reduced cysteines) |

The DataFrame is displayed interactively and carried forward into clustering and heatmap steps.

### 6. Sequence Similarity

Performs all-vs-all **global pairwise alignment** using `Bio.Align.PairwiseAligner` (mode: `global`):

```python
aligner = Align.PairwiseAligner()
aligner.mode = 'global'
```

For each unique protein pair, the pipeline reports:

- **Raw Score** — Absolute alignment score (positive = similar, negative = divergent)
- **Similarity Percentage** — Raw score normalized by the length of the longer sequence (`score / max_len * 100`)

> **Note:** Because this metric is not a true percent identity, it can be negative when mismatches and gaps dominate the alignment.

### 7. Store Sequence Similarity

Collects all pairwise alignment results into a `pandas` DataFrame (`similarity_df`):

| Column | Description |
|---|---|
| `Protein_1` | First protein in the pair |
| `Protein_2` | Second protein in the pair |
| `Raw_Score` | Biopython PairwiseAligner score |
| `Similarity_Percentage` | Normalized similarity metric |

The notebook also prints a plain-language explanation of how to interpret positive vs. negative scores.

### 8. Relationship Between Proteins — Dendrogram

Generates a **hierarchical clustering dendrogram** based on standardized physicochemical features using Ward's linkage method:

```python
from scipy.cluster.hierarchy import dendrogram, linkage
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()
scaled_features = scaler.fit_transform(clustering_data)
Z = linkage(scaled_features, method='ward')
```

Features used for clustering: `Length`, `Molecular_Weight_Da`, `Isoelectric_Point`, `Net_Charge_pH7`, `Extinction_Coeff_Reduced`.

Standardization ensures that large-scale properties (e.g., molecular weight) do not dominate over small-scale ones (e.g., net charge). Proteins that merge at lower y-axis values are more physicochemically similar.

### 9. Relationship Outputs — Heatmaps

Generates three side-by-side normalized distance heatmaps (0 = identical, 1 = maximally dissimilar):

1. **Physicochemical Distances** — Euclidean distances in standardized feature space
2. **Sequence Similarity Distances** — Derived from pairwise alignment scores
3. **Combined Distances** — Average of the two above matrices

```python
combined_dist_norm = (phys_dist_norm + seq_dist_norm) / 2.0
```

All matrices are normalized to [0, 1] using `MinMaxScaler` before display.

---

## Output Summary

| Output | Variable | Type |
|---|---|---|
| Physicochemical properties per protein | `properties_df` | `pd.DataFrame` |
| All-vs-all alignment scores | `similarity_df` | `pd.DataFrame` |
| Hierarchical clustering dendrogram | — | `matplotlib` figure |
| Distance heatmaps (×3) | — | `matplotlib/seaborn` figure |

---

## Notes & Limitations

- **Cell execution order matters.** Each stage depends on variables set in the previous step. Run cells top to bottom.
- **Minimum sequence count:** At least 2 sequences are required for similarity and clustering analyses.
- **Alignment scoring:** The similarity percentage used here is a custom normalized metric and is not equivalent to BLAST percent identity. Highly divergent sequences will produce negative values.
- **Global alignment only:** The pipeline uses global (Needleman-Wunsch-style) alignment, which is best suited for sequences of similar length. For distantly related or length-mismatched sequences, consider local alignment (`mode='local'`).
- **Platform:** Designed for Google Colab. The `files.upload()` step will not work in a standard local Jupyter environment without modification.
