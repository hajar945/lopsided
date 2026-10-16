# Lopsided — Project Claim

## The claim, in one sentence

We show that simple, interpretable left-vs-right blood-flow features match a fairly-tuned small image model at locating stroke damage on held-out patients — and we map the exact conditions under which the simple approach breaks down.

---

## The setup in plain terms

A stroke is when part of the brain does not get enough blood. Some of the starved tissue is already dead; some is still alive and might survive if blood returns. On a blood-flow scan, both look similarly bad — the scan cannot tell you which is which.

The dataset resolves this. For 149 stroke patients, each has four blood-flow maps (CBF, CBV, MTT, Tmax). A follow-up scan was used by experts to mark exactly which tissue was lost. That expert markup (the "lesion mask") is the answer key.

We test whether blood-flow patterns can predict that damage location on patients the model has never seen.

---

## Why this is a machine-learning study (not just a medical application)

The medical setting is the test bed, not the contribution. The actual study is about something general in ML: **how far simple, interpretable features can go against a learned representation, and precisely where they stop.** That question is not specific to stroke — but stroke gives us a clean, well-labeled setting to answer it with real data and a real answer key.

---

## The five things this project claims

### Novelty

The asymmetry idea itself is not new — left-vs-right comparison is a standard tool in this area, and we do not claim it as our contribution. The novelty is elsewhere:

* **Boundary characterization.** Not "simple features work," but "simple features match a learned model *up to a measurable condition*, then fail, and here is the mechanism." Mapping that boundary precisely is the contribution.  
* **A fair, controlled comparison.** Most such comparisons are done loosely. Doing it rigorously — same patients, same metric, both sides tuned with equal effort — is itself a research contribution at this level.

### Reliability

* Patient-level train/test split: no patient appears in both sets.  
* Both models evaluated on identical held-out patients with the same metric.  
* Results reported with variation across folds/patients, not a single lucky score.  
* Every number regenerable from the repository (pinned dependencies, one-click run).

### Impact (stated in ML terms)

* Shows whether interpretability costs accuracy here — and how much.  
* Produces a model that is cheap to train, fast to run, and readable by a human in seconds.  
* Most importantly: **a model that signals when it should not be trusted.** Knowing a method's limits is a result in itself.

### Benefit (concrete)

* No GPU required; trains and runs on a free Colab instance.  
* Far smaller and faster than the image model — quantify both.  
* Every prediction is explainable: you can point to which feature drove it.  
* The failure analysis tells a user when the simple method is safe to use and when it is not.

### Correctness

This is the foundation everything else rests on:

* No patient leakage between train and test.  
* Identical metric and identical split for both models.  
* The image model is tuned fairly — it is a genuine foil, not a weakened one.  
* Failures are reported, not hidden.  
* Every claim traces to a reproducible number.

---

## Research questions

### 1\. Signal — does the simple idea carry real information?

Do left-vs-right asymmetry features (hemisphere averages, percentiles, ratios, percent of tissue past a threshold) carry enough information to distinguish the stroke-affected side from the healthy one?  
*Checked by:* EDA showing the features separate damaged vs. undamaged tissue.

### 2\. Performance — how close does simple get to complex?

Trained on identical patients and graded on identical held-out patients, how close does the simple-feature model come to a fairly-tuned small image model at locating the damage?  
*Checked by:* a head-to-head comparison on a patient-level split, with the same metric for both.

### 3\. Trade-off — what does simplicity cost, and what does it buy?

What accuracy is given up for a model that is cheaper to run, faster to train, and readable by a human in seconds?  
*Checked by:* reporting accuracy alongside compute cost and interpretability.

### 4\. Failure modes — where and why does the simple idea break?

Which patients does the simple approach get wrong — small lesions, lesions sitting on the midline, strokes that hit both hemispheres — and what does that tell us about the limits of the asymmetry idea?  
*Checked by:* isolating the failure cases and explaining the mechanism.

---

## What counts as success

Any of these is a result worth reporting:

* The simple approach matches the image model — a win for interpretability.  
* The simple approach falls short only in specific, explainable cases — a win for understanding the limits.  
* The simple approach fails broadly — and we can show exactly why.

There is no outcome here that is "no result."

---

## Scope

* Studies **acute localization** of existing stroke damage.  
* Does **not** predict future stroke risk or patient outcome.  
* A machine-learning study, not a clinical tool.

