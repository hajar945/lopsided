# Claims ledger (Wed Oct 28)

Every number a reader could ask about, where it came from, how strong it is, and the wording to use. **Status key:** PLANNED = in the project statement before results; EXPLORATORY = run after seeing results; DESCRIPTIVE = a measurement, no test; HYPOTHESIS = not tested; SOURCED = from a cited outside source; CHECK = verify before use.

| # | Claim | Evidence | Status | Say | Do not say |
|---|---|---|---|---|---|
| 1 | Interpretable model 0.207 vs U-Net 0.197 (mirror) and 0.193 (raw) | Cell 8 | PLANNED (margin +/-0.05 fixed in advance) | "Equivalent at +/-0.05; non-inferior at +/-0.02 and +/-0.03; intervals include 0" | "matches", "beats the U-Net" |
| 2 | Three regimes by training size | Cell 9, F | PLANNED curve; n=10/20/40 re-checked over 3 draws for xgb and threshold; U-Net and n=80 one draw | "On mean patient Dice" before every size claim | "U-Net needs exactly 120" (it is a tie around there) |
| 3 | A tuned threshold beats both learned models at about 10 scans | Cell 9, F | PLANNED; noisy (3 validation patients at n=10) | "About 10" | a precise cutoff |
| 4 | U-Net ties the simple model at about 118 scans (largest size tested) | Cell 9 (+0.010, CI -0.010 to +0.031) | PLANNED | "ties" | "catches up and wins" |
| 5 | Pooled by lesion volume: raw U-Net ahead by 0.043 (CI 0.010 to 0.074); mirror U-Net tied | Cell J | EXPLORATORY, post hoc, full data only, one of three comparisons | "Exploratory; depends on weighting" | "the U-Net wins" or "never wins" without the weighting |
| 6 | Left-right features add 0.010 Dice (CI 0.003 to 0.017) | Cells 13, G | PLANNED ablation; position ablation (-0.011) single run | "small but real" | "asymmetry drives accuracy" |
| 7 | Trust flag beats random abstention (p = 0.011) | Cell 12 | PLANNED | "better than chance" | "reliable" |
| 8 | Size rule gains +0.052 vs flag +0.030; difference not distinguishable | Cells 12, E (CI -0.005 to +0.049) | EXPLORATORY comparison | "comparable; the size rule needs no second model" | "size rule is better" |
| 9 | Lesions in several pieces: Dice 0.109 vs 0.317 (0.184 vs 0.379 in larger lesions) | Cell D | EXPLORATORY association, simple model only | "associated with" | "causes" |
| 10 | Sensitivity 0.39 / precision 0.22 (interpretable); other models in README | Cell H | DESCRIPTIVE, derived; precision omits patients with no prediction | "finds about 4 in 10 lesion voxels" | any interval or significance |
| 11 | Subgroups by sex, age, severity | Cell I | EXPLORATORY check; no intervals; confounded by lesion size | "no sign of a sex gap; age and severity track lesion size" | any fairness claim |
| 12 | Predicted volume does not forecast recovery (rho 0.00; AUC 0.658 vs 0.660) | Cell 13b | EXPLORATORY, labelled | "no relation found" | "our model predicts recovery" |
| 13 | Resolution shift: drops 0.039 / 0.033 / 0.011 | Cell 10 | DESCRIPTIVE, confounded by half the training data | "weak stand-in" | "robust" or "hospital shift" |
| 14 | Simple model trains 4.6x faster (CPU vs T4); predicts slower | Cell 8 | DESCRIPTIVE; different hardware | "different hardware" | "cheaper at everything" |
| 15 | Tmax>6 s region 157 mL vs 32 mL final lesion | Cell 4 | DESCRIPTIVE | "marks tissue at risk; much was saved" | "the rule is wrong" |
| 16 | Four named patients (0012, 0145, 0066, 0095) | demo notebook, fold 0 | ILLUSTRATIVE. Picked by rule among the 30 fold-0 patients, not at random: best Dice (0095); median Dice (0012); worst miss among larger lesions (0066); smallest lesion (0145). 0012 sits at the fold-0 median, so its low Dice (0.13) is close to the middle of that fold | "illustrative cases picked by rule" | "typical", "random" |
| 17 | Hemisphere AUC 0.96 to 0.98 | Week 2 | SANITY CHECK; easy by construction | "sanity check" | "evidence" |
| 18 | Models share a ceiling because of the data and labels | none | HYPOTHESIS | "one reading, not tested" | "proves" |
| 19 | User is an imaging/AI team with 50 to 150 labelled cases | none | HYPOTHESIS until a clinician or mentor confirms | "our intended user (hypothesis)" | "hospitals need this" |
| 20 | 87% ischemic; costs nearly $56.2B (2019 to 2020) | CDC page snippets | SOURCED | cite CDC | |
| 21 | 795,000 strokes per year | archived pages | dropped: not on the current CDC page | use: "About 87% of US strokes are ischemic; costs nearly $56.2 billion in 2019 to 2020 (CDC)."| quote it only after checking |
| 22 | Ghost core; threshold errors both ways | abstracts, one 22-patient poster | SOURCED, small studies | "documented in small studies" | "well established" |
| 23 | ISLES'24 top Dice 0.285 | arXiv 2408.10966 | SOURCED context | "scale of the task" | "we are close to the winners" |
| 24 | Novelty: question and control structure | literature passes 1 and 2 | NARROW | "extension, never first" | "first", "new algorithm" |
| 25 | Reproducibility | fold 0 xgb reproduced exactly | PARTIAL: U-Net and whole pipeline not rerun in one go | "three-stage, partly verified" | "one-click" |
| 26 | Brain mask is a box (2.7 L vs about 1.3 L) | Cell C | LIMITATION | state it | |

**Pre-registered, in the strict sense, only:** the +/-0.05 margin, patient-level folds, identical tuning, mean patient Dice as the planned primary metric. Say "planned" or "exploratory" next to every other number.
