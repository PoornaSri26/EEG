---
title: "Subject-Independent EEG Classification of Mental Arithmetic vs. Rest: A Leak-Free Pipeline with Robustness Validation"
author:
  - name: Poorna Sri Nandyala
    affiliation: Department of Computer Science and Engineering, IIIT Sri City
    email: poornasri.n24@iiits.in
    orcid: 0000-0000-0000-0000
date: "November 2026"
geometry: margin=1in
fontsize: 10pt
linestretch: 1.15
bibliography: references.bib
csl: ieee.csl
header-includes:
  - \usepackage{booktabs}
  - \usepackage{multirow}
  - \usepackage{makecell}
  - \usepackage{siunitx}
  - \usepackage{float}
  - \usepackage{hyperref}
  - \hypersetup{colorlinks=true,linkcolor=blue,citecolor=blue,urlcolor=blue}
---

# Abstract

Electroencephalography (EEG)-based mental workload classification holds promise for brain-computer interfaces and cognitive monitoring, yet reported accuracies often collapse under subject-independent evaluation due to data leakage and artifact confounds. This work presents a complete, leak-free pipeline for classifying **Rest versus Mental Arithmetic Task** epochs using the public EEGMAT dataset (36 subjects, 19-channel EEG). We employ rigorous preprocessing (band-pass filtering, ICA-based artifact removal with Fp1/Fp2 as EOG proxies, AutoReject), extract **1,076 features** per 4-second epoch spanning time-domain, spectral, wavelet, non-linear complexity, and connectivity domains, and evaluate 17 models under **leave-one-subject-out (LOSO) cross-validation** with in-fold feature selection and per-subject z-normalization without label access. The best single model (SVM-RBF) achieves **73.7% balanced accuracy** (AUC 0.820), consistent with the only prior LOSO study on this dataset. A fixed hold-out test on three unseen subjects yields **92.0% raw accuracy** (AUC 0.974), though this estimate is noisy due to small sample size. SHAP analysis identifies frontal/temporal complexity and gamma features as top discriminators, but robustness checks excluding artifact-prone channels and frequency bands show performance degradation, indicating non-neural contributions. We conclude that subject-independent classification of this task is feasible but modest (~74%), and that prior claims of 90–99% accuracy stem from subject-dependent evaluation designs.

**Keywords:** EEG, mental arithmetic, mental workload, leave-one-subject-out, cross-subject generalization, artifact robustness, SHAP interpretability, data leakage

---

# 1. Introduction

Electroencephalography (EEG) provides a non-invasive window into brain dynamics with millisecond temporal resolution, making it a candidate for real-time mental workload monitoring and brain-computer interfaces (BCIs). Mental arithmetic—serial subtraction of a multi-digit minuend and subtrahend—is a well-studied cognitive load paradigm that reliably elicits frontal midline theta (Fm$\theta$) synchronization and alpha/beta desynchronization [@berka2007eeg; @chikhi2022eeg; @katahira2018eeg]. However, translating this neuroscience finding into a reliable single-trial classifier for new, unseen individuals remains challenging.

The public **EEGMAT dataset** [@zyma2019electroencephalograms] (PhysioNet: `eegmat/1.0.0`) comprises 36 healthy subjects, each recorded during Rest (eyes-open baseline) and Task (mental arithmetic) conditions using a 23-channel montage (19 scalp EEG + 2 ear references + ECG). Prior work on this dataset reports accuracies of 88–99% using random trial splits or subject-dependent cross-validation [@nath2020eeg; @rambhia2023classification; @aslam2025electroencephalograph]. However, Sibisanu *et al.* [@sibisanu2026high] recently demonstrated that these results collapse under subject-independent evaluation (leave-one-subject-out, LOSO), with Cohen's $\kappa$ falling to 0.19–0.26 and ROC-AUC to 0.66–0.71. This discrepancy arises because random splits allow the model to learn subject-specific idiosyncrasies (skull thickness, electrode impedance, baseline spectral profile) rather than task-relevant neural patterns—a form of **data leakage** [@pusica2025machine].

This paper presents a rigorously leak-free pipeline for Rest vs. Task classification on EEGMAT with the following contributions:

