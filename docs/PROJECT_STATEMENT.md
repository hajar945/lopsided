# PROJECT STATEMENT AND TOPIC DETAILS (v3)

## 1. Title (descriptive)
**When Does Simple Beat Deep? A Data-Efficiency Study, With an Image-Resolution Shift Check, of Interpretable vs. Deep Voxel-Level Stroke-Lesion Prediction from Perfusion CT, With a Built-In Trust Flag**

Working name: **Lopsided**

> **Status note.** Written before the full run and edited afterwards for accuracy. What changed from the plan: the planned "unseen site" check became an image-resolution check (the data has no site column); voxel-level ROC-AUC was not saved, so sensitivity and precision are derived from stored Dice and volumes; pooled Dice, subgroups, the several-pieces analysis and the size-rule comparison were added after seeing results (exploratory). Only the +/-0.05 margin, patient-level folds, identical tuning and mean patient Dice were fixed in advance. See [`docs/CLAIMS_LEDGER.md`](CLAIMS_LEDGER.md) for the status of every number.

## 2. The one-paragraph version
About 87% of US strokes are ischemic, and stroke-related costs were nearly $56.2 billion in 2019 to 2020 (CDC). For selected patients, treatment up to 16 to 24 hours after onset is guided by perfusion CT volumes computed by commercial software using fixed thresholds (core: relative blood flow below 30%; delayed flow: Tmax above 6 s), and small studies report that those thresholds can misjudge the tissue finally lost, in both directions. Our hypothesized intended user is an imaging/AI team with roughly 50 to 150 labelled cases, choosing whether an interpretable model or a deep network is worth pursuing; this user and decision have not yet been validated with practitioners. Lopsided answers that with a controlled experiment on the public ISLES'24 dataset: clinical threshold rules vs. an interpretable gradient-boosting model vs. two U-Nets, scored voxel by voxel against expert lesion masks, as a function of training-set size. Sources: see CLAIMS_LEDGER.md.

