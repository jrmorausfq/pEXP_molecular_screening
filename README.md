# pEXP Molecular Screening

Python workflow for molecular screening of pEXP using molecular descriptors from **ToMoCoMD**, **PaDEL APC2D**, and **RDKit**, followed by a common applicability-domain filter and prediction with pre-trained **scikit-learn** regression and classification models.

The workflow is designed for screening molecules stored as 3D `.sdf` files and writes the main prediction outputs to `Screening_Results/`.

> This repository contains the workflow code and metadata. External programs, trained models, large descriptor tables, and third-party `.jar` files should only be included if their licenses and file sizes allow redistribution.

---

## Main workflow

The script performs the following steps:

1. Reads 3D molecules from `ToMoCoMD/chemical_datasets/*.sdf`.
2. Calculates ToMoCoMD descriptors using `ToMoCoMD-CARDD_CLI.jar`.
3. Calculates selected PaDEL APC2D descriptors using `padelpy`.
4. Calculates selected RDKit descriptors listed in `rdkit_descriptors.txt`.
5. Merges ToMoCoMD, PaDEL, and RDKit descriptors into one matrix.
6. Calculates one common applicability domain using `training_AD.csv`.
7. Keeps only molecules inside the common applicability domain.
8. Predicts pEXP values with regression models and class labels with classification models.
9. Writes final results to `Screening_Results/`.

---

## Recommended repository structure

```text
pEXP-molecular-screening/
├── pEXP_molecular_screening.py
├── README.md
├── LICENSE
├── CITATION.cff
├── environment.yml
├── rdkit_descriptors.txt
├── padel_descriptors.txt          # optional; defaults are used if absent
├── descriptors.xml                # optional PaDEL XML source
├── training_AD.csv                # common AD training descriptors
├── ToMoCoMD/
│   ├── README.md
│   ├── ToMoCoMD-CARDD_CLI.jar     # download manually if redistribution is not allowed
│   ├── headings.txt
│   └── chemical_datasets/
│       └── *.sdf
├── models/
│   ├── regression/
│   │   ├── Ensamble.skl           # or Ensambe.skl / Ensemble.skl
│   │   ├── Ensamble.txt
│   │   ├── Ensamble_train.csv     # optional
│   │   └── individuals/
│   │       ├── *.skl
│   │       ├── *.txt
│   │       └── *_train.csv        # optional
│   └── classification/
│       ├── classifier.skl
│       ├── classifier.txt
│       └── *_train.csv            # optional
└── Screening_Results/             # generated automatically
```

---

## System requirements

A Linux, macOS, or Windows environment with Conda/Mamba is recommended.

Required software:

| Software | Purpose |
|---|---|
| Python 3.10 | Runs the workflow |
| Java JDK/JRE | Runs ToMoCoMD and PaDEL-related Java tools |
| ToMoCoMD-CARDD | Calculates ToMoCoMD molecular descriptors |
| RDKit | Reads molecules and calculates RDKit descriptors |
| PaDEL / padelpy | Calculates selected APC2D descriptors |
| scikit-learn / joblib | Loads and runs the trained models |

---

## Installation

Create the Conda environment:

```bash
conda env create -f environment.yml
conda activate pexp-molecular-screening
```

Check the installation:

```bash
python -c "import numpy, pandas, joblib, sklearn; from rdkit import Chem; import padelpy; print('OK')"
java -version
```

---

## Required input files

### 1. Molecules to screen

Place 3D SDF files in:

```text
ToMoCoMD/chemical_datasets/*.sdf
```

### 2. RDKit descriptor list

Create a file named:

```text
rdkit_descriptors.txt
```

with one RDKit descriptor name per line, for example:

```text
MolWt
MolLogP
TPSA
NumHDonors
NumHAcceptors
```

### 3. Optional PaDEL descriptor list

If `padel_descriptors.txt` is absent, the script uses the default PaDEL descriptors:

```text
APC2D2_C_F
APC2D3_C_F
APC2D1_C_F
```

### 4. Common applicability-domain training table

The file:

```text
training_AD.csv
```

must contain the descriptor columns used to define the applicability domain. The screening descriptor matrix is reordered to match the exact column order in this file before AD calculation.

### 5. Trained models

Regression models should be placed in:

```text
models/regression/
models/regression/individuals/
```

Classification models should be placed in:

```text
models/classification/
```

The `.txt` files beside each model should contain the feature names or feature indexes expected by that model. Optional `*_train.csv` files can be used by the script to recover training feature order and medians.

---

## Running the workflow

From the repository root:

```bash
python pEXP_molecular_screening.py
```

Useful options:

```bash
# Generate descriptors only
python pEXP_molecular_screening.py --descriptors-only

# Resume from existing Screening_Results/descriptors_merged.csv
python pEXP_molecular_screening.py --use-existing-descriptors

# Run AD only, without predictions
python pEXP_molecular_screening.py --ad-only

# Keep only the final output files after a successful run
python pEXP_molecular_screening.py --final-only

# Use a custom AD training CSV
python pEXP_molecular_screening.py --ad-training-csv path/to/training_AD.csv
```

---

## Main outputs

The workflow writes files inside `Screening_Results/`. The most important outputs are:

| File | Description |
|---|---|
| `results_reg.csv` | Regression predictions, including the ensemble pEXP prediction |
| `results_class.csv` | Classification predictions and class scores |
| `molecules_outside_AD.csv` | Molecules excluded by the common applicability-domain filter |
| `screening_log.txt` | Execution log |
| `descriptors_merged.csv` | Merged ToMoCoMD + PaDEL + RDKit descriptor matrix |
| `applicability_domain.csv` | Common applicability-domain audit table |

When `--final-only` is used, the script keeps only:

```text
results_reg.csv
results_class.csv
molecules_outside_AD.csv
screening_log.txt
```

---

## Reproducibility notes

For a reproducible release, report or archive the following:

- repository release version;
- Zenodo DOI;
- Python version;
- Java version;
- RDKit version;
- scikit-learn version;
- padelpy / PaDEL version;
- ToMoCoMD-CARDD version or download date;
- model files used for regression and classification;
- `training_AD.csv` used for the applicability domain;
- descriptor lists used by RDKit and PaDEL.

---

## Zenodo archiving workflow

1. Push the repository to GitHub.
2. Add `CITATION.cff`, `LICENSE`, `README.md`, and `environment.yml` to the repository root.
3. Create a GitHub release, for example `v1.0.0`.
4. Link the GitHub repository with Zenodo.
5. Archive the GitHub release in Zenodo.
6. Copy the DOI assigned by Zenodo.
7. Update the citation section below and, if desired, add the DOI to `CITATION.cff`.
8. Create a new GitHub release after updating the metadata.

---

## Citation

If you use this software, please cite the archived Zenodo release. After Zenodo assigns a DOI, update this section:

```text
Mora, J. R. pEXP Molecular Screening. Version 1.0.0. Zenodo. https://doi.org/10.5281/zenodo.20787378
```

GitHub should also display a **Cite this repository** button when `CITATION.cff` is present in the repository root.

---

## License

This project is distributed under the MIT License. See the `LICENSE` file.

The MIT License applies only to the code and documentation in this repository. It does not automatically apply to third-party software, external `.jar` files, trained models, or datasets unless those files are explicitly released under compatible terms.
