# BTP 4th Evaluation — Submission Checklist & Package

**Group ID:** B28SSR04  
**Name:** Poorna Sri Nandyala  
**Roll:** S20240010160  
**Email:** poornasri.n24@iiits.in  
**Program:** CSE, UG3  
**Evaluation Date:** 27-11-2026 (Friday)  
**Time Slot:** 2:00 – 2:30 PM  
**Meet Link:** https://meet.google.com/rdz-zcdh-jff

---

## 📦 Required Submission Package (Shared Drive)

Create a folder named **`B28SSR04`** in the shared drive with the following structure:

```
B28SSR04/
├── slides/
│   └── EEG_Mental_Arithmetic_Presentation.pptx     (15-20 slides, 20 min talk)
├── report/
│   ├── EEG_IEEE_Report.pdf                         (Main IEEE-style report)
│   ├── EEG_IEEE_Report.md                          (Source markdown)
│   ├── EEG_Project_Report.md                       (Slide-by-slide deep script)
│   └── EEG_Summary_Report.pdf                      (10-page summary, already in repo)
├── code/
│   └── eeg_analysis.ipynb                          (Full analysis notebook)
├── dataset/
│   ├── Subject00_1.edf ... Subject35_2.edf         (All 72 EDF files)
│   └── subject-info.csv                            (Metadata: group, counter quality)
├── results/
│   ├── topomap_theta_task_minus_rest.png
│   ├── shap_summary_holdout.png
│   ├── roc_loso_svm_rbf.png
│   ├── confusion_loso_svm_rbf.png
│   ├── robustness_checks.png
│   └── feature_correlation_heatmap_fz.png
├── references/
│   ├── RELATED_WORKS.md                            (Literature review with mappings)
│   └── references.bib                              (BibTeX for IEEE report)
└── README.md                                       (This file / repo root README)
```

---

## 📄 Report: What to Submit (Best IEEE Style)

### Primary Report: `EEG_IEEE_Report.pdf`
**Generated from:** `EEG_IEEE_Report.md` (this repo: `report/EEG_IEEE_Report.md`)  
**Format:** IEEE Conference two-column, 10pt, ~6-8 pages  
**Contents:**
- Abstract (150 words)
- 1. Introduction (motivation, leakage problem, contributions)
- 2. Related Work (Table 1: prior EEGMAT studies vs. evaluation design)
- 3. Materials & Methods (Dataset, Preprocessing, Features, Normalization, Pipeline, Models, Evaluation)
- 4. Results (EDA, LOSO model comparison Table 2, Hold-out Table 3, SHAP, Robustness Table 4)
- 5. Discussion (Why 74%, why 92%, literature alignment, limitations)
- 6. Conclusion
- References (27 entries, IEEE style)

### Supplementary Report: `EEG_Project_Report.md`
**Already in repo:** `report/EEG_Project_Report.md` (118 KB)  
**Purpose:** Slide-by-slide deep script for presentation prep; contains every number, statistic, caveat, and likely Q&A.

### Summary Report: `EEG_Summary_Report.pdf`
**Already in repo:** `report/EEG_Summary_Report.pdf` (8.7 MB, 10 pages)  
**Purpose:** Quick reference for evaluators.

---

## 🎯 Slides: What to Present (15-20 Slides, 20 Minutes)

**Current deck:** 10 slides (`slides/EEG_Mental_Arithmetic_Presentation.pptx`) — **too few**.  
**Target:** 15-20 slides (~1 min/slide). Expand by splitting dense slides:

