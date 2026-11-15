# ds003876 — iEEG Biomarker Pipeline for Resection Zone Mapping

A capstone project implementing a reproducible intracranial EEG (iEEG) biomarker extraction pipeline on the [OpenNeuro ds003876](https://openneuro.org/datasets/ds003876) dataset.

## Overview

This pipeline processes interictal iEEG recordings from drug-resistant epilepsy patients to extract quantitative biomarkers for identifying the **resection zone (RZ)**. The goal is to compare channel-level spectral, aperiodic, and connectivity features between RZ and non-RZ electrodes.

### Dataset

- **Source:** [OpenNeuro ds003876](https://openneuro.org/datasets/ds003876)
- **Cohort:** 23 subjects, 38 recordings (interictal, awake + asleep)
- **Modality:** Intracranial EEG (sEEG / ECoG)
- **Institutions:** NIH, JHU, UMF, PY

### Pipeline Stages

| Step | Description |
|------|-------------|
| **1. Data discovery** | Locate EDF recordings and BIDS channel TSVs for the 23-subject cohort |
| **2. Bad channel removal + 60 Hz notch** | Remove channels marked "bad" in BIDS TSVs; notch filter at 60 Hz |
| **3. Common-average reference (CAR)** | Re-reference all channels to the common average |
| **4. Resample to 500 Hz** | Standardize sampling rate across all recordings |
| **5. Bandpass filter (0.5–150 Hz)** | Remove drift and high-frequency noise |
| **6. 10-second windowing** | Non-overlapping 10 s epochs (1,401 windows total) |
| **8. Spectral features** | Band powers (δ, θ, α, β, low-γ, high-γ), spectral entropy, line length, variance, kurtosis |
| **9. Aperiodic features** | 1/f exponent and offset via `specparam` (parallel with joblib) |
| **10. Coherence connectivity** | Pairwise coherence matrices per recording per band (parallel) |

### Features Extracted

**Per channel, per 10 s window:**
- **6 band powers:** delta (1–4 Hz), theta (4–8 Hz), alpha (8–13 Hz), beta (13–30 Hz), low gamma (30–80 Hz), high gamma (80–150 Hz)
- **Spectral entropy** (1–150 Hz)
- **Line length, variance, kurtosis**
- **Aperiodic offset and exponent** (specparam / FOOOF)

**Per recording:**
- **Coherence matrices** (channels × channels × 6 bands) for each window

## Project Structure

```
.
├── README.md
├── LICENSE
├── requirements.txt
├── .gitignore
└── ds003876_capstone.ipynb     # Full pipeline notebook
```

## Requirements

- Python ≥ 3.9
- See [requirements.txt](requirements.txt) for pinned dependencies

```bash
pip install -r requirements.txt
```

## Usage

1. **Download the dataset** from [OpenNeuro ds003876](https://openneuro.org/datasets/ds003876) into a `ds003876/` directory.
2. **Update paths** in the notebook — change `/workspace/ds003876` and `/workspace/ds003876_processed` to your local paths.
3. **Run the notebook** end-to-end:
   ```bash
   jupyter notebook ds003876_capstone.ipynb
   ```

## Output

The pipeline produces:
- `spectral_features/spectral_features.csv` — 141,121 rows (channel × window) with 14 feature columns
- `aperiodic_features/aperiodic_features.csv` — 141,121 rows with aperiodic offset and exponent
- `connectivity/coherence/*.npy` — Per-recording coherence matrices (windows × channels × channels × 6 bands)

## Future Work

- High-frequency oscillation (HFO) detection
- Neural fragility computation
- Learned graph-based connectivity methods
- RZ vs. non-RZ statistical comparison and classification

## License

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.

## Acknowledgments

- Dataset: [OpenNeuro ds003876](https://openneuro.org/datasets/ds003876)
- Tools: [MNE-Python](https://mne.tools), [specparam](https://github.com/fooof-tools/fooof), [SciPy](https://scipy.org), [joblib](https://joblib.readthedocs.io)
