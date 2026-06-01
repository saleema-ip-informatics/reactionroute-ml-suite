# ReactionRoute ML Suite
### Computational Retrosynthesis & Reaction Intelligence for EGFR Kinase Inhibitors

**Author:** Saleema Begam A — Cheminformatics Scientist  
**Target:** Schrödinger Retrosynthesis Researcher (ML) Role  
**Dataset:** ChEMBL EGFR (CHEMBL203) · USPTO Open Reaction Dataset · EGFR Patent Literature

---

## Project Overview

This project builds an end-to-end machine learning pipeline for retrosynthetic analysis and reaction intelligence, applied to FDA-approved EGFR kinase inhibitors (Erlotinib, Gefitinib, Osimertinib, Lapatinib, Afatinib).

The suite replicates and automates core workflows from pharmaceutical patent analysis — scaffold identification, synthetic route planning, reaction condition optimisation, and potency prediction — using modern ML architectures aligned with industry-standard tools (AiZynthFinder, ASKCOS, RDKit, PyTorch).

---

## Notebooks

### `07_retrosynthesis_aizynthfinder.ipynb` — MCTS Retrosynthesis
**Tools:** AiZynthFinder (AstraZeneca) · ASKCOS (MIT) · RDKit

Applies Monte Carlo Tree Search with a USPTO-trained neural network policy to decompose EGFR inhibitors into commercially purchasable precursors. Integrates patent-aware synthetic accessibility profiling using Bemis-Murcko scaffold extraction and Lipinski physicochemical descriptors.

- Retrosynthetic route prediction for 5 clinical EGFR inhibitors
- Patent-aware precursor analysis (US5747498, EP0566226, WO2013014448)
- Scaffold family comparison: Quinazoline vs Acrylamide-Pyrimidine series
- ASKCOS MIT API integration for cross-validation of routes
- Checkpoint saving per compound — crash-safe on Colab free tier

**Key output:** `results/metrics/retrosynthesis_results.csv` · `results/figures/erlotinib_retrosynthesis_route.png`

---

### `08_graph_neural_network.ipynb` — GCN for pIC50 Prediction
**Tools:** PyTorch Geometric · RDKit · ChEMBL API

Implements a 3-layer Graph Convolutional Network (GCN) that operates directly on molecular graphs — encoding atoms as nodes and bonds as edges — to predict EGFR inhibitor potency (pIC50). Outperforms classical fingerprint-based models by capturing topological molecular structure.

- Molecule → graph conversion using RDKit atom/bond features
- MolecularGCN: GCNConv → BatchNorm → Dropout → Global Mean Pool → MLP
- 80/20 train-test split · Adam optimiser · StepLR scheduler · 60 epochs
- Benchmark comparison: GCN vs Morgan FP baselines (SVR, ElasticNet, Ridge, PyTorch NN)

**Key output:** `results/metrics/gcn_model.pt` · `results/figures/gcn_training_results.png`

---

### `09_transformer_reaction_prediction.ipynb` — Transformer for Property Prediction
**Tools:** PyTorch · SMILES tokenizer · ChEMBL API

Builds a character-level SMILES Transformer encoder for pIC50 regression. Multi-head self-attention captures long-range atom interactions that fingerprint and graph-convolution methods miss — directly analogous to production models in ChemBERTa and MolBERT.

- Character-level SMILES tokeniser with `<PAD>/<UNK>` handling
- SMILESTransformer: Token embedding + positional encoding → 4-head attention (2 layers) → mean pooling → MLP
- Correct padding mask computed from token ids (pre-embedding)
- 60-epoch training with LR scheduling · R² and RMSE evaluation

**Key output:** `results/metrics/transformer_model.pt` · `results/figures/transformer_training_results.png`

---

### `10_reaction_conditions_prediction.ipynb` — Yield & Condition Prediction
**Tools:** RDKit · scikit-learn · Patent literature

Predicts reaction yield and identifies optimal conditions for EGFR inhibitor synthesis reactions curated from 4 key patents. Encodes reaction type, solvent, temperature, catalyst, and Morgan fingerprint features into a unified feature matrix for ML regression.

