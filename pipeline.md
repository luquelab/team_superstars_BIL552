---
layout: default
title: Pipeline
nav_order: 3
---

# Pipeline
{: .no_toc }

A step-by-step walkthrough of the *Mnemiopsis* cathepsin [analysis notebook](https://github.com/luquelab/team_superstars_BIL552/blob/main/docs/Mnemiopsis_cathepsin_subset.fasta). Run all cells **in order** — each stage depends on variables set in the previous step.

<details open markdown="block">
  <summary>Table of contents</summary>
  {: .text-delta }
- TOC
{:toc}
</details>

---

## Stage 1 — Install & Import Packages

Installs `biopython` and imports all libraries needed for the notebook.

```python
!pip install biopython

import google.colab.files as files
from pathlib import Path
```

**Dependencies installed:** `biopython`, `numpy` (pulled in automatically)

---

## Stage 2 — Input or Fetch Data

Prompts you to upload one or more FASTA files using the Colab file dialog.

```python
uploaded = files.upload()

for filename in uploaded.keys():
    print(f'Uploaded file: {filename}')
```

**Input format:** Standard FASTA (`.fasta` or `.fa`) containing protein sequences.
Stop codons (`*`) are stripped automatically in later stages.

**Example upload:** `Mnemiopsis_cathepsin_subset.fasta` — 4,580 bytes, 6 sequences

---

## Stage 3 — Store FASTA File

Reads each uploaded file into memory and stores the raw content in a dictionary for downstream use.

```python
fasta_data_dict = {}
for file_path in fasta_files:
    with open(file_path, 'r') as f:
        fasta_data_dict[file_path] = f.read()
```

**Output variable:** `fasta_data_dict` — keys are filenames, values are raw FASTA strings.

> If this step is skipped, all subsequent cells will fail. Run it before anything else.

---

## Stage 4 — Calculate Physicochemical Properties

Uses `Bio.SeqUtils.ProtParam.ProteinAnalysis` to compute six properties per sequence.

```python
from Bio import SeqIO
from Bio.SeqUtils.ProtParam import ProteinAnalysis
import io

analyzed_seq = ProteinAnalysis(sequence)
```

| Property | Method | Output |
|---|---|---|
| Length | `len(sequence)` | Integer (amino acids) |
| Molecular Weight | `.molecular_weight()` | Float (Da) |
| Amino Acid Composition | `.count_amino_acids()` | Dict of 20 AA counts |
| Isoelectric Point | `.isoelectric_point()` | Float (pH units) |
| Net Charge at pH 7.0 | `.charge_at_pH(7.0)` | Float |
| Extinction Coefficient | `.molar_extinction_coefficient()` | Tuple (reduced, oxidized) in M⁻¹ cm⁻¹ |

Results are printed to the console for each sequence in the file.

---

## Stage 5 — Create a Property Dictionary

Aggregates all per-protein properties into a single `pandas` DataFrame.

```python
import pandas as pd

prop_dict = {
    'Protein_ID': record.id,
    'Length': len(sequence),
    'Molecular_Weight_Da': round(analyzed_seq.molecular_weight(), 2),
    'Isoelectric_Point': round(analyzed_seq.isoelectric_point(), 2),
    'Net_Charge_pH7': round(analyzed_seq.charge_at_pH(7.0), 2),
    'Extinction_Coeff_Reduced': analyzed_seq.molar_extinction_coefficient()[0]
}
properties_df = pd.DataFrame(protein_properties)
```

**Output variable:** `properties_df` — used in clustering and heatmap steps.

**Example output:**

| Protein_ID | Length | Molecular_Weight_Da | Isoelectric_Point | Net_Charge_pH7 | Extinction_Coeff_Reduced |
|---|---|---|---|---|---|
| g-680   | 462  | 52501.82  | 5.56 | -7.98   | 131670 |
| g-11155 | 335  | 37421.28  | 8.14 | 2.28    | 67840  |
| g-15657 | 340  | 38453.17  | 6.76 | -0.74   | 95800  |
| g-4430  | 333  | 36689.52  | 7.17 | 0.45    | 64860  |
| g-15835 | 2652 | 286578.77 | 4.05 | -703.49 | 48360  |
| g-15451 | 335  | 37663.53  | 8.53 | 4.16    | 80330  |

---

## Stage 6 — Sequence Similarity

Performs all-vs-all **global pairwise alignment** using `Bio.Align.PairwiseAligner`.

```python
from Bio import Align
import itertools

aligner = Align.PairwiseAligner()
aligner.mode = 'global'

pairs = list(itertools.combinations(sequences.keys(), 2))
for id1, id2 in pairs:
    alignments = aligner.align(sequences[id1], sequences[id2])
    best_alignment = alignments[0]
    similarity_score = (best_alignment.score / max(len(seq1), len(seq2))) * 100
```

**Score interpretation:**

- **Positive score** → more matches than mismatches/gaps; sequences are related
- **Negative score** → penalty for mismatches/gaps outweighs matches; sequences are highly divergent

> The similarity percentage here is a custom normalized metric (not standard BLAST percent identity) and can be negative for divergent pairs.

---

## Stage 7 — Store Sequence Similarity

Stores all pairwise alignment results in a `pandas` DataFrame.

**Output variable:** `similarity_df`

| Column | Description |
|---|---|
| `Protein_1` | First protein in the pair |
| `Protein_2` | Second protein in the pair |
| `Raw_Score` | Biopython alignment score |
| `Similarity_Percentage` | Score normalized by length of longer sequence |

---

## Stage 8 — Dendrogram

Clusters proteins by physicochemical similarity using **Ward's hierarchical linkage** on standardized features.

```python
from scipy.cluster.hierarchy import dendrogram, linkage
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()
scaled_features = scaler.fit_transform(clustering_data)
Z = linkage(scaled_features, method='ward')
dendrogram(Z, labels=clustering_data.index.tolist())
```

Features used: `Length`, `Molecular_Weight_Da`, `Isoelectric_Point`, `Net_Charge_pH7`, `Extinction_Coeff_Reduced`

Standardization ensures large-scale properties (molecular weight) don't dominate small-scale ones (net charge). Proteins merging at **lower y-axis values** are more similar.

---

## Stage 9 — Relationship Heatmaps

Generates three normalized distance matrices (0 = identical, 1 = maximally dissimilar) and displays them as side-by-side heatmaps.

```python
import seaborn as sns
from scipy.spatial.distance import pdist, squareform
from sklearn.preprocessing import MinMaxScaler

combined_dist_norm = (phys_dist_norm + seq_dist_norm) / 2.0
```

| Heatmap | Source |
|---|---|
| 1. Physicochemical Distances | Euclidean distance in standardized feature space |
| 2. Sequence Similarity Distances | Derived from global alignment scores |
| 3. Combined Distances | Average of the two above matrices |

Lighter cells = more similar. Darker cells = more dissimilar.
