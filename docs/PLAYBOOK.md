# Playbook slide: before you build a deep network, beat one number

For a small team mapping stroke tissue from CT-perfusion maps, on this dataset (guidance from one study, not a law):

1. **Tune one threshold on one perfusion map first.** That is your bar (Dice about 0.17 here).
2. **Under about 40 labelled scans, stay with the threshold or a simple model.** A small U-Net trailed the threshold up to 20 scans and only tied it at 40.
3. **From about 40 scans, use a simple, explainable model.** It led the threshold by +0.02 to +0.03 Dice and the small U-Net by +0.02 to +0.05 up to 80 scans (mean patient Dice).
4. **At about 120 scans, test the U-Net too.** It ties on the typical patient and, weighted by lesion volume, did as well or better (exploratory).
5. **Whatever you build, route small predicted lesions and lesions in several pieces to a human.** Every model failed there, and a size rule flags it without a second model.

Footer: one public dataset, no external validation, research benchmark, not a clinical tool.
