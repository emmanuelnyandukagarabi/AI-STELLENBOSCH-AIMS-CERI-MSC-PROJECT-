# AI-Powered Classification and Early Detection of Dengue Virus Lineages

Machine learning and deep learning for **alignment-free classification of Dengue virus
(DENV) genomes into lineages**, together with a fast **early-detection scheme for
potential new variants**.

This is the code and material for the MSc research project *"AI-Powered Classification
and Early Detection of Dengue Virus Lineages for Timely Public Health Response"*, done at
**AIMS South Africa** (MSc in Artificial Intelligence for Science, funded by Google
DeepMind) in partnership with the **Centre for Epidemic Response and Innovation (CERI),
Stellenbosch University**.

- **Author:** Emmanuel Nyandu Kagarabi &lt;emmanuelnk@aims.ac.za&gt;
- **Submitted:** 24 October 2024 &nbsp;·&nbsp; **Defended:** 7 November 2024
- **Thesis:** [`Msc_Thesis/emmanuelnk_Msc_Thesis_Stellenbosch_University_CERI.pdf`](Msc_Thesis/emmanuelnk_Msc_Thesis_Stellenbosch_University_CERI.pdf)
- **Poster:** presented at the **Deep Learning Indaba 2025, Kigali (Rwanda)**

> ⚠️ The training sequences come from **GISAID** and are **not redistributed** here.
> See [`DATA_AVAILABILITY.md`](DATA_AVAILABILITY.md) for how to rebuild the dataset.

## Why this project

Dengue is one of the most widespread mosquito-borne viral diseases, with an estimated
100 to 400 million infections every year. The virus has four serotypes (DENV-1 to
DENV-4), and in 2024 Hill *et al.* introduced a finer, hierarchical naming system that
adds **major** and **minor lineages** below each genotype (for example `3III_B.3.2` reads
as serotype **3**, genotype **III**, major lineage **B**, minor lineage **B.3.2**).

The usual way to assign a lineage is to build a phylogenetic tree, which is accurate but
slow and computationally heavy. This project asks a simpler question: **can a model read a
raw genome and predict its lineage directly, without alignment, and can it flag sequences
that do not fit any known lineage?** At the time of writing, no published work had applied
ML or DL to Dengue at the lineage level.

## What the pipeline does

```
GISAID genomes ──► clean & label ──► feature extraction ──► classifiers ──► lineage
   (7,689)          (3,527 seqs,       k-mer / FCGR /        (ML + DL)
                     38 lineages)      one-hot

                          └──► VAE generates synthetic genomes
                                      │
                                      ▼
                          soft-voting ensemble + 0.8 threshold ──► "known lineage" or
                                                                    "Uncertain" (new variant?)
```

1. **Data and labels.** 7,689 complete, high-coverage genomes were pulled from GISAID,
   typed with the Genome Detective Dengue Virus Typing Tool, and cross-checked against
   the Dengue Lineages Initiative. After removing unassigned records and ambiguous bases,
   **3,527 sequences across 38 minor lineages** remained. The data is strongly imbalanced
   (DENV1 42.8%, DENV2 30.7%, DENV3 20.9%, DENV4 5.6%), so training used an 80/20
   stratified split followed by **SMOTE** on the training set only.

2. **Feature extraction (three views of a genome).**
   - **k-mer counts** — frequency of every length-*k* substring (best value **k = 5**, giving 1,024 features).
   - **FCGR** — Frequency Chaos Game Representation, which folds a genome into a **32×32 grayscale image**.
   - **One-hot** — each base as a 4-dim vector, padded to 10,821 × 4 for the neural networks.

3. **Classifiers.**
   - **Flat ML:** Random Forest, XGBoost, LightGBM (hyper-parameters tuned with Optuna).
   - **Hierarchical ML:** Local Classifier per Node / per Parent Node / per Level (via `hiclass`), predicting all four taxonomic levels.
   - **Deep learning:** a **1D-CNN with a self-attention layer** (one-hot input) and a **2D-CNN on FCGR images**.

4. **Early detection of new variants.** A **Variational Autoencoder (VAE)** generates
   synthetic genomes; those validated by Nextclade are passed to a **soft-voting ensemble**
   of three pretrained models (LightGBM-5mer, 1D-CNN-attention, 2D-CNN-FCGR). If the mean
   confidence stays below **0.8**, the sequence is flagged **"Uncertain"**, i.e. a
   candidate new variant for expert review.