| Current Slide | Split Into | Minutes |
|---------------|------------|---------|
| 1: Title + 4 metrics | 1. Title (Group ID, name, roll) + 2. Problem & Headline Results | 1.5 |
| 2: What/Why/How + Pipeline | 3. Problem Statement & Motivation + 4. Pipeline Overview | 2 |
| 3: Preprocessing + Features | 5. Preprocessing (6 steps) + 6. Feature Families (table) + 7. Normalization | 3 |
| 4: Topomaps (36 subjects) | 8. EDA: Theta Topography + 9. Subject 0 Case Study (stats) | 2 |
| 5: Redundancy + Frontal Theta | 10. Feature Redundancy (heatmap) + 11. Group Frontal Theta (box plot + stats) | 2 |
| 6: Model Comparison | 12. LOSO Model Comparison (bar chart) + 13. Tree vs Non-Tree Analysis | 2.5 |
| 7: ROC + Confusion | 14. ROC Curves (per-subject spread) + 15. Confusion Matrix | 1.5 |
| 8: SHAP | 16. SHAP Feature Importance (beeswarm) + 17. Channel Distribution & Directionality | 2 |
| 9: Hold-Out | 18. Hold-Out Results (table + per-subject) + 19. Caveats (small N, smoothing) | 2 |
| 10: Verdict + Limitations | 20. Why 74%? + 21. Limitations (7 items) + 22. Conclusion & Future Work | 2.5 |

**Total: ~22 slides → trim to 18 by combining 8+9, 10+11, 20+21**

### Slide Design Rules
- Group ID **B28SSR04** on title slide (mandatory)
- Minimal text; one key message per slide
- All figures from `results/` folder (high-res PNG)
- Tables: max 6 rows, highlight best in bold
- No animations; PDF export for safety

---

## 💻 Code: What to Include

**Primary:** `code/eeg_analysis.ipynb` (complete, runnable)  
**Requirements:**
- Runs end-to-end without errors
- `EEG_QUICK=1` environment variable for smoke test (< 5 min)
- Pinned dependencies in first cell (`%pip install mne==1.6 autoreject==0.2.2 antropy==0.1.5 pywt==1.4.1 shap==0.42.1 xgboost==2.0.3`)
- Data path auto-detection: checks `EEG_DATA_PATH` env var, then `./dataset`, then `./eeg-during-mental-arithmetic-tasks-1.0.0`

**Verify before submission:**
```bash
cd code
EEG_QUICK=1 jupyter nbconvert --execute --to notebook --inplace eeg_analysis.ipynb
```

---

## 📊 Dataset: What to Include

**All 72 EDF files** (36 subjects × 2 recordings) in `dataset/`  
**Metadata:** `dataset/subject-info.csv` with columns:
- `subject_id` (00-35)
- `group` (G/B)
- `counter_quality` (1/0)

**Note:** Raw data is ~500 MB. If shared drive has size limits, include a `DATA_ACCESS.md` with:
- PhysioNet link: https://physionet.org/content/eegmat/1.0.0/
- Download script: `wget -r -N -c -np --user <user> --ask-password https://physionet.org/files/eegmat/1.0.0/`

---

## 🔗 GitHub Repository (Required)

**URL:** https://github.com/PoornaSri26/EEG  
**Must contain:** All folders above (slides, report, code, dataset/, results/, references/)  
**README.md** at root with:
- Project title, group ID, author
- One-line problem statement
- Headline result (73.7% LOSO)
- Quick start instructions
- Link to report PDF

---

## ✅ Pre-Submission Verification Checklist

| Item | Done? |
|------|-------|
| [ ] Folder `B28SSR04/` created in shared drive | |
| [ ] `slides/EEG_Mental_Arithmetic_Presentation.pptx` (15-20 slides) | |
| [ ] `report/EEG_IEEE_Report.pdf` (IEEE two-column, compiles) | |
| [ ] `report/EEG_Project_Report.md` (deep script) | |
| [ ] `report/EEG_Summary_Report.pdf` (10-page summary) | |
| [ ] `code/eeg_analysis.ipynb` (runs end-to-end, `EEG_QUICK=1` works) | |
| [ ] `dataset/` (all 72 EDF + subject-info.csv) | |
| [ ] `results/` (all 6+ key figures as PNG) | |
| [ ] `references/RELATED_WORKS.md` + `references.bib` | |
| [ ] GitHub repo synced with all above | |
| [ ] Presentation rehearsed to 20 min + 10 min Q&A | |
| [ ] Title slide shows **Group ID: B28SSR04** | |

