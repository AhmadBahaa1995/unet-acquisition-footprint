# Before you publish this repository

Delete this file once you have worked through it. It is a checklist, not part of the
release.

## 1. Fill in the placeholders

- `README.md`: replace `<ORG>/<REPO>` in the clone URL (two places).
- `README.md`: add the Zenodo DOI once minted.
- `README.md`, section "Reproducing": the degraded-input shot count and maximum source
  perturbation are left pointing at the notebook rather than stated, because the notebook,
  the ablation code and the manuscript currently disagree (10 vs. 13 vs. 15 shots; 130 vs.
  150 vs. 200 m). Settle on the values that produced the released weights, then state them
  in the table and make Table 1 of the manuscript match.
- `LICENSE`: confirm MIT is acceptable to all co-authors and to ENEOS Xplora before
  publishing. If the company needs different terms, CC-BY-4.0 is the usual alternative for
  research code.

## 2. Add the files that are not in this bundle

- `data/velocity_models/*.bin` (three files). If redistribution is permitted, commit them;
  each is about 1.2 MB, well within limits. If not, replace the Data section with
  instructions for regenerating them.
- `models/unet_32filters.keras`. Uncomment the `models/*.keras` line in `.gitignore` only
  if you would rather attach it to the GitHub release than commit it. Under about 100 MB
  commits fine; above that use Git LFS.

## 3. Run each notebook once in a clean environment

The cells were extracted from a working session, so some rely on variables defined earlier
in the original notebook. Running each top to bottom in a fresh kernel is the only way to
find those. Fix any `NameError` before publishing, since a reviewer who hits one on the
first cell will not try the second.

## 4. Verify the seed claim actually holds

The response letter to Comment 1.27 states that the seeds are documented and the training
data regenerates exactly. Notebook 01 now sets `GLOBAL_SEED = 42` across all four
libraries, but confirm by regenerating a stack and comparing it against the cached one
before you make that claim to the editor.

## 5. Mint the DOI

Push to GitHub, connect the repository at zenodo.org/account/settings/github, then cut a
release (`v1.0.0`). Zenodo mints the DOI automatically. Put that DOI in the manuscript's
Data Availability section and in the README.

## What was removed from the original notebook

For your own reference, in case something turns out to be needed:

- Cells 91 to 119: a separate SEG-Y field-data super-resolution experiment
  (1024x1024 patches, `velan` import, `stream` objects). Never executed, unrelated to this
  paper.
- Cells 1 to 25: an earlier iteration of the forward-modeling and stacking pipeline,
  superseded by cells 26 to 33.
- Cells 34 to 45: a shot-spacing exploration, superseded by the ablation notebooks.
- Cells 66, 69, 70, 74 to 76, 78, 79, 83: duplicate or partial versions of the evaluation
  code kept in notebook 01.
- Cell 52: an earlier patch-preparation cell that loaded `.npy` files and then immediately
  overwrote the arrays from the in-memory dictionary. Replaced by cell 53, which does the
  same thing without the dead load.
