# Model card: Lopsided interpretable voxel model (research benchmark)

**Not a clinical tool. Not validated for diagnosis or treatment decisions.**

**What it is.** A gradient-boosted model (XGBoost, 27 per-voxel features) that predicts which brain voxels end up in the follow-up stroke lesion, from four CT-perfusion maps (CBF, CBV, MTT, Tmax). Compared against fixed threshold rules, a tuned single threshold and two small 2-D U-Nets.

**Intended use.** Comparing model families on small labelled stroke datasets; a transparent, CPU-trainable baseline; studying where models fail.

**Not intended for.** Treatment eligibility, patient triage, lesion volume reporting, outcome prediction, or any use on patients.

**Data.** ISLES'24 training set, 147 patients, one public dataset, no site column, no external validation. Labels are post-treatment follow-up infarcts after successful reperfusion, so saved tissue is not in the mask.

**Evaluation.** 5 patient-level folds; Dice per patient; paired bootstrap over patients; +/-0.05 equivalence margin fixed in advance.

**Performance (mean patient Dice).** 0.207 (sd of the five fold means 0.018; not a confidence interval). Single threshold 0.173; mirror U-Net 0.197; raw U-Net 0.193. Sensitivity 0.39, precision 0.22 (per-patient means). Pooled by lesion volume, the raw U-Net is ahead (0.365 vs 0.322; exploratory).

**Where it fails.** Smallest lesion quartile Dice 0.05; lesions in several pieces 0.109 vs 0.317; lesions crossing the midline 0.07; saved tissue produces false alarms; large lesions with little Tmax delay are missed. Predicted volume tracks true volume weakly (rho 0.22) and does not relate to 3-month recovery.

**Subgroups (check only).** Women 0.216, men 0.200; age thirds 0.220 / 0.207 / 0.194; severity thirds 0.159 / 0.178 / 0.294, largely explained by lesion size. No intervals.

**Cost.** About 179 s training per fold on CPU, 677 KB; about 3.2 s per patient to predict on CPU.

**Known limitations.** Box-shaped brain mask; 4x downsampling erodes tiny lesions; clinical rules are our approximations, not commercial software; no time-from-onset or reperfusion inputs; one dataset.

**Ethical notes.** No identifiable data is redistributed; the dataset is public and must be cited (doi 10.1148/ryai.250603). Subgroup results are too small to support any fairness conclusion.

**Version.** Notebook v4h, pinned in requirements.txt (Python 3.13).