- 10 curated patent reactions: SNAr coupling, acylation, N/O-alkylation, Suzuki coupling
- Morgan fingerprint (64-bit) + 8 RDKit molecular descriptors + reaction-type one-hot encoding
- 3 models compared: Random Forest · Gradient Boosting · Ridge Regression
- Leave-One-Out CV (correct strategy for small pharmaceutical datasets)
- 4-panel analysis: yield by reaction type, solvent, temperature, and model comparison

**Key output:** `results/metrics/yield_prediction_results.csv` · `results/figures/reaction_conditions_analysis.png`

---

## Results Summary

| Model | Task | Metric |
|---|---|---|
| GCN (NB08) | pIC50 regression | R² reported after training |
| Transformer (NB09) | pIC50 regression | R² reported after training |
| Random Forest (NB10) | Yield prediction | LOO-CV R² |
| Gradient Boosting (NB10) | Yield prediction | LOO-CV R² |
| AiZynthFinder (NB07) | Retrosynthesis | Routes found per compound |

*Exact R² values are computed live during notebook execution on ChEMBL data.*

---

## Installation & Usage

All notebooks are designed to run on **Google Colab** (free tier compatible).

```bash
# Clone the repository
git clone https://github.com/<your-username>/reactionroute-ml-suite.git
cd reactionroute-ml-suite

# Open any notebook in Colab
# Each notebook installs its own dependencies in Cell 1
```

**NB07 note:** AiZynthFinder requires ~2 GB model download (USPTO policy + ZINC stock). Run Step 2 once and keep the `/content/aizynthfinder_data/` folder. On Colab free tier, `iteration_limit=20` is set to prevent RAM crashes; increase to 100+ on Colab Pro or local machine.

---

## Repository Structure

```
reactionroute-ml-suite/
│
├── 07_retrosynthesis_aizynthfinder.ipynb
├── 08_graph_neural_network.ipynb
├── 09_transformer_reaction_prediction.ipynb
├── 10_reaction_conditions_prediction.ipynb
│
├── results/
│   ├── figures/
│   │   ├── egfr_target_compounds.png
│   │   ├── erlotinib_retrosynthesis_route.png
│   │   ├── retrosynthesis_scaffold_analysis.png
│   │   ├── gcn_training_results.png
│   │   ├── transformer_training_results.png
│   │   └── reaction_conditions_analysis.png
│   └── metrics/
│       ├── retrosynthesis_results.csv
│       ├── egfr_synthetic_profiles.csv
│       ├── gcn_model.pt
│       ├── gcn_model_comparison.csv
│       ├── transformer_model.pt
│       ├── yield_prediction_results.csv
│       └── egfr_patent_reactions.csv
│
└── README.md
```

---

## Technical Stack

| Category | Tools |
|---|---|
| Retrosynthesis | AiZynthFinder 4.3, ASKCOS MIT API |
| Cheminformatics | RDKit, Bemis-Murcko scaffolds, SMARTS pharmacophores |
| Deep Learning | PyTorch, PyTorch Geometric, GCNConv, TransformerEncoder |
| Classical ML | scikit-learn (Random Forest, Gradient Boosting, Ridge) |
| Data Sources | ChEMBL EGFR (CHEMBL203), USPTO Open Reaction Dataset |
| Patent Coverage | US5747498 · EP0566226 · WO2013014448 · WO9935146 · WO2002050043 |

---

## Biological Context

All models are applied to **EGFR (Epidermal Growth Factor Receptor)** kinase inhibitors — a clinically validated oncology target with a rich patent landscape spanning three generations of drugs:

- **1st gen:** Erlotinib, Gefitinib — reversible quinazoline inhibitors
- **2nd gen:** Afatinib, Lapatinib — irreversible/dual HER inhibitors  
- **3rd gen:** Osimertinib — mutant-selective, acrylamide covalent warhead

This progression makes EGFR an ideal test case for Markush-aware retrosynthesis: each generation introduces distinct scaffold chemistry, new reaction types, and different synthetic accessibility challenges.

---

## Related Work

This suite is part of a broader cheminformatics project (Notebooks 01–10) covering:
- Reaction data extraction from ChEMBL and USPTO (NB01)
- Yield prediction with classical ML (NB02)
- Retrosynthetic route scoring (NB03)
- Markush structure and SAR analysis (NB04)
- GNN-based reaction classification (NB05)

---

*Built as part of ongoing research in computational retrosynthesis and ML-driven drug discovery.*
