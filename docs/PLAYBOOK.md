# Playbook slide: before you build a deep network, beat one number

For a small team mapping stroke tissue from CT-perfusion maps, on this dataset (guidance from one study, not a law):

1. **Tune one threshold on one feature first.** That is your bar (Dice about 0.17 here).
2. **Under about 40 labelled scans, stay with the threshold or a simple model.** A small U-Net trailed the threshold up to 20 scans and only tied it at 40.
3. **From about 40 scans, use a simple, explainable model.** It led the threshold by +0.02 to +0.03 Dice and the small U-Net by +0.02 to +0.05 up to 80 scans (mean patient Dice).
4. **At about 120 scans (we tested up to 118), test the U-Net too.** It ties on mean patient Dice and, weighted by lesion volume, did as well or better (exploratory).
5. **Expect every model to do poorly on small lesions and on lesions in several pieces.** A simple size rule flags small predicted lesions without a second model (exploratory; not tested on lesions in several pieces).

Footer: one public dataset, no external validation, research benchmark, not a clinical tool.
