# EEG Mental Arithmetic: Rest vs Task Classification

End-to-end EEG classification pipeline built on the
[EEG during mental arithmetic tasks](https://physionet.org/content/eegmat/1.0.0/) dataset.
The notebook classifies **Rest vs Mental Arithmetic (Task)** EEG epochs using
subject-independent evaluation.

## Contents

| Path | Description |
|---|---|
| `eeg_analysis.ipynb` | Full pipeline: preprocessing → feature extraction → EDA → LOSO evaluation → hold-out test → robustness checks |
| `dataset/` | Raw EDF recordings (36 subjects × 2 recordings) plus metadata (`subject-info.csv`, `RECORDS`, `SHA256SUMS.txt`, etc.) |

## Dataset

- **Source:** Zyma, Tukaev & Seleznov — *EEG during mental arithmetic tasks* (PhysioNet EEGMAT, v1.0.0)
- **Format:** EDF, 23-channel monopolar EEG (10–20 montage, ear reference)
- **Task:** serial subtraction of a 4-digit minuend and 2-digit subtrahend
- **Recordings:** `SubjectXX_1.edf` = resting-state baseline, `SubjectXX_2.edf` = during task
- **Subjects:** 36 total — Group "G" (24, good task performance) and Group "B" (12, poor performance); see `dataset/subject-info.csv` for age, gender, and count quality

## Pipeline overview

1. **Preprocessing** — band-pass 1–45 Hz + 50 Hz notch, resample to 128 Hz, ICA artifact removal (blinks via Fp1/Fp2 proxies, muscle components), 4 s epochs, AutoReject, balanced Rest/Task epoch counts
2. **Features** — per-channel time-domain, spectral, wavelet (db4), and non-linear features; per-channel-pair correlation, theta–alpha PLV, and coherence
3. **Normalization** — within-subject z-scoring (no labels used)
4. **Evaluation** — leave-one-subject-out (LOSO) cross-validation across a model zoo, a clean hold-out test on Subject01/22/35, SHAP feature importance, and robustness checks excluding artifact-prone channels/bands

## Running

Requires Python with `mne`, `autoreject`, `antropy`, `pywt`, `shap`, `xgboost`, and the usual scientific stack (the notebook installs pinned versions itself).

1. Clone the repo
2. Open `eeg_analysis.ipynb` in Jupyter / VS Code
3. Run all cells — or set `EEG_QUICK=1` for a fast smoke-test run

The notebook locates the dataset automatically (it checks `EEG_DATA_PATH`, then looks for `Subject00_1.edf` in the working directory or `./eeg-during-mental-arithmetic-tasks-1.0.0`). Since `dataset/` is already included, point it there:

```bash
export EEG_DATA_PATH=/path/to/EEG/dataset   # folder containing Subject00_1.edf
```