## 3. The problem (why anyone should care), with sources
1. **Scale [CDC]:** about 87% of US strokes are ischemic; stroke is a leading cause of serious long-term disability; nearly $56.2 billion in costs (2019 to 2020).
2. **The decision [DEFUSE 3, DAWN]:** perfusion imaging helps select patients for thrombectomy 6 to 24 hours after onset, using software-computed core and delayed-flow volumes with fixed cut-offs.
3. **The weakness [small studies]:** fixed thresholds can overestimate (ghost core) or underestimate the final infarct; the best threshold can change with treatment; a 47-patient study of one commercial software package described *absolute* agreement with follow-up infarct as poor (ICC 0.64 to 0.69; https://pmc.ncbi.nlm.nih.gov/articles/PMC9601142/).
4. **Our data:** across 147 ISLES'24 patients the Tmax>6 s region averaged 157 mL against a 32 mL final lesion (about 5x), consistent with tissue saved by treatment (cause not tested).
5. **The gap:** the software is closed, and small teams lack guidance on how much labelled data a deep network needs before it beats a simple explainable model.

## 3b. User and decision (stated as a hypothesis, to validate)
The decision perfusion imaging informs is thrombectomy eligibility. **This project does not make or change that decision.** Our user is an imaging/AI team with roughly 50 to 150 labelled stroke perfusion cases , choosing which model to build, how many cases to label, and which predictions to send for human review. Stroke teams are the downstream users. This is a hypothesis until a clinician or mentor confirms it.

## 4. The core question (thesis)
Using only information available at admission, can a small set of interpretable, clinically-motivated per-voxel features (relative flow values, left-right mirror differences, local context) locate stroke-damaged tissue as well as a small deep network, **and how does the answer change with (a) the number of training patients, (b) a shift in image resolution (the data has no site column, so this is not a hospital test), and (c) lesion size?**

## 5. Research questions
- **RQ1 Beat the standard.** Does a learned interpretable model beat the fixed clinical threshold rules and the best single-feature rule, in Dice and lesion-volume error?
- **RQ2 Test non-inferiority to a U-Net.** Is the interpretable model non-inferior (margin of ±0.05 Dice fixed in advance, paired bootstrap over patients) to a U-Net, including a U-Net that is *given* the mirror-difference channels?
- **RQ3 Data-efficiency crossover (headline).** Plot Dice against the number of training patients (10 → all). Does the deep model ever overtake the simple one, and at what N*?
- **RQ4 (optional) Resolution shift.** The data has no hospital column, so this is *not* a site test: how much Dice does each approach lose when trained on finer-pixel scans and tested on coarser ones (and the reverse)?
- **RQ5 Failure map + trust.** Where does the simple model fail (lesion size, posterior territory, bilateral/midline)? Does a label-free flag (the interpretable model and the U-Net disagree) identify those failures, and does it beat a trivial rule that abstains on small predicted lesions?

## 6. Data
- **ISLES'24** (Zenodo 16813698; mirrored on Hugging Face): multi-center acute ischemic stroke, perfusion-CT maps (CBF, CBV, MTT, Tmax) with expert-annotated **follow-up infarct** masks (NIfTI). Publicly available under CC BY-NC-SA 4.0 (attribution, non-commercial, share-alike); its license is separate from this project's MIT code license. Distribution of derived tables should be reviewed before further release. See https://doi.org/10.1148/ryai.250603.
- 147 patients in the analysed cohort (149 shards; the notebook reports the exclusions). One patient has 0 mL of lesion on the cached grid (probably lost in the 4x downsample; check), so lesion-based summaries use 146.
- **Checked on the real data:** there is **no** hospital/center column. There are `nihss_admission`, `mrs_admission` and `mrs_3month`. Only the four perfusion maps are model inputs; the follow-up MRI scans (DWI/ADC) and other columns are never inputs. `mrs_3month` is used only to grade results.
- **What the labels are:** the expert mask is the **post-treatment** infarct on a follow-up MRI; ISLES'24 patients all had successful reopening of the blocked vessel, so tissue that was saved is not in the mask. Absolute Dice is therefore expected to be modest.

## 7. Method
1. **Preprocess once:** orientation standardised (47 scans were flipped), brain mask, midline, 4x in-plane downsample (about 1.6 x 1.6 x 2 mm voxels; 2x needs ~11 GB of RAM for features alone and crashes Colab), cached to Drive.
2. **Per-voxel interpretable features (27):** raw maps; values relative to whole-brain median; left-right mirror differences and ratios; smoothed context; position. Built from the scan only, never the lesion mask; an assertion guards against answer-key columns.
3. **Contestants:** `rule_tmax6`, `rule_core` (fixed clinical thresholds: delayed flow, and low flow inside the delayed-flow region); `rule_best` (best single feature + tuned threshold); `xgb` (XGBoost, voxel features); `unet_raw` (4 maps); `unet_mirror` (4 maps + mirror differences).
4. **Fair tuning:** every learned model gets 2 hyper-parameter settings, one patient-level validation split, and a Dice-tuned threshold; mirror-flip augmentation for the U-Nets. Both families train on 80% of the training patients and the winner on the other 20% is scored (no refit), so neither gets extra data.
5. **Evaluation:** identical patient-level folds for all; Dice and volume error per patient, plus sensitivity and precision derived from them (voxel-level ROC-AUC was not saved; the hemisphere AUC from Week 2 is the only AUC); paired bootstrap CIs over patients; equivalence margin fixed before looking at results.
6. **Sweeps:** training-size curve (headline); optional image-resolution shift check; failure strata.
7. **Trust:** label-free interpretable-vs-U-Net volume disagreement as an abstention score, shown as a selective-risk curve against random abstention and a perfect oracle. (Conformal intervals were considered and dropped: hard to defend live, small gain.)
8. **Interpretability shown, not claimed:** SHAP + feature-group ablation (does removing position or mirror features hurt?).

## 8. Deliverables
Three-stage, partly verified notebook workflow (prepared Parquet shards → cached grid → features → all models); results tables with confidence intervals; **hero figure** (head-to-head, data-efficiency curve, resolution-shift gap); failure map; trust-flag curves; SHAP/ablation; cost table (train time, size, inference time); sensitivity and precision table; subgroup check (sex, age, severity); public repo with pinned dependencies; mini-paper deck.

## 9. Scope boundaries
**In:** locating tissue that ended up infarcted, on already-imaged strokes; a controlled methods comparison.
**Out:** clinical use or treatment decisions; predicting future strokes; claims about outperforming state-of-the-art challenge systems (our U-Net is deliberately small); subtype classification. A 90-day-outcome analysis happens only if the column exists, and only as a clearly-labelled extension.

## 10. Prior work and honest positioning
No new algorithm. What exists already (verified in a first literature pass, not exhaustive):
- **Learned models vs clinical thresholds at voxel level:** a study on MRI inputs compared gradient boosting, random forests and a U-Net with clinical thresholding for final-infarct prediction; gradient boosting did best (https://pmc.ncbi.nlm.nih.gov/articles/PMC8756479). So "simple learned model vs U-Net vs clinical rule" is **not** new by itself.
- **Training-size crossovers in stroke:** for predicting stroke *severity* from lesion location, a linear model beat deep learning at 100 patients and the reverse held at 900 (Bourached et al., Brain Communications 2024, doi 10.1093/braincomms/fcae007). Different task, same idea.
- **ISLES'24 itself:** challenge overview arXiv 2408.10966; dataset paper doi 10.1148/ryai.250603; winning-solution note arXiv 2505.18424 (nnU-Net with heavy preprocessing and all CT inputs). We make **no state-of-the-art claim**: our U-Net is small and uses only the perfusion maps.
- **CT-perfusion final-infarct prediction with deep learning** (e.g., Robben et al., arXiv 1812.02496) reports modest Dice and calls the problem inherently hard.
- **Second literature pass (Wed Oct 28, web search, not exhaustive):** (a) the ISLES'24 overview reports the top hidden-test model (multimodal nnU-Net) at Dice 0.285 (+/- 0.213), which sets the scale of this task (arXiv 2408.10966); not comparable to our cross-validated perfusion-only numbers. (b) A 2026 multicenter NCCT segmentation/domain-adaptation preprint reported a median Dice of 0.20 for its ISLES NCCT segmentations against post-treatment infarct, and higher scores for lesions at least 30 mL (arXiv 2609.20151). It uses different inputs and should not be treated as a head-to-head comparison. (c) A 2026 systematic review of acute ischemic stroke segmentation reported deep learning in 78% of included approaches and classical ML in 22% (PMC13445797); its studies span imaging modalities and prediction targets, so their pooled Dice cannot be compared directly with our results. We found no study that combines a voxel-level CT-perfusion training-size curve, a tuned-threshold baseline and a trust-flag test; absence in a quick search is not proof, so we still never say "first".

**What we claim (narrowly):** an extension of that work to the *public CT-perfusion ISLES'24 data* in the *small-labelled-data regime*, with (1) a strong clinical-rule baseline, (2) a U-Net that is also handed the mirror-difference channels, (3) a training-size curve at voxel level, (4) a trust-flag analysis (disagreement flag tested against chance and against a trivial lesion-size rule), and (5) a three-stage Colab workflow using public data (partly verified; the U-Net stage may require paid GPU compute units, and the NIfTI-to-Parquet conversion is not supplied). We do not say "first." Treat this as replication-plus-extension unless a deeper search shows otherwise.

## 11. Known limitations (each paired with a next step)
- Single public dataset, 147 patients → report CIs; next step: external validation set.
- Labels are *follow-up* infarcts after treatment; salvaged tissue is not in the mask → state it; next step: check for treatment/reperfusion fields.
- 4x downsampled masks slightly erode tiny lesions (affects all models equally) → next step: full-resolution run.
- Small, 2D U-Net, not nnU-Net → we make no state-of-the-art claim; next step: nnU-Net as a ceiling reference.
- No hospital column exists: the shift check uses image resolution only and is not a hospital test. It also trains on roughly half the available cases, so the loss is confounded with training size; next step: fixed-training-size external domain testing.
- Labels are post-treatment infarcts after successful reperfusion, so absolute Dice is modest by construction (published perfusion-based work reports modest Dice too).
- The "brain mask" is essentially the rectangular box where the maps have values, not a skull-stripped brain (mean ~2.7 L vs a normal ~1.3 L). Dice ignores true negatives, so model comparisons are probably little affected; a skull-stripped mask is future work.
- The U-Net training-size results and the n = 80 point use one sampled subset per fold; next step: repeat across independent training draws.
- The U-Net selected for follow-up sweeps was chosen using the same out-of-fold cohort; this can favor the deep comparison; next step: choose model family in a separate development set.
- At n = 10, threshold selection uses just three validation patients; next step: repeated validation splits with more patients.
- The clinical-style threshold rules are our own approximations, not output from commercial software; next step: benchmark against actual software outputs.
- No onset-time or per-patient reperfusion/treatment variables were used as model inputs; next step: assess their availability and conduct a controlled extension.
- Mean patient Dice and post hoc pooled Dice weight lesions differently and produce different model rankings; next step: specify the deployment-relevant weighting before a future study.
- The official NIfTI-to-Parquet conversion and full three-stage rerun are not verified; next step: provide the conversion script, complete a clean-room run, and test README steps.
- Research finding, not a clinically validated tool.

## 12. Tools and cost
ISLES'24 (free), Google Colab (free tier; a T4 GPU is needed for the U-Nets, CPU for everything else), Google Drive, GitHub; nibabel, numpy, pandas, scikit-learn, xgboost, shap, torch, matplotlib. **Cost:** the public data and MIT code can be accessed without a purchase; the U-Net stage needs several T4 GPU hours and may require paid Colab compute units. A complete rebuild from the official NIfTI files is not yet demonstrated because the Parquet conversion is undocumented.