1. A complete preprocessing-to-classification pipeline with **zero data leakage**: in-fold imputation, variance filtering, standardization, and ANOVA feature selection; per-subject z-normalization computed without label access; fixed hold-out subjects never used in any training or hyperparameter choice.
2. A rich feature set (1,076 features/epoch) combining time-domain, spectral (Welch PSD), wavelet (db4), non-linear complexity (entropy, fractal dimension, DFA, LZ), and connectivity (theta-alpha PLV/coherence) descriptors.
3. Comprehensive model comparison (17 models/ensembles) under LOSO, with hyperparameter tuning via grouped 5-fold CV on training subjects only.
4. Interpretability via SHAP on unseen subjects, plus robustness checks excluding artifact-prone channels (Fp1/Fp2) and frequency bands (gamma) to assess neural vs. non-neural contributions.
5. Honest reporting: LOSO balanced accuracy **73.7%** (AUC 0.820); hold-out 92.0% (AUC 0.974) with explicit caveats on small-sample optimism.

---

# 2. Related Work

## 2.1 EEGMAT Dataset Studies

Table 1 summarizes prior work on this dataset. The critical divide is evaluation design: subject-dependent (random splits, within-subject CV) versus subject-independent (LOSO).

| Study | Subjects | Features | Evaluation | Reported Acc. |
|-------|----------|----------|------------|---------------|
| Nath *et al.* [@nath2020eeg] (2020) | 36 | Spectral, ratios | Random split (80/20) | 97.5% (SVM) |
| Rambhia & Sutar [@rambhia2023classification] (2023) | 36 | Wavelet (O1 only) | Within-subject | >95% |
| Aslam [@aslam2025electroencephalograph] (2025) | 36 | EMD + RF-RFE | Within-subject | 98–99% |
| Alhothali & Attar [@alhothali2026validation] (2026) | 36 | Welch PSD + ERP | Subject-level CV | 92% (RF) |
| **Sibisanu *et al.* [@sibisanu2026high] (2026)** | **36** | **Multiple** | **LOSO** | **66–71% AUC** |
| **This work** | **36** | **1,076** | **LOSO + hold-out** | **73.7% bal. acc.** |

**Table 1:** Prior Work on EEGMAT Dataset: Evaluation Design Determines Reported Performance

## 2.2 Mental Workload Neurophysiology

