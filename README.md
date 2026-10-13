
# Machine Learning-Guided Discovery of EGFR Inhibitors

## Project Overview

This project develops an integrated computational drug discovery pipeline to identify and prioritize potential small-molecule inhibitors of the epidermal growth factor receptor (EGFR).

The workflow combines experimental bioactivity data, molecular feature engineering, machine learning, virtual screening, drug-likeness evaluation, structural diversity analysis, and molecular docking.

Machine learning models were trained to predict EGFR inhibitory activity (pIC50). The best-performing model was subsequently applied to an external ZINC compound library to identify promising candidates for structure-based evaluation.

**Current status:** Machine learning, virtual screening, and candidate prioritization are complete. Molecular docking is in progress, with EGFR receptor preparation completed.

---

## 1. Project Objectives

1. Collect and preprocess experimentally measured EGFR inhibitory activity data.
2. Generate physicochemical descriptors and Morgan fingerprints using RDKit.
3. Train and compare machine learning models for pIC50 prediction.
4. Evaluate model performance using random splits, cross-validation, and scaffold-based splitting.
5. Screen an external ZINC compound library for potential EGFR inhibitors.
6. Prioritize compounds based on predicted activity, chemical similarity, drug-likeness, and structural diversity.
7. Perform molecular docking against the EGFR kinase domain.
8. Evaluate candidate binding poses and docking scores relative to a crystallographic reference ligand.

---

## 2. Repository Structure

```text
egfr-ml-drug-discovery/
│
├── data/
│   ├── raw/
│   │   ├── egfr_ic50_filtered_raw.csv
│   │   └── zinc_lead_like_random.csv
│   └── processed/
│       └── egfr_molecule_activity_with_smiles.csv
│
├── notebooks/
│   ├── 01_data_collection.ipynb
│   ├── 02_data_preprocessing.ipynb
│   ├── 03_molecular_features.ipynb
│   ├── 04_model_training.ipynb
│   ├── 05_candidate_screening.ipynb
│   └── 06_molecular_docking.ipynb
│
├── src/
│   ├── __init__.py
│   ├── descriptors.py
│   ├── modeling.py
│   └── preprocessing.py
│
├── docking/
│   ├── input/
│   └── output/
│
├── results/
│   ├── candidates/
│   ├── models/
│   └── model_comparison.csv
│
├── figures/
├── docs/
├── requirements.txt
├── .gitignore
└── README.md
```

---

## 3. Computational Workflow

### 3.1. EGFR Bioactivity Data Collection

Experimental EGFR inhibitory activity data were collected and processed to create a molecular dataset containing chemical structures and IC50 measurements.

IC50 values were converted to pIC50 values for continuous regression modeling:

\[
pIC50 = -\log_{10}(IC50\;[\mathrm{M}])
\]

The final modeling dataset contained **10,381 compounds**.

**Notebook:** `01_data_collection.ipynb`

### 3.2. Molecular Preprocessing

Molecular structures were processed using RDKit to generate standardized molecular representations.

Preprocessing included SMILES validation, molecular structure handling, and canonical SMILES generation.

**Notebook:** `02_data_preprocessing.ipynb`

### 3.3. Molecular Feature Engineering

Two primary molecular representations were generated:

- **Physicochemical descriptors:** Molecular weight, LogP, hydrogen-bond donors and acceptors, topological polar surface area, and rotatable bonds.
- **Morgan fingerprints:** Radius 2, 2,048-bit molecular fingerprints encoding structural features.

Combined descriptor and fingerprint representations were also evaluated.

**Notebook:** `03_molecular_features.ipynb`

### 3.4. Machine Learning Model Development

Three regression algorithms were evaluated:

1. Linear Regression
2. Random Forest Regression
3. Neural Network Regression

Models were evaluated using:

- Coefficient of determination (R²)
- Root mean squared error (RMSE)
- Mean absolute error (MAE)

Random train-test splitting, five-fold cross-validation, and scaffold-based splitting were used to evaluate predictive performance and generalization.

**Notebook:** `04_model_training.ipynb`

### 3.5. Machine Learning Results

