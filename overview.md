---
layout: default
title: Overview
nav_order: 2
---

# Example: *Mnemiopsis leidyi*
{: .no_toc }

<details open markdown="block">
  <summary>Table of contents</summary>
  {: .text-delta }
- TOC
{:toc}
</details>

---

## Background

*Mnemiopsis leidyi* is a gelatinous ctenophore (comb jelly) that has attracted significant interest in evolutionary and genomic research. As an early-branching animal lineage, it offers a rare window into the origins of complex nervous and digestive systems. Its sequenced genome provides a rich resource for identifying and characterizing gene families — including proteases such as cathepsins.

**Cathepsins** are a family of lysosomal proteases responsible for protein degradation and turnover inside cells. They play important roles in digestion, immune function, and programmed cell death. Characterizing cathepsin diversity in *Mnemiopsis* can shed light on how these enzymes evolved across the animal tree of life.

---

## Project Goals

This project aims to:

1. **Characterize** the physicochemical properties of candidate cathepsin sequences from the *Mnemiopsis* genome
2. **Assess** sequence similarity across the cathepsin gene family using pairwise global alignment
3. **Cluster** proteins based on both sequence and physicochemical features to identify potential subfamilies
4. **Visualize** evolutionary relationships through dendrograms and distance heatmaps

---

## Dataset

The example dataset (`Mnemiopsis_cathepsin_subset.fasta`) contains **6 protein sequences** identified from the *Mnemiopsis leidyi* genome assembly:

| Protein ID | Length (aa) | Molecular Weight (Da) | pI   | Net Charge (pH 7) |
|---|---|---|---|---|
| g-680      | 462         | 52,501.82             | 5.56 | -7.98             |
| g-11155    | 335         | 37,421.28             | 8.14 | +2.28             |
| g-15657    | 340         | 38,453.17             | 6.76 | -0.74             |
| g-4430     | 333         | 36,689.52             | 7.17 | +0.45             |
| g-15835    | 2,652       | 286,578.77            | 4.05 | -703.49           |
| g-15451    | 335         | 37,663.53             | 8.53 | +4.16             |

> **Note:** g-15835 is notably larger than the other sequences and has an extreme net charge, suggesting it may represent a fusion protein or a structurally distinct isoform.

---

## Tools & Technologies

| Tool | Use |
|---|---|
| **Python 3.12** | Core programming language |
| **Biopython** | Sequence parsing, protein analysis, pairwise alignment |
| **Pandas** | Data organization and display |
| **SciPy** | Hierarchical clustering, pairwise distance calculation |
| **scikit-learn** | Feature scaling |
| **Matplotlib / Seaborn** | Visualization |
| **Google Colab** | Cloud-based notebook environment |

---

## Workflow Summary

```
FASTA Input
    │
    ▼
Physicochemical Property Calculation
    │
    ▼
Pairwise Sequence Alignment (Global)
    │
    ├──► Dendrogram (Hierarchical Clustering)
    │
    └──► Distance Heatmaps (Physicochemical + Sequence + Combined)
```
