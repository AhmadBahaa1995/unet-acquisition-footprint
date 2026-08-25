# Mitigating Acquisition Footprints in Time-Lapse Seismic Data

Code, trained weights and reproduction notebooks for:

> Ahmad, A.B., Tsuji, T., Kioka, A., Kimura, T., Xu, S.
> *Mitigating Acquisition Footprints in Time-Lapse Seismic Data: A U-Net Reconstruction
> Approach for Consistent Monitoring.* Earth Systems and Environment (under review).

A U-Net is trained to map sparse, irregularly acquired monitor surveys onto a dense
baseline grid, so that 4D differences reflect subsurface change rather than acquisition
geometry. The distinguishing element is upstream of the network: training pairs are built
by perturbing source coordinates in the continuous spatial domain **before** solving the
wave equation, rather than by decimating traces from a precomputed dense grid. The network
therefore learns to invert real wavefield distortion (NMO stretch, wavelet interference,
fold loss) instead of filling digital gaps.

---

## Repository layout

```
.
├── notebooks/
│   ├── 01_main_workflow.ipynb            End-to-end: modeling → preprocessing → training → evaluation
│   ├── 02_ablation_data_generation.ipynb Physics-based scattering vs. post-hoc decimation (3×3 cross-test)
│   ├── 03_ablation_capacity.ipynb        16 / 32 / 64 / 128 base-filter comparison
│   ├── 04_velocity_sensitivity.ipynb     NMO velocity error of 0, ±5, ±10 percent
│   ├── 05_perturbation_scale.ipynb       Source mispositioning from field-realistic to stress-test scale
│   └── 06_conventional_baselines.ipynb   U-Net vs. F-K filtering and horizontal regularization
├── models/
│   └── unet_32filters.keras              Trained weights for the reported model
├── data/velocity_models/                 Input velocity models (see Data below)
├── outputs/                              Generated stacks, figures and logs (not tracked)
├── requirements.txt
└── LICENSE
```

---

## Installation

Python 3.11, CUDA 12.4. A GPU is required for forward modeling and training; inference
with the released weights runs on CPU.

```bash
git clone https://github.com/<ORG>/<REPO>.git
cd <REPO>
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
```

The pipeline deliberately spans two frameworks: PyTorch and Deepwave for wave propagation
and pre-processing, TensorFlow/Keras for the network. Both will try to claim GPU memory on
import. If you hit an out-of-memory error, restart the kernel between the modeling stage
(notebook 01, sections 1 to 3) and the training stage (sections 4 onward), reloading the
cached stacks from `outputs/stacked/`.

---

## Data

The three CCS time-lapse velocity models (`baseline.bin`, `monitoring_stage1.bin`,
`monitoring_stage2.bin`) go in `data/velocity_models/`. They are raw `float32`, shaped
`601 × 501` (nx × nz) at 2 m grid spacing. Monitoring stages carry a CO₂ plume introduced
as a Gaussian-smoothed perturbation of −25 percent Vp, +2 percent Vs and −5 percent density
over the injection interval.

The Marmousi2 model used for the generalization test is available from the
[Allied Geophysical Laboratories](https://www.agl.uh.edu/downloads/downloads).

---

## Reproducing the reported results

Run `notebooks/01_main_workflow.ipynb` top to bottom. Section 5 retrains from scratch;
skip it and run section 6 to load the released weights instead.

The reported model is trained on the **monitoring stage 1** pair (`stack_mon1_bad` as
input, `stack_mon1_good` as target). Key configuration:

| Parameter | Value |
|---|---|
| Grid spacing `dx` | 2 m |
| Time step `dt` | 0.004 s |
| Samples `nt` | 300 |
| Ricker dominant frequency | 25 Hz |
| Receivers per shot | 300 |
| Shots, dense target | 100, no perturbation |
| Shots, degraded input | see notebook 01, section 2 |
| Patch size | 128 × 128 |
| Sliding-window step | 5 |
| Train / validation / test | 70 / 21 / 9 percent, `random_state = 42` |
| Base filters | 32 |
| Loss | Mean absolute error |
| Optimizer | Adam, learning rate tuned by Hyperband, gradient clipping 1.0 |
| Batch size | 32 |
| Hardware | NVIDIA RTX A6000, 47 GB |

Online augmentation (horizontal flip, small rotation, small translation, and Gaussian
noise on the input only) is applied per batch, so no two epochs see identical patches.

### Random seeds

`GLOBAL_SEED = 42` is set at the top of every notebook, covering Python's `random`, NumPy,
PyTorch (including CUDA) and TensorFlow. This seeds the stochastic source scattering, the
train/validation/test split and the augmentation stream, so the reported training data and
splits regenerate exactly.

One deliberate exception: the generalization checks re-run forward modeling with a fresh
random source distribution, which is the point of the test. Those cells are marked in the
notebooks. Re-running them gives a new realization each time and metrics will move by a
percentage point or two.

### Metrics

Three metrics are used, and values are not comparable across evaluation contexts:

- **NRMS** (Kragh and Christie, 2002), computed globally over the full stacked section with
  no sub-windowing. Lower is better; below 10 percent is conventionally considered reliable
  for 4D interpretation.
- **SSIM**, on the reconstructed section or on the time-lapse difference map depending on
  context.
- **Reconstruction accuracy**, the percentage of samples within an absolute tolerance of
  0.01 of the target.

Each notebook prints which context it is reporting.

---

## Citation

```bibtex
@article{ahmad2026footprint,
  title  = {Mitigating Acquisition Footprints in Time-Lapse Seismic Data:
            A U-Net Reconstruction Approach for Consistent Monitoring},
  author = {Ahmad, Ahmad Bahaa and Tsuji, Takeshi and Kioka, Arata and
            Kimura, Toshinori and Xu, Shibo},
  journal = {Earth Systems and Environment},
  year   = {2026},
  note   = {Under review}
}
```

Archived release: [DOI to be added on acceptance]

## License

MIT. See [LICENSE](LICENSE).

## Acknowledgements

Built on [Deepwave](https://github.com/ar4/deepwave) (Richardson, 2020) for wave
propagation. Supported by the Ministry of the Environment of Japan
("Environmentally conscious CCUS integrated experimental site and supply chain
construction") and JSPS KAKENHI JP24H00440.

## Contact

Ahmad B. Ahmad, The University of Tokyo, `Ahmadbahaa@g.ecc.u-tokyo.ac.jp`
