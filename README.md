# Lopsided

**Can simple left-vs-right blood-flow asymmetry locate acute stroke damage about as well as a small image model — while staying fully interpretable?**

This is a machine-learning study, not a clinical tool. It is not intended for
diagnosis or any medical use.

---

## The question

Acute ischemic stroke starves part of the brain of blood. Perfusion CT produces
four blood-flow maps (CBF, CBV, MTT, Tmax). Because a stroke usually hits one
side, the affected hemisphere looks different from the healthy one.

We test whether **hand-measured left-vs-right asymmetry features** fed to a
simple classifier can locate the stroke-affected tissue, and we benchmark that
against a **small image model** trained on the same scans and the same
patient-level split.

## Research questions

1. **Signal** — do left-vs-right asymmetry features carry enough information to
   locate stroke damage?
2. **Performance** — how close does the simple feature model get to a small
   image model on identical held-out patients?
3. **Trade-off** — what is the cost in accuracy for the gain in
   interpretability and compute?
4. **Failure modes** — where does the simple approach break (small lesions,
   near-midline lesions, bilateral strokes), and why?

## Why it matters

If a simple, explainable comparison performs close to a black-box network, the
black box buys little here — and a human can verify the reasoning in seconds.
If it does not, we show exactly where and why it fails. Either outcome is a
result.

---

## Data

- **Dataset:** ISLES'24 (Ischemic Stroke Lesion Segmentation Challenge),
  training release, 149 subjects.
- **Source:** public mirror on Hugging Face
  (`hugging-science/isles24-stroke`), synced to the Zenodo v7 archive.
- Each patient ships precomputed perfusion maps (CBF, CBV, MTT, Tmax) plus an
  expert lesion mask (ground truth).

**The data is NOT included in this repository.** It is downloaded at runtime
from the public source. Do not commit scans, derived volumes, or any personal
file paths.

## Reproducibility

- Runs end-to-end in Google Colab (free tier).
- Loads one patient at a time to stay within ~12 GB RAM.
- Pinned dependencies in `requirements.txt`.
- Patient-level train/test split (no patient appears in both).

## Notebooks

| Notebook | Purpose | Status |
|---|---|---|
| `lopsided_week1.ipynb` | Load data, visualize, extract features for one patient | Done |
| `lopsided_week2.ipynb` | Loop all 149 patients → feature table | Planned |
| `lopsided_week3.ipynb` | Two-model showdown + failure analysis | Planned |

## Quick start

1. Open `lopsided_week1.ipynb` in Google Colab (same account as your Drive).
2. Run cells top to bottom. First run downloads the data once to your Drive.
3. Later sessions skip the download automatically.

## Scope

- This studies **acute detection/localization** of existing stroke damage.
- It does **not** predict future stroke risk or patient outcome.

## License

Code released under the MIT License (see `LICENSE`). The ISLES'24 data is
governed by its own license at the original source and is not redistributed
here.