## Headline results

- Every model reached **≥ 97% accuracy and/or F1-score**; the **2D-CNN on FCGR** reached
  essentially perfect scores on the held-out test set.
- The voting ensemble produced the **same lineage calls as the Genome Detective typing
  tool**, but classified 43 sequences in **under 10 seconds** versus roughly **40 minutes**
  for the online tool.

> **Note for readers and future work.** The near-perfect scores partly reflect how cleanly
> Dengue lineages separate by nucleotide composition, and partly a hyper-parameter search
> that was scored on the test set. A revised version of this work will use a proper
> validation split (or nested cross-validation) and report calibrated, per-lineage metrics.

## Repository structure

```
.
├── Code/                          # Full pipeline (6 Jupyter notebooks, run in order)
│   ├── ...Part1_Data_collection...ipynb    # download, clean, feature extraction (k-mer, FCGR, one-hot)
│   ├── ...Part2_Optimal_parameters...ipynb # best k and FCGR resolution + Optuna, t-SNE
│   ├── ...Part3_ML_Classifiers...ipynb     # flat + hierarchical ML classifiers
│   ├── ...Part4_DL_classifiers...ipynb     # 1D-CNN + self-attention, 2D-CNN-FCGR
│   ├── ...Part5_VAE...ipynb                # Variational Autoencoder (synthetic genomes)
│   └── ...Part6_Early_Detection...ipynb    # soft-voting ensemble for new-variant detection
├── 24_10_2024_fine_tuning_...optuna.ipynb  # full Optuna hyper-parameter search
├── MODELS_ARCHITECTURES/          # Netron diagrams of the CNNs and the VAE
├── DENGUE_DETECTIVE_RESULTS/      # Genome Detective labels + phylogenetic trees (synthetic seqs)
├── NEXTCLADE_RESULTS/             # Nextclade validation of generated sequences
├── NCBI_BLASTN_RESULTS/           # BLASTN dot-plots of generated vs reference genomes
├── NCBI_SEQUENCES_FOR_REFERENCES/ # public NCBI reference genomes (DENV1-4)
├── Msc_Thesis/                    # full thesis PDF
├── DATA_AVAILABILITY.md           # how to obtain / rebuild the GISAID dataset
├── CITATION.cff                   # how to cite this work
└── README.md
```

## How to reproduce

The notebooks were written for **Google Colab with a GPU**. To run them:

1. Rebuild the dataset by following [`DATA_AVAILABILITY.md`](DATA_AVAILABILITY.md).
2. Open the `Code/` notebooks **in order (Part 1 → Part 6)**.
3. Main libraries (versions used in the thesis): Python 3.10, Biopython 1.84,
   scikit-learn 1.4.2, TensorFlow-Keras 3.4.1, imbalanced-learn 0.12.3, Optuna 4.0.0,
   plus `xgboost`, `lightgbm`, `hiclass`, and `keras-self-attention`.

```bash
pip install biopython scikit-learn tensorflow imbalanced-learn optuna \
            xgboost lightgbm hiclass keras-self-attention mmh3 prince
```

## Limitations and next steps

- Only **38 of the 64** recognised minor lineages were covered; broader coverage and newer GISAID data are planned.
- The models are **composition-based and not yet explainable**; adding SHAP / attention maps and linking key k-mers to genome regions is on the roadmap.
- The VAE yields relatively few valid genomes; stronger generative models (GANs, diffusion) and a more principled definition of "novel" (out-of-distribution / calibrated uncertainty) are being explored.
- The notebooks will be refactored into a clean, seeded, reproducible Python package.

## How to cite

If you use this work, please cite the thesis (see [`CITATION.cff`](CITATION.cff)) and, once
available, the archived Zenodo record.

## Acknowledgements

Funded by **Google DeepMind** through the AIMS South Africa MSc in AI for Science, and
supervised in partnership with **CERI, Stellenbosch University**. We thank the authors and
the originating and submitting laboratories of the Dengue sequences shared via **GISAID**.

## License

The **code** in this repository is released under the MIT License. The **thesis text and
poster** remain © Emmanuel Nyandu Kagarabi. The Dengue sequence data is governed by the
**GISAID** terms of use and is not distributed here.
