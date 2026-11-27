# EEG Mental Arithmetic: Rest vs Task Classification

**UG3 — 4th Evaluation Submission**

| | |
|---|---|
| Name | Poorna Sri Nandyala |
| Group ID | B28SSR04 |
| Roll Number | S20240010160 |
| Email | poornasri.n24@iiits.in |
| Program | CSE, UG3 |

End-to-end EEG classification pipeline built on the
[EEG during mental arithmetic tasks](https://physionet.org/content/eegmat/1.0.0/) dataset
(Zyma, Tukaev & Seleznov). The notebook classifies **Rest vs Mental Arithmetic (Task)**
EEG epochs using subject-independent (leave-one-subject-out) evaluation.

## Repository structure

| Path | Contents |
|---|---|
| `slides/` | Presentation slides (`EEG_Mental_Arithmetic_Presentation.pptx`) |
| `report/` | Project report — slide-by-slide script and deep analysis (`EEG_Project_Report.md`), plus a 10-page summary PDF (`EEG_Summary_Report.pdf`) |
| `references/` | Related work and citations (`RELATED_WORKS.md`, `references.bib`) — dataset paper, prior EEGMAT studies, workload literature, and methods references |
| `code/` | Analysis notebook (`eeg_analysis.ipynb`) |
| `dataset/` | Raw EDF recordings — 36 subjects × 2 recordings (Rest `_1`, Task `_2`) + metadata |
| `results/` | Key output figures: topomaps, SHAP importance, ROC/confusion matrices |

## Dataset

- **Format:** EDF, 23-channel monopolar EEG (10–20 montage, ear reference)
- **Task:** serial subtraction of a 4-digit minuend and 2-digit subtrahend
- **Subjects:** 36 total — Group "G" (24, good task performance) and Group "B" (12, poor performance); see `dataset/subject-info.csv`

## Pipeline overview

1. **Preprocessing** — band-pass 1–45 Hz + 50 Hz notch, resample to 128 Hz, ICA artifact removal (blinks via Fp1/Fp2 proxies, muscle components), 4 s epochs, AutoReject, balanced Rest/Task epoch counts
2. **Features** — per-channel time-domain, spectral, wavelet (db4), and non-linear features; per-channel-pair correlation, theta–alpha PLV, and coherence
3. **Normalization** — within-subject z-scoring (no labels used)
4. **Evaluation** — leave-one-subject-out (LOSO) cross-validation across a model zoo, clean hold-out test on Subject01/22/35, SHAP feature importance, robustness checks excluding artifact-prone channels/bands

## Running the code

Requires Python with `mne`, `autoreject`, `antropy`, `pywt`, `shap`, `xgboost`, and the usual scientific stack (the notebook installs pinned versions itself).

1. Clone the repo
2. Open `code/eeg_analysis.ipynb` in Jupyter / VS Code
3. Run all cells — or set `EEG_QUICK=1` for a fast smoke-test run

The notebook locates the dataset automatically (it checks `EEG_DATA_PATH`, then looks for `Subject00_1.edf` in the working directory or `./eeg-during-mental-arithmetic-tasks-1.0.0`). Since `dataset/` is included in this repo, either copy it next to the notebook or point to it:

```bash
export EEG_DATA_PATH=/path/to/EEG/dataset   # folder containing Subject00_1.edf
```
