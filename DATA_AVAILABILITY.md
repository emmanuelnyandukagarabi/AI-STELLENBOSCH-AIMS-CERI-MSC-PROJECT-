# Data Availability

This project classifies Dengue virus (DENV) genomes into lineages. The raw genomic
sequences used for training **are not redistributed in this repository**.

## Why the sequences are not shared here

The genomes were obtained from **GISAID** (https://gisaid.org/). Under the GISAID
Database Access Agreement (EpiArbo / EpiFlu terms), users may not pass on the sequence
data to third parties or repost it publicly. To respect these terms, this repository
shares only the **code**, the **model architectures**, and the **downstream results**,
not the underlying GISAID sequences.

## How to rebuild the dataset

You can reproduce the working dataset by following the same steps used in the thesis:

1. **Download genomes from GISAID.** Select *complete*, *high-coverage* DENV genomes
   with a confirmed collection date (the thesis used 7,689 records retrieved on
   29 August 2024). Export the sequences (FASTA) and their metadata (collection date,
   submission date, location).

2. **Assign lineages.** Type each sequence with the
   [Genome Detective Dengue Virus Typing Tool](https://www.genomedetective.com/app/typingtool/dengue/)
   and cross-reference the labels against the
   [Dengue Lineages Initiative](https://dengue-lineages.org/) (Hill et al., 2024).

3. **Clean.** Drop records labelled *unassigned*, *related to*, or *similar to*, and
   remove sequences containing ambiguous bases (N, R, Y, M, K, S, W, B, D, V, ...).
   The thesis retained **3,527 sequences across 38 minor lineages** after this step.

4. **Build the feature tables.** Run `Code/October_24_2024_Part1_Data_collection_*.ipynb`,
   which produces the k-mer, FCGR, and one-hot representations used by the models.

## What *is* included in this repository

- `Code/` : the six notebooks for the full pipeline (data prep, hyper-parameter search,
  ML classifiers, DL classifiers, VAE, early-variant detection).
- `MODELS_ARCHITECTURES/` : diagrams of the 1D-CNN + self-attention, 2D-CNN-FCGR, and VAE.
- `DENGUE_DETECTIVE_RESULTS/`, `NEXTCLADE_RESULTS/`, `NCBI_BLASTN_RESULTS/`,
  `NCBI_SEQUENCES_FOR_REFERENCES/` : validation outputs for the generated (synthetic)
  sequences. The NCBI reference genomes are public (accessions NC_001477.1 / DENV1,
  NC_001474.2 / DENV2, NC_001475.2 / DENV3, NC_002640.1 / DENV4).
- `Msc_Thesis/` : the full thesis PDF.

## Acknowledgement

We gratefully acknowledge the authors, originating and submitting laboratories of the
Dengue genomic sequences obtained from GISAID that made this work possible.