---

## 🎤 Presentation Tips (20 min talk + 10 min Q&A)

### Do
- **Start with the headline:** "73.7% LOSO balanced accuracy — honest, leak-free, consistent with prior LOSO study"
- **Show the leakage table (Table 1)** early — it frames why your number is credible
- **Explain LOSO vs random split** in 30 seconds (key evaluator question)
- **Own the limitations** (condition order, no EOG, small hold-out) — gains credibility
- **SHAP + Robustness together:** "Top features include gamma/Fp1/Fp2; when we remove them, accuracy drops 5.9% but stays above chance → mixed neural/artifact signal"

### Anticipated Questions & Answers

| Question | Answer |
|----------|--------|
| "Why not deep learning?" | 36 subjects, ~1,000 epochs — insufficient for DL; features + classical ML is correct tool; also explainable |
| "Is 73.7% good?" | Well above 50% chance; matches only rigorous prior LOSO (Sibisanu 2026: AUC 0.66-0.71); not deployable but scientifically honest |
| "Why hold-out 92% so different?" | Only 3 subjects (88 epochs) — high variance; smoothing exploits temporal autocorrelation; LOSO is the reliable estimate |
| "What does SHAP actually show?" | Frontal/temporal complexity (Higuchi FD) + gamma; but gamma/Fp1/Fp2 are artifact-prone; robustness check confirms |
| "Can this run real-time?" | Feature extraction (entropy, fractal dim) is heavy; would need trimmed feature set; not tested |
| "Why per-subject z-score without labels?" | Removes baseline differences (skull, impedance); uses no label info; caveat: needs both conditions at deploy time |

---

## 📁 File Locations (Your Local Machine)

```
C:\Users\npoor\EEG\                    ← Git repo root (push this)
├── slides\EEG_Mental_Arithmetic_Presentation.pptx
├── report\
│   ├── EEG_IEEE_Report.md            ← NEW: IEEE report source
│   ├── EEG_IEEE_Report.pdf           ← Generate via: pandoc report/EEG_IEEE_Report.md -o report/EEG_IEEE_Report.pdf --pdf-engine=xelatex -V geometry:margin=1in -V fontsize=10pt --citeproc --bibliography=report/references.bib --csl=ieee.csl
│   ├── EEG_Project_Report.md         ← Existing deep script
│   └── EEG_Summary_Report.pdf        ← Existing 10-page summary
├── code\eeg_analysis.ipynb
├── dataset\ (72 EDF + subject-info.csv)
├── results\ (6+ PNG figures)
└── references\
    ├── RELATED_WORKS.md
    └── references.bib
```

---

## 🚀 Quick PDF Generation (if Pandoc + LaTeX installed)

```bash
cd C:\Users\npoor\EEG\report
# Option 1: Pandoc (needs LaTeX: MiKTeX/TeX Live)
pandoc EEG_IEEE_Report.md -o EEG_IEEE_Report.pdf --pdf-engine=xelatex -V geometry:margin=1in -V fontsize:10pt --citeproc --bibliography=references.bib --csl=https://raw.githubusercontent.com/citation-style-language/styles/master/ieee.csl

# Option 2: VS Code (Markdown PDF extension)
# Open EEG_IEEE_Report.md → Ctrl+Shift+P → "Markdown PDF: Export (pdf)"

# Option 3: Online (HackMD, Overleaf) — paste markdown, export PDF
```

---

**Final Note:** The IEEE report (`EEG_IEEE_Report.md/.pdf`) is your *primary submission artifact*. The deep script (`EEG_Project_Report.md`) is your *presentation backup*. The summary (`EEG_Summary_Report.pdf`) is the *evaluator quick-read*. All three should be in `report/`.

**Good luck!** 🎓