# Data support: provenance, attribution, and reuse

This folder contains **patient-level derived tables** used by Lopsided's research benchmark. They were made from the public ISLES'24 stroke dataset; they are **not** the original CT/MRI images, and they are **not** independent synthetic data.

## Files

| File | Contents and origin |
| --- | --- |
| `features_v2.csv` | One row per analyzed patient, with computed CT-perfusion summaries, left/right and regional features, lesion measurements, and selected dataset-supplied clinical fields (including age, sex, and admission NIHSS). Produced by `notebooks/week2_features_eda.ipynb`. |
| `folds.csv` | Pseudonymous subject identifiers, patient-level cross-validation fold assignments, and lesion laterality. Generated for this benchmark's evaluation split. |

These tables correspond to the **147 analyzed patients** described in the repository. Their `sub-stroke...` subject IDs link rows back to the public dataset. The tables do not include raw image arrays, but they do retain individual-level clinical and derived imaging information.

## Original dataset and credit

- **Data:** *The Ischemic Stroke Lesion Segmentation Challenge (ISLES)'24 Dataset*, released by the ISLES'24 dataset contributors.
- **Dataset paper:** *The Ischemic Stroke Lesion Segmentation Challenge (ISLES)'24 Dataset: A Multimodal Stroke Imaging Dataset with Hyperacute CT, Acute Postinterventional MRI, and 3-month Clinical Outcomes*. https://doi.org/10.1148/ryai.250603
- **Original dataset archive (Zenodo):** https://doi.org/10.5281/zenodo.17652035
- **Preconverted Parquet mirror used by the notebook workflow:** https://huggingface.co/datasets/hugging-science/isles24-stroke/tree/main/data (credited to the Hugging Science dataset publisher).
- **Original dataset license:** [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International (CC BY-NC-SA 4.0)](https://creativecommons.org/licenses/by-nc-sa/4.0/).

**Changes made by Lopsided:** Instead of republishing the source scans, the Week-2 notebook extracts patient-level numerical features and clinical fields, calculates lesion summaries, and assigns patient-level folds for model evaluation. These CSVs are derived outputs; neither is a separately collected cohort.

If you reuse these files or underlying inputs, credit the ISLES'24 dataset authors, cite the dataset paper, acknowledge the Parquet mirror if used, and describe the modifications.

## Licensing and responsible reuse

The repository's **MIT license covers Lopsided's original code and documentation**; it must **not** be interpreted as relicensing ISLES'24 data or automatically covering these two CSVs.

The ISLES'24 source is distributed under **CC BY-NC-SA 4.0**, which includes attribution, noncommercial, and ShareAlike conditions when applicable. These CSVs are derived from that source. **To the extent their redistribution implicates protected source material or database rights, follow the source license**, unless you have separate permission or a legal exception. Whether all individual numerical values or portions of these tables are themselves protected by such rights has not been legally determined here. Do not assume that GitHub availability grants additional commercial-use rights.

This documentation is **not** a determination that publishing patient-level derived tables is permitted for every reuse or jurisdiction. The tables include pseudonymous subject IDs and clinical attributes. Public availability and removal of direct identifiers do not guarantee zero re-identification risk. Consult the dataset's governing terms and an appropriate institutional data/privacy reviewer before republishing, combining with other patient-level information, or using commercially. Do not attempt to re-identify subjects.

For the complete data source and experiment workflow, see the [main README](../README.md) and [Week-2 notebook](../notebooks/week2_features_eda.ipynb).