Meta-analysis of 24 studies (723 participants) confirms cognitive workload robustly increases theta (Hedges' $g = 0.68$) and beta ($g = 0.50$), with **frontal theta the best single index** [@chikhi2022eeg]. Frontal-midline theta generators localize to dorsal anterior cingulate / medial prefrontal cortex during mental subtraction [@ishii2014frontal]. Theta-alpha coupling between frontal and parieto-occipital regions predicts arithmetic performance [@dimitriadis2016causal], motivating our connectivity features.

## 2.3 Methodological Foundations

Preprocessing follows MNE-Python [@gramfort2013meg] and PREP pipeline [@bigdely2015prep] principles: average referencing, band-pass 1–45 Hz + 50 Hz notch, resampling to 128 Hz, ICA artifact removal [@makeig1997independent] with AutoReject [@jas2017autoreject]. Non-linear features draw from Hjorth [@hjorth1970eeg] and antropy [@antropy]. SHAP interpretability uses Shapley additive explanations [@lundberg2017unified].

---

# 3. Materials and Methods

## 3.1 Dataset

EEGMAT [@zyma2019electroencephalograms]: 36 subjects (24 "good counters", 12 "bad counters"), ages 18–35. Two recordings per subject: `SubjectXX_1.edf` (Rest, ~3 min) and `SubjectXX_2.edf` (Task, serial subtraction, ~3 min). 23 channels in EDF format: 19 scalp (10-20 system: Fp1, Fp2, F3, F4, Fz, F7, F8, C3, C4, Cz, T3, T4, T5, T6, P3, P4, Pz, O1, O2), A1, A2 (ear references), A2-A1 (reference difference), ECG. Sampling rate 256 Hz (original).

## 3.2 Preprocessing Pipeline

All steps applied identically to Rest and Task recordings per subject to avoid introducing condition-specific bias.

1. **Channel selection:** Match channel names to standard 10-20 montage; retain 19 scalp electrodes; drop A2-A1 and ECG.
2. **Average reference:** Subtract channel mean from each channel ($V_i \leftarrow V_i - \frac{1}{19}\sum_j V_j$), removing common-mode noise.
3. **Filtering:** Zero-phase FIR band-pass 1–45 Hz (transition width 1 Hz, 60 dB stopband) + 50 Hz notch (mains hum). Order 844 (1 Hz) and 36 (45 Hz).
4. **Resampling:** 128 Hz (Nyquist 64 Hz > 45 Hz); reduces compute while preserving all bands of interest.
5. **ICA artifact removal:** FastICA (15 components; rank after average reference = 18). Components correlated with Fp1/Fp2 (EOG proxies) $|r| > 0.7$ flagged as blink; top kurtosis components flagged as muscle. Maximum 4 components removed. One ICA fit per subject on concatenated Rest+Task ensures identical unmixing.
6. **Epoching & AutoReject:** 4-second non-overlapping epochs (512 samples). AutoReject [@jas2017autoreject] learns per-channel amplitude thresholds via cross-validation; repairs or drops bad epochs. Rest epochs truncated to match Task count per subject (balanced classes, chance = 50%).

**Result:** 1,076 clean epochs total (538 Rest, 538 Task), ~30 epochs/subject (~15 per condition).

## 3.3 Feature Extraction

Per epoch, per channel (19), 56 features across four families (1,064 total), plus 12 connectivity features (4 electrode pairs × 3 measures).

### 3.3.1 Time-Domain (11 per channel)
Mean, std, variance, skewness, kurtosis, RMS, peak-to-peak, zero-crossing rate, line length, Hjorth mobility, Hjorth complexity [@hjorth1970eeg].

### 3.3.2 Spectral (22 per channel)
Welch PSD (256-sample Hamming windows, 50% overlap → 0.5 Hz resolution). Absolute power (5 bands: $\delta$ 0.5–4 Hz, $\theta$ 4–8 Hz, $\alpha$ 8–13 Hz, $\beta$ 13–30 Hz, $\gamma$ 30–45 Hz), log power, relative power, spectral edge 95%, peak frequency, spectral entropy, engagement index ($\beta/(\alpha+\theta)$), $\theta/\alpha$, $\theta/\beta$, $\alpha/\theta$ ratios.

### 3.3.3 Wavelet (13 per channel)
Discrete wavelet transform (Daubechies-4, level 5). Energy and relative energy for 6 levels (L0: 0–2 Hz, L1: 2–4 Hz, L2: 4–8 Hz, L3: 8–16 Hz, L4: 16–32 Hz, L5: 32–64 Hz → effectively 32–45 Hz post-filtering) + wavelet entropy.

### 3.3.4 Non-Linear Complexity (10 per channel)
Permutation entropy, SVD entropy, approximate entropy (ApEn), sample entropy (SampEn), Petrosian fractal dimension, Katz fractal dimension, Higuchi fractal dimension (HFD), detrended fluctuation analysis (DFA), zero-crossing count, Lempel-Ziv complexity.

### 3.3.5 Connectivity (12 total)
Pairs: Fp1-Fp2 (inter-hemispheric frontal), F3-F4 (frontal), Fz-Pz (fronto-parietal), F7-F8 (lateral frontal). Measures: Pearson correlation (broadband), Phase-Locking Value (PLV) [@lachaux1999measuring] in 4–13 Hz ($\theta$-$\alpha$), Magnitude-squared coherence in 4–13 Hz.

## 3.4 Normalization

**Per-subject z-scoring:** for each feature, subtract subject's mean (across all epochs, both conditions) and divide by subject's standard deviation. **No labels used.** This removes between-subject baseline differences (skull thickness, electrode contact, arousal) while preserving within-subject Rest/Task contrast. *Caveat:* a deployed system would need both conditions from a new user to compute this normalization.

## 3.5 Classification Pipeline

Scikit-learn `Pipeline` ensuring zero leakage:

1. Median imputation (training-fold median)
2. VarianceThreshold (1e-8)
3. StandardScaler (training-fold fit)
4. SelectKBest (ANOVA F-test, $k=150$, training-fold fit)
5. Classifier

## 3.6 Models Evaluated

- **Linear:** Logistic Regression (L2, $C=0.05$), SVM-Linear ($C=0.03$)
- **Kernel:** SVM-RBF ($C=30$, $\gamma=0.0003$)
- **Single Trees:** DecisionTree (max_depth=5, min_samples_leaf=10), ExtraTree (max_depth=6)
- **Bagging:** Bagging (300 trees, depth 6, 80% samples, 50% features), RandomForest (tuned: 300 trees, leaf 2, $\sqrt{p}$ features), ExtraTrees (tuned: 600 trees, leaf 2, max_features=0.3)
- **Boosting:** AdaBoost (300 estimators, lr=0.5), GradientBoosting (tuned: 400 trees, depth 4, lr=0.1, subsample=0.6), HistGradBoost (tuned: depth 4, lr=0.05, L2=0.1), XGBoost (tuned: 200 trees, depth 3, lr=0.1, colsample=0.5, reg_lambda=3)
- **Ensembles (soft voting):** Top-3 by LOSO AUC, SVM+Boosting, All SVMs, All Trees, All Models

## 3.7 Evaluation Protocol

- **LOSO (primary):** Train on 35 subjects, test on 1; repeat 36 times. Per-subject balanced accuracy and pooled metrics reported. Hyperparameter tuning via **grouped 5-fold CV** (GroupKFold, groups = subjects) on training subjects only, 12 random draws (RandomizedSearchCV).
- **Hold-out (secondary):** Subjects 01, 22, 35 fixed *a priori*, never used in training, tuning, or model selection. Final model (SVM-RBF, all features, chosen by grouped CV on training 33 subjects) evaluated on these 3 subjects' 88 epochs.
- **Metrics:** Balanced accuracy (primary), ROC-AUC, precision, recall, F1, Matthews Correlation Coefficient (MCC), confusion matrix. Per-subject mean ± SD reported.
- **SHAP:** TreeSHAP for tree models; KernelSHAP for SVM. Computed on hold-out subjects using model trained on all 33 training subjects.
- **Robustness checks:** Re-run LOSO excluding (a) Fp1/Fp2 channels, (b) gamma-band features, (c) both.

---

# 4. Results

## 4.1 Exploratory Data Analysis

**Topography:** Task-minus-Rest theta power topomaps show high inter-subject variability; no universal spatial pattern (Figure 1). Subject 0: no region significant after Bonferroni correction ($p \ge 0.78$).

**Group-level frontal theta:** Relative power increases slightly in Task (median +0.0102, paired Wilcoxon $p=0.0105$, $n=36$). Effect size small vs. between-subject spread. Good vs. bad counters: no difference (Mann-Whitney $p=0.54$).

**Feature redundancy:** High correlation among band powers, log-powers, and ratios at same channel (e.g., Fz $\theta$ power vs. log $\theta$ power $r \approx 0.98$; $\theta/\alpha$ vs. $\alpha/\theta$ $r \approx -1$). Justifies regularisation and feature selection.

## 4.2 LOSO Model Comparison

Table 2 summarizes per-subject mean balanced accuracy and AUC for all 17 models.

| Model | Family | Bal. Acc. | AUC | SD (Acc.) |
|-------|--------|-----------|-----|-----------|
| SVM-RBF | Kernel | **0.737** | **0.820** | 0.166 |
| AdaBoost | Boosting | 0.736 | 0.812 | 0.192 |
| XGBoost | Boosting | 0.735 | 0.816 | 0.198 |
| LogReg-L2 | Linear | 0.733 | 0.817 | 0.168 |
| SVM-Linear | Linear | 0.732 | 0.812 | 0.161 |
| Ensemble: SVM+Boosting | Ensemble | 0.738 | 0.822 | 0.194 |
| Ensemble: Top-3 (post-hoc) | Ensemble | 0.743 | 0.826 | 0.177 |
| ExtraTrees | Tree Ensemble | 0.724 | 0.788 | 0.206 |
| GradBoost | Boosting | 0.728 | 0.809 | 0.193 |
| HistGradBoost | Boosting | 0.727 | 0.807 | 0.194 |
| Bagging | Tree Ensemble | 0.708 | 0.782 | 0.206 |
| RandomForest | Tree Ensemble | 0.702 | 0.784 | 0.215 |
| ExtraTree (single) | Tree | 0.654 | 0.715 | 0.165 |
| DecisionTree (single) | Tree | 0.641 | 0.703 | 0.174 |

**Table 2:** LOSO Results: Per-Subject Mean Balanced Accuracy and AUC (SD across 36 subjects)

**Key observations:**

- Single trees severely overfit (64–65%). Ensembling adds 7–9 points.
- Best non-tree (SVM-RBF, 73.7%) matches best boosting (XGBoost 73.5%, AdaBoost 73.6%).
- Top models within ~1% of each other; differences not significant given per-subject SD ~0.17.
- Post-hoc "Top-3" ensemble (74.3%) chosen after seeing LOSO → optimistic.

Grouped CV tuning accuracies (training 33 subjects) were higher (SVM-RBF 74.5%, XGBoost 74.2%, GradBoost 74.7%), confirming modest optimism from tuning on same subject pool.

## 4.3 Hold-Out Test (Subjects 01, 22, 35)

SVM-RBF (all features, chosen by grouped CV on 33 training subjects) on fixed hold-out:

| Subject | Rest Epochs | Task Epochs | Bal. Acc. | AUC |
|---------|-------------|-------------|-----------|-----|
| 01 | 22 | 22 | 82.1% | 0.92 |
| 22 | 22 | 22 | 96.7% | 0.99 |
| 35 | 22 | 22 | 96.7% | 0.99 |
| **Pooled** | **66** | **66** | **92.0%** | **0.974** |
| **Smoothed (w=3)** | - | - | **97.7%** | - |

**Table 3:** Hold-Out Results (3 Unseen Subjects, 88 Epochs Total)

Confusion: 40/44 Rest correct, 41/44 Task correct. All 6 recordings (3 subjects × 2 conditions) classified correctly at recording level. **Caveat:** Only 3 subjects (88 epochs); estimate has high variance. Smoothing (rolling mean over 3 epochs) exploits temporal adjacency, not strict per-epoch performance. LOSO 73.7% is the reliable generalization estimate.

## 4.4 SHAP Feature Importance (Hold-Out Subjects)

KernelSHAP on SVM-RBF (hold-out subjects, model trained on 33 training subjects). Top 20 features:

1. `T6_higuchi_fd` (Higuchi fractal dimension, right temporal)
2. `T5_gamma_power` (gamma absolute power, left temporal)
3. `C3_wavelet_energy_L4` (wavelet energy 16–32 Hz, central)
4. `P3_wavelet_energy_L3` (wavelet energy 8–16 Hz, parietal)
5. `Fz_theta_power` (frontal midline theta)
6. `Fp2_gamma_power`, `Fp1_gamma_power` (frontal pole gamma)
7. `T6_katz_fd`, `T6_petrosian_fd` (complexity, right temporal)
8. `C4_beta_power`, `C3_beta_power` (central beta)
9. Connectivity features: none in top 50.

**Channel distribution:** 45% of top 20 from frontal/temporal channels (Fp1, Fp2, Fz, F3, F4, F7, F8, T3, T4, T5, T6) vs. 42% of scalp. **Directionality:** High Higuchi FD at T6 → Task; high gamma at T5/Fp1/Fp2 → Task; high frontal theta → Task (consistent with workload literature [@chikhi2022eeg; @katahira2018eeg]).

**Critical caveat:** Gamma and frontal pole (Fp1/Fp2) features are muscle/blink-prone [@puce2017review]. Top features mix complexity (neural) and gamma (potentially artifactual).

## 4.5 Robustness Checks: Neural vs. Artifact Contributions

LOSO re-run with feature exclusions (SVM-RBF):

| Exclusion | Bal. Acc. | Δ vs. Full |
|-----------|-----------|------------|
| Full (all 1,076 features) | 0.737 | — |
| Exclude Fp1/Fp2 channels (112 feats) | 0.712 | -0.025 |
| Exclude all gamma-band features (95 feats) | 0.701 | -0.036 |
| Exclude Fp1/Fp2 + gamma (187 feats) | 0.678 | -0.059 |

**Table 4:** Robustness: LOSO Balanced Accuracy Under Feature Exclusions

Performance degrades when artifact-prone features removed, confirming non-neural contributions. However, 67.8% remains above chance (50%), indicating genuine neural signal persists.

## 4.6 Per-Subject Variability

Per-subject balanced accuracy (SVM-RBF, LOSO) ranges from ~0.35 to ~0.95 (SD 0.166). Subjects with high artifact burden (visualized via ICA component rejection count) tend toward lower accuracy. No correlation with good/bad counter label ($r = 0.08$, $p = 0.64$).

---

# 5. Discussion

## 5.1 Why LOSO ~74% and Not Higher?

1. **High dimensionality, small $N$:** 1,076 features, ~1,000 epochs, but only 36 independent subjects. Effective sample size for generalization is 36, not 1,076.
2. **Heterogeneous spatial patterns:** No universal theta topography (Figure 1); classifier must integrate weak, person-varying cues.
3. **Non-neural confounds:** Rest and Task are *separate recordings* (Rest first, then Task). Time-on-task, fatigue, and arousal drift are confounded with condition [@berka2007eeg].
4. **Limited montage:** 19 channels, no dedicated EOG/EMG. ICA with Fp1/Fp2 proxies is imperfect [@puce2017review].
5. **Task difficulty variance:** 12/36 subjects are "bad counters"; their Task condition may reflect failure rather than effort.

## 5.2 Why Hold-Out ~92%?

Small test set (3 subjects, 88 epochs) → high variance. SVM-RBF used all features (including gamma/Fp1/Fp2) which boost separability on these specific subjects. Smoothing (w=3) exploits epoch autocorrelation. **Do not cite 92% as generalization performance.**

## 5.3 Comparison with Literature

Our 73.7% LOSO balanced accuracy (AUC 0.820) aligns with Sibisanu *et al.* [@sibisanu2026high] (LOSO AUC 0.66–0.71 on same dataset with simpler features). Prior 90–99% claims [@nath2020eeg; @rambhia2023classification; @aslam2025electroencephalograph] use subject-dependent evaluation and are not comparable. Alhothali & Attar [@alhothali2026validation] (subject-level CV, RF 92%) report same directional band effects (theta↑, alpha↓) but weaker CV.

## 5.4 Limitations

1. **Condition order confound:** Rest always precedes Task; fatigue/drift mimics workload effects.
2. **No dedicated artifact channels:** Fp1/Fp2 as EOG proxies; no EMG. ICA limited by 19 channels.
3. **Per-subject normalization:** Requires both conditions from new user; not deployable for single-condition inference.
4. **Hold-out size:** 3 subjects insufficient for reliable estimation.
5. **Hyperparameter tuning optimism:** Tuned on same 36 subjects evaluated by LOSO.
6. **Binary only:** Rest vs. Task; workload *levels* not addressed.
7. **Demographics:** Young adults only; generalizability unknown.

---

# 6. Conclusion

We presented a leak-free EEG classification pipeline for Mental Arithmetic vs. Rest on the EEGMAT dataset. Under rigorous leave-one-subject-out evaluation, the best model (SVM-RBF) achieves **73.7% balanced accuracy** (AUC 0.820), with boosting ensembles (XGBoost, AdaBoost) statistically indistinguishable. A fixed hold-out test yields 92.0% but is unreliable due to small sample size. SHAP reveals frontal/temporal complexity and gamma features as top discriminators, but robustness checks show 5.9% accuracy drop when artifact-prone features are excluded, confirming mixed neural/non-neural origins. The result is consistent with the only prior LOSO study on this dataset and substantially lower than subject-dependent claims (90–99%), underscoring that **evaluation design—not model cleverness—drives reported performance**. Future work: within-session condition counterbalancing, higher-density EEG with dedicated artifact channels, and workload-level regression.

---

# Acknowledgment

This work was completed as part of the UG3 BTP 4th Evaluation (Group B28SSR04) at IIIT Sri City. Dataset courtesy of Zyma *et al.* via PhysioNet. Computational resources: local workstation. Code and data: https://github.com/PoornaSri26/EEG

---

# References