---
layout: home
title: Home
nav_order: 1
---

# Welcome to the Superstars Team Pipeline

This site documents our bioinformatics pipeline for the analysis of cathepsin protein sequences from *Mnemiopsis leidyi*, a comb jelly native to the western Atlantic Ocean.

---

## What We Do

Our pipeline takes raw protein sequences in FASTA format and walks through every step of analysis — from computing fundamental physicochemical properties, to measuring pairwise sequence similarity, to visualizing evolutionary and structural relationships through clustering and heatmaps.

All analyses are implemented in **Python** and run on **Google Colab**, making the workflow accessible and reproducible for the whole team.

---

## Quick Links

- [Overview](overview.md) — Background and goals of the project
- [Pipeline](pipeline.md) — Step-by-step walkthrough of the notebook
- [Team](team.md) — Meet the Superstars

---

## Getting Started

1. Open the pipeline notebook in Google Colab
2. Upload your `.fasta` file when prompted
3. Run each cell in order from top to bottom
4. Outputs include property tables, similarity scores, a dendrogram, and relationship heatmaps

> **Tip:** Cells must be run in sequence — each stage depends on variables set in the previous step.