| Model | Features | Evaluation | R² | RMSE | MAE |
|---|---|---|---:|---:|---:|
| Linear Regression | Descriptors | Random split | 0.136 | 1.207 | 0.976 |
| Random Forest | Descriptors | Random split | 0.419 | 0.989 | 0.749 |
| Random Forest | Morgan fingerprints | Random split | 0.721 | 0.685 | 0.506 |
| Random Forest | Fingerprints + descriptors | Random split | 0.709 | 0.700 | 0.517 |
| Random Forest | Morgan fingerprints | 5-fold CV | 0.711 | 0.702 | 0.510 |
| Random Forest | Morgan fingerprints | Scaffold split | 0.619 | 0.784 | 0.584 |
| Neural Network | Morgan fingerprints | Random split | 0.637 | 0.782 | 0.586 |
| Neural Network | Morgan fingerprints | Scaffold split | 0.500 | 0.898 | 0.673 |

Five-fold cross-validation results for the Morgan fingerprint Random Forest model:

| Metric | Mean ± SD |
|---|---|
| R² | 0.711 ± 0.012 |
| RMSE | 0.702 ± 0.017 |
| MAE | 0.510 ± 0.010 |

The Morgan fingerprint Random Forest model demonstrated the strongest performance among the evaluated random-split models.

Its scaffold-split performance (R² = 0.619) exceeded that of the neural network (R² = 0.500), supporting its selection for external compound screening.

The final Random Forest model was trained using all 10,381 compounds and saved as:

`results/models/egfr_morgan_rf_final.joblib`

### 3.6. Virtual Screening

An external library of **10,000 lead-like compounds** was obtained from ZINC22.

The compounds were processed using RDKit, converted into Morgan fingerprints, and screened using the trained Random Forest model.

No canonical SMILES overlap was identified between the screened compounds and the EGFR training dataset.

The 100 highest-predicted compounds were selected for further prioritization.

**Notebook:** `05_candidate_screening.ipynb`

### 3.7. Drug-Likeness and Chemical Similarity

The top 100 compounds were evaluated using physicochemical properties and Lipinski's Rule of Five.

**99 of 100 compounds passed the implemented Lipinski criteria.**

Maximum Tanimoto similarity to the EGFR training compounds was calculated using Morgan fingerprints.

Observed maximum similarity values ranged from approximately **0.157 to 0.397**, indicating that the selected compounds were structurally distinct from the training molecules.

These low similarity values also indicate potential uncertainty when extrapolating predicted inhibitory activity to previously unseen chemical structures.

### 3.8. Candidate Prioritization

Candidates were ranked using a weighted score combining:

- 70% normalized predicted pIC50
- 30% normalized maximum training-set similarity

Candidates passing the Lipinski criteria were prioritized.

Structural diversity was subsequently considered using pairwise Tanimoto similarity to reduce selection of highly redundant chemical structures.

A final set of **10 candidate compounds** was selected for molecular docking.

Candidate results are stored in:

`results/candidates/`

---

## 4. Molecular Docking

**Notebook:** `06_molecular_docking.ipynb`

### 4.1. EGFR Structure Selection

The EGFR crystal structure was obtained from the Protein Data Bank.

- **PDB ID:** 6LUD
- **Protein chain:** A
- **Co-crystallized ligand:** YY3
- **Target region:** EGFR kinase domain

The protein and reference ligand were extracted into separate structural files.

Initial extraction produced:

| Component | Count |
|---|---:|
| Protein residues | 310 |
| Protein atoms | 2,388 |
| Reference ligand atoms | 37 |

### 4.2. Binding-Site Identification

The crystallographic ligand was used to identify the EGFR binding pocket.

Protein residues within 5 Å of the ligand were identified using interatomic distance calculations.

A total of **19 binding-site residues** were identified, including:

- LYS745
- LEU792
- MET793
- ASP800
- GLU804

These residues were used to characterize the local binding environment.

### 4.3. Docking Grid Definition

The docking search region was defined using the spatial coordinates of the co-crystallized ligand.

An 8 Å padding was added on each side of the ligand's coordinate bounds.

| Axis | Center (Å) | Size (Å) |
|---|---:|---:|
| X | -49.874 | 22.877 |
| Y | 0.026 | 24.719 |
| Z | -18.135 | 29.918 |

The grid is intended to encompass the experimentally observed ligand-binding region while allowing conformational exploration.

### 4.4. Receptor Inspection and Preparation

PDBFixer was used to inspect the extracted EGFR receptor.

Structural inspection identified:

- 310 protein residues
- 22 residues containing missing heavy atoms
- No nonstandard residues
- No missing heavy atoms among the 19 identified binding-site residues
- One missing terminal OXT atom

Although residue-numbering gaps were observed, PDBFixer did not propose reconstruction of entire missing residue segments.

Missing heavy atoms were reconstructed, and hydrogen atoms were added at pH 7.4.

The prepared receptor contained:

| Property | Value |
|---|---:|
| Total atoms | 4,981 |
| Total residues | 310 |
| Hydrogen addition pH | 7.4 |

Prepared receptor file:

`docking/input/egfr_prepared.pdb`

### 4.5. Remaining Docking Workflow

The following steps are planned:

1. Validate the prepared receptor by comparing original and prepared heavy-atom coordinates.
2. Convert the prepared receptor into AutoDock-compatible PDBQT format.
3. Prepare the crystallographic reference ligand and selected ZINC compounds.
4. Validate the docking protocol, including assessment of reference-ligand redocking where appropriate.
5. Dock the selected candidates using AutoDock Vina.
6. Compare docking scores and binding poses.
7. Examine protein-ligand interactions and prioritize compounds for further investigation.

Because the selected structure contains a co-crystallized ligand that may involve covalent binding, the suitability of conventional noncovalent docking and reference-ligand redocking must be assessed before interpreting docking results.

**Docking results are pending.**

---

## 5. Software and Dependencies

The project uses the following computational tools:

| Software | Application |
|---|---|
| Python | Computational analysis |
| pandas / NumPy | Data processing |
| RDKit | Molecular descriptors, fingerprints, similarity |
| scikit-learn | Machine learning and evaluation |
| Matplotlib | Data visualization |
| Biopython | Protein structure analysis |
| SciPy | Distance calculations |
| PDBFixer | Protein structure repair |
| OpenMM | Molecular structure handling |
| Meeko | Docking input preparation |
| AutoDock Vina | Molecular docking |
| Open Babel | Chemical structure conversion |

### Environment

A dedicated Conda environment named `egfr-drug-discovery` is used.

```bash
conda activate egfr-drug-discovery
```

Dependencies are documented in `requirements.txt` and the Conda environment.

---

## 6. Reproducibility

The analysis is organized into sequential Jupyter notebooks.

Recommended execution order:

```text
01_data_collection.ipynb
        ↓
02_data_preprocessing.ipynb
        ↓
03_molecular_features.ipynb
        ↓
04_model_training.ipynb
        ↓
05_candidate_screening.ipynb
        ↓
06_molecular_docking.ipynb
```

Generated datasets, trained models, candidate rankings, figures, and docking outputs are organized into their corresponding repository directories.

---

## 7. Limitations

Several limitations should be considered when interpreting the results.

**Model generalization:** Scaffold-based evaluation showed reduced predictive performance compared with random splitting, indicating sensitivity to structural differences between training and test compounds.

**Chemical novelty:** The screened candidates showed low similarity to EGFR training molecules. Although this supports chemical diversity, predictions for these compounds may be less reliable.

**Drug-likeness:** Passing Lipinski criteria does not establish biological activity, safety, or clinical suitability.

**Docking:** Docking scores approximate molecular interactions and do not directly establish experimental binding affinity or inhibitory activity. Receptor conformation, protonation, ligand preparation, and docking methodology may affect the results.

**Experimental validation:** All predicted EGFR inhibitors require experimental testing before inhibitory activity can be confirmed.

---

## 8. Current Progress

| Project Stage | Status |
|---|---|
| EGFR bioactivity data collection | Complete |
| Molecular preprocessing | Complete |
| Molecular feature generation | Complete |
| Machine learning model comparison | Complete |
| Cross-validation and scaffold evaluation | Complete |
| Final model training | Complete |
| ZINC virtual screening | Complete |
| Drug-likeness evaluation | Complete |
| Chemical similarity analysis | Complete |
| Candidate prioritization | Complete |
| EGFR structure retrieval | Complete |
| Binding-site identification | Complete |
| Docking grid definition | Complete |
| Receptor repair and hydrogen addition | Complete |
| Prepared receptor validation | Pending |
| Receptor and ligand PDBQT preparation | Pending |
| Docking protocol validation | Pending |
| Molecular docking | Pending |
| Docking results analysis | Pending |

---

## 9. Conclusion

This project demonstrates an integrated computational workflow for prioritizing potential EGFR inhibitors through machine learning and structure-based analysis.

Morgan fingerprint Random Forest regression provided the strongest overall predictive performance among the evaluated models and was used to screen 10,000 external compounds.

Drug-likeness, chemical similarity, and structural diversity analyses were incorporated to prioritize compounds for further evaluation.

Molecular docking will provide an additional structure-based assessment of the selected candidates, complementing the ligand-based machine learning predictions.

The resulting candidates represent computational hypotheses requiring further validation rather than experimentally confirmed EGFR inhibitors.
