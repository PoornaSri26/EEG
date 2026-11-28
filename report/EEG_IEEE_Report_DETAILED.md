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
  - \usepackage{graphicx}
  - \hypersetup{colorlinks=true,linkcolor=blue,citecolor=blue,urlcolor=blue}
  - \newcommand{\figref}[1]{Figure~\ref{#1}}
  - \newcommand{\tabref}[1]{Table~\ref{#1}}
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

### Why This Table Matters
The 30+ percentage point gap between subject-dependent (97.5%) and subject-independent (66–71% AUC) results on the *same dataset* is not a model failure—it is an **evaluation design artifact**. Random splits leak subject identity because epochs from the same person are highly correlated (intra-subject correlation > 0.8 for spectral features). When the same subject appears in both train and test, the model learns "what Subject 5's EEG looks like" rather than "what mental arithmetic looks like." Our LOSO protocol eliminates this by keeping all epochs of a subject strictly in either train or test.

## 2.2 Mental Workload Neurophysiology

Meta-analysis of 24 studies (723 participants) confirms cognitive workload robustly increases theta (Hedges' $g = 0.68$) and beta ($g = 0.50$), with **frontal theta the best single index** [@chikhi2022eeg]. Frontal-midline theta generators localize to dorsal anterior cingulate / medial prefrontal cortex during mental subtraction [@ishii2014frontal]. Theta-alpha coupling between frontal and parieto-occipital regions predicts arithmetic performance [@dimitriadis2016causal], motivating our connectivity features.

## 2.3 Methodological Foundations

Preprocessing follows MNE-Python [@gramfort2013meg] and PREP pipeline [@bigdely2015prep] principles: average referencing, band-pass 1–45 Hz + 50 Hz notch, resampling to 128 Hz, ICA artifact removal [@makeig1997independent] with AutoReject [@jas2017autoreject]. Non-linear features draw from Hjorth [@hjorth1970eeg] and antropy [@antropy]. SHAP interpretability uses Shapley additive explanations [@lundberg2017unified].

---

# 3. Materials and Methods

## 3.1 Dataset

EEGMAT [@zyma2019electroencephalograms]: 36 subjects (24 "good counters", 12 "bad counters"), ages 18–35. Two recordings per subject: `SubjectXX_1.edf` (Rest, ~3 min) and `SubjectXX_2.edf` (Task, serial subtraction, ~3 min). 23 channels in EDF format: 19 scalp (10-20 system: Fp1, Fp2, F3, F4, Fz, F7, F8, C3, C4, Cz, T3, T4, T5, T6, P3, P4, Pz, O1, O2), A1, A2 (ear references), A2-A1 (reference difference), ECG. Sampling rate 256 Hz (original).

### Why This Dataset?
- **Public, standardized**: PhysioNet hosting ensures reproducibility.
- **Built-in workload manipulation**: Good vs. bad counter labels provide a ground-truth effort gradient.
- **Adequate N for LOSO**: 36 subjects is the minimum for meaningful LOSO (N-1=35 training subjects).
- **19-channel 10-20**: Standard montage enables comparison with literature.

### Why Not Other Datasets?
- **SEED/SEED-IV**: Emotion, not workload; different paradigm.
- **PhysioNet MI**: Motor imagery, not cognitive load.
- **Custom data**: Would prevent direct comparison with prior EEGMAT studies.

## 3.2 Preprocessing Pipeline

All steps applied identically to Rest and Task recordings per subject to avoid introducing condition-specific bias.

### Step 1: Channel Selection (Keep 19, Drop 4)

| Channel | Type | Action | Reason |
|---------|------|--------|--------|
| Fp1, Fp2, F3, F4, Fz, F7, F8, C3, C4, Cz, T3, T4, T5, T6, P3, P4, Pz, O1, O2 | Scalp EEG (10-20) | **Keep** | Standard montage; matched to MNE's `standard_1020` |
| A1, A2 | Ear references | Drop | Not scalp signals; used only for re-referencing |
| A2-A1 | Reference difference | Drop | Linear combination of A1/A2; zero brain signal |
| ECG | Heart | Drop | Non-neural; contaminates average reference |

**Why not keep ECG for artifact regression?** ECG regression assumes clean ECG channel; here ECG is single-lead and noisy. ICA handles cardiac artifacts better (see Step 5).

### Step 2: Average Reference

$$V_i^{ref} = V_i - \frac{1}{19}\sum_{j=1}^{19} V_j$$

**Why average reference over others?**
| Reference | Pros | Cons | Verdict |
|-----------|------|------|---------|
| **Average (this work)** | Unbiased; no single channel dominates; standard for high-density | Assumes uniform coverage (19-ch ok) | ✅ Chosen |
| Single electrode (e.g., Cz) | Simple | Reference noise propagates to all channels | ❌ |
| Linked ears (A1+A2)/2 | Physiological | Requires clean ear electrodes; not always available | ❌ (ears dropped) |
| REST (Reference Electrode Standardization Technique) | Reference-free in theory | Needs high-density (>64 ch) for accurate lead field | ❌ (19 ch insufficient) |

### Step 3: Filtering (1–45 Hz Band-pass + 50 Hz Notch)

| Filter | Parameters | Why This Choice |
|--------|------------|-----------------|
| High-pass | 1 Hz, FIR, zero-phase, transition 1 Hz, 60 dB stopband, order 844 | Removes slow drift (sweat, movement); 1 Hz > 0.5 Hz delta edge preserves delta; ICA requires drift-free data |
| Low-pass | 45 Hz, FIR, zero-phase, transition 5 Hz, order 36 | Matches gamma band edge; original data has 45 Hz hardware filter; avoids aliasing at 128 Hz |
| Notch | 50 Hz, IIR, Q=30 | Mains hum not fully removed by 45 Hz low-pass (transition band leakage); 50 Hz standard for India/Europe |

**Why zero-phase (filtfilt)?** Prevents phase distortion—critical for ERP/time-locked analysis (not used here but preserves waveform shape for ICA).

**Why not 0.1–100 Hz?** 0.1 Hz retains drift that breaks ICA; >45 Hz is mostly muscle (EMG) with little neural signal in scalp EEG.

### Step 4: Resampling to 128 Hz

**Why 128 Hz?**
- Nyquist = 64 Hz > 45 Hz (max band of interest) → no aliasing
- 2× reduction from 256 Hz → 4× faster processing, 4× less memory
- Standard in EEG literature (e.g., BCI Competition IV)

**Why not keep 256 Hz?** No information above 45 Hz; Welch PSD with 256-sample windows at 256 Hz gives 1 Hz resolution (too coarse for delta/theta separation). At 128 Hz with 256-sample windows → 0.5 Hz resolution (better).

### Step 5: ICA Artifact Removal (FastICA, 15 Components)

| Parameter | Value | Justification |
|-----------|-------|---------------|
| Algorithm | FastICA (deflation, log-cosh) | Fast, stable, standard in MNE |
| Components | 15 | Rank after average reference = 18; 15 < 18 avoids over-fitting; captures blinks, saccades, muscle, cardiac |
| EOG proxy | Fp1, Fp2 correlation \|r\| > 0.7 | No dedicated EOG channels; frontal poles best capture blinks |
| Muscle detection | Top 2 kurtosis components (after blink removal) | EMG = high kurtosis (spiky) |
| Max removed | 4 components | Conservative: prevents removing neural signal; typical blink=1, saccade=1, muscle=1–2 |
| Fit data | Concatenated Rest + Task (per subject) | **Critical**: Same unmixing matrix for both conditions → no condition-specific bias |

**Why ICA over regression?** Regression (e.g., EOG channel subtraction) assumes linear mixing and clean reference. ICA separates statistically independent sources—better for overlapping artifacts.

**Why not more components?** With 19 channels, rank=18. 15 components captures major artifacts while leaving neural subspace intact. 18 components often splits neural sources artificially.

**Why not ICA per condition?** Fitting separate ICAs on Rest vs. Task would learn different unmixing matrices, potentially removing different artifacts from each condition → **artificial Rest/Task difference** (data leakage).

### Step 6: Epoching & AutoReject

| Parameter | Value | Reason |
|-----------|-------|--------|
| Epoch length | 4 seconds (512 samples @ 128 Hz) | ≥2 cycles at 0.5 Hz; enough for stable entropy/fractal estimates; ~30 epochs/subject |
| Overlap | None (non-overlapping) | Independence assumption for CV; overlapping inflates N artificially |
| AutoReject | MNE-Autoreject default (CV=5, consensus=0.5) | Data-driven per-channel thresholds; repairs (interpolation) preferred over rejection |
| Class balancing | Rest truncated to match Task count | Equal epochs → chance = 50%; avoids class-weight hack |

**Result:** 1,076 total epochs (538 Rest, 538 Task), ~30 epochs/subject (~15/condition).

**Why not longer epochs (e.g., 10 s)?** Fewer epochs → worse statistical estimates for entropy/fractal features; non-stationarity increases.

**Why not shorter (2 s)?** 2 s = 1 cycle at 0.5 Hz → unreliable delta/theta power; entropy estimates unstable.

---

## 3.3 Feature Extraction

Per epoch, per channel (19), 56 features across four families (1,064 total), plus 12 connectivity features (4 electrode pairs × 3 measures) = **1,076 features/epoch**.

### 3.3.1 Time-Domain (11 per channel)

| Feature | Formula | What It Captures | Why Include |
|---------|---------|------------------|-------------|
| Mean | $\frac{1}{N}\sum x_i$ | DC offset | Baseline shift |
| Std, Variance | $\sqrt{\frac{1}{N-1}\sum(x_i-\bar{x})^2}$ | Signal power | Total activity |
| Skewness | $\frac{\frac{1}{N}\sum(x_i-\bar{x})^3}{\sigma^3}$ | Asymmetry | Blink = positive skew |
| Kurtosis | $\frac{\frac{1}{N}\sum(x_i-\bar{x})^4}{\sigma^4} - 3$ | Tailedness | Muscle = high kurtosis |
| RMS | $\sqrt{\frac{1}{N}\sum x_i^2}$ | Average amplitude | Overall signal strength |
| Peak-to-peak | $\max(x) - \min(x)$ | Dynamic range | Artifact detection |
| Zero-crossing rate | $\frac{1}{N-1}\sum \mathbb{1}[x_i x_{i+1} < 0]$ | Dominant frequency proxy | Fast vs. slow activity |
| Line length | $\sum \|x_{i+1} - x_i\|$ | Signal complexity | Erratic = high |
| Hjorth mobility | $\sqrt{\frac{\text{var}(\dot{x})}{\text{var}(x)}}$ | Mean frequency | Spectral centroid (time-domain) |
| Hjorth complexity | $\frac{\text{mobility}(\dot{x})}{\text{mobility}(x)}$ | Spectral bandwidth | Complexity of frequency content |

**Why these 11?** Hjorth parameters [@hjorth1970eeg] are the classical time-domain EEG descriptors—proven in workload literature. Others are standard signal statistics. All computable without spectral estimation.

**Why not more (e.g., autoregressive coefficients)?** AR coefficients require model order selection; unstable for short epochs. Our 11 are robust and interpretable.

### 3.3.2 Spectral / Frequency-Domain (22 per channel)

**Method:** Welch PSD — 256-sample Hamming windows (2 s), 50% overlap → 0.5 Hz frequency resolution.

| Feature Group | Count | Bands / Measures | Why |
|---------------|-------|------------------|-----|
| Absolute band power | 5 | $\delta$ (0.5–4), $\theta$ (4–8), $\alpha$ (8–13), $\beta$ (13–30), $\gamma$ (30–45) Hz | Core rhythms; workload = $\theta\uparrow$, $\alpha\downarrow$ |
| Log band power | 5 | $\log(\text{power})$ | Power is log-normal; log normalizes for ML |
| Relative band power | 5 | $\text{band power} / \text{total power}$ | Normalizes inter-subject amplitude differences |
| Spectral edge 95% | 1 | Freq where 95% cum. power reached | "How fast" summary |
| Peak frequency | 1 | $\arg\max_f \text{PSD}(f)$ | Dominant rhythm |
| Spectral entropy | 1 | $-\sum p(f)\log p(f)$ (normalized) | Flatness of spectrum |
| Engagement index | 1 | $\beta / (\alpha + \theta)$ | Classic workload index [@berka2007eeg] |
| Band ratios | 3 | $\theta/\alpha$, $\theta/\beta$, $\alpha/\theta$ | Alternative workload indices |

**Why Welch over multitaper?** Multitaper gives lower variance but requires parameter choice (time-bandwidth product). Welch is standard, deterministic, and 0.5 Hz resolution is adequate.

**Why 0.5–45 Hz bands?** Matches filter. Delta starts at 0.5 Hz (not 0) because 1 Hz high-pass attenuates <1 Hz. Gamma ends at 45 Hz (filter edge).

**Why include gamma (30–45 Hz)?** Controversial: may contain high-frequency neural activity [@ishii2014frontal] but also muscle. We include it **but test robustness** (Section 4.5).

### 3.3.3 Wavelet (13 per channel)

**Method:** Discrete Wavelet Transform (DWT) — Daubechies-4 (db4), 5 decomposition levels.

| Level | Freq Range (approx.) | Corresponds To | Features |
|-------|---------------------|----------------|----------|
| L0 (approx) | 0–2 Hz | Delta + sub-delta | Energy, rel. energy |
| L1 (detail) | 2–4 Hz | Low delta | Energy, rel. energy |
| L2 (detail) | 4–8 Hz | Theta | Energy, rel. energy |
| L3 (detail) | 8–16 Hz | Alpha + low beta | Energy, rel. energy |
| L4 (detail) | 16–32 Hz | Beta | Energy, rel. energy |
| L5 (detail) | 32–64 Hz → 32–45 Hz* | Gamma (filtered) | Energy, rel. energy |
| — | — | — | Wavelet entropy (1) |

*After 45 Hz low-pass, L5 effectively 32–45 Hz.

**Why db4 wavelet?** Compact support (length 8), 2 vanishing moments—good match for EEG transient shapes. Standard in EEG literature (e.g., [@rambhia2023classification]).

**Why DWT over CWT (continuous)?** DWT is faster (O(N) vs O(N²)), non-redundant (critical sampling), and energy-preserving (Parseval's theorem). CWT over-completes representation.

**Why wavelet entropy?** Captures time-frequency complexity—complements spectral entropy.

### 3.3.4 Non-Linear Complexity (10 per channel)

| Feature | What It Measures | Workload Relevance |
|---------|------------------|-------------------|
| Permutation entropy | Ordinal pattern diversity | ↑ with cognitive load |
| SVD entropy | Singular value spread of delay embedding | ↑ complexity |
| Approximate entropy (ApEn) | Pattern repeatability | ↓ regularity with load |
| Sample entropy (SampEn) | ApEn without self-matches | More robust |
| Petrosian FD | Signal complexity (fast) | Fractal dimension proxy |
| Katz FD | Path length vs. diameter | Alternative FD |
| Higuchi FD (HFD) | Best fractal dimension estimator | ↑ with neural complexity |
| Detrended Fluctuation Analysis (DFA) | Long-range correlations | α ≈ 1 = 1/f noise (healthy) |
| Zero-crossing count | Same as ZC rate × N | Redundant but fast |
| Lempel-Ziv complexity | Compressibility of binarized signal | ↑ randomness |

**Why all 10?** Different measures capture different aspects of complexity. HFD and SampEn are most validated for EEG workload [@chikhi2022eeg]. We include all and let feature selection choose.

**Why not more (e.g., multiscale entropy)?** Computationally heavy; marginal gain over SampEn + DFA for 4-s epochs.

### 3.3.5 Connectivity (12 total)

| Pair | Rationale | Measures |
|------|-----------|----------|
| Fp1–Fp2 | Inter-hemispheric frontal | Corr, PLV (4–13 Hz), Coh (4–13 Hz) |
| F3–F4 | Dorsolateral prefrontal | Same |
| Fz–Pz | Frontal–parietal (midline) | Same |
| F7–F8 | Lateral frontal (Broca area) | Same |

**Why only 4 pairs?** 19 channels = 171 pairs. 171×3 = 513 features → curse of dimensionality. These 4 pairs are theoretically motivated:
- Fp1-Fp2: Inter-hemispheric integration
- F3-F4: Working memory network
- Fz-Pz: Frontal theta ↔ parietal alpha coupling [@dimitriadis2016causal]
- F7-F8: Language/verbal arithmetic areas

**Why theta-alpha (4–13 Hz) for PLV/coherence?** Workload literature shows theta-alpha coupling is most predictive [@dimitriadis2016causal; @katahira2018eeg].

**Why not phase-amplitude coupling (PAC)?** PAC requires longer epochs for reliable phase estimation; 4 s is borderline.

---

## 3.4 Normalization: Per-Subject Z-Scoring

For each subject $s$ and feature $f$:
$$z_{s,f} = \frac{x_{s,f} - \mu_{s,f}}{\sigma_{s,f}}$$
where $\mu_{s,f}, \sigma_{s,f}$ are computed over **all epochs of subject $s$ (Rest + Task)**.

### Why Per-Subject (Not Global)?
| Normalization | Problem |
|---------------|---------|
| Global (all subjects) | Between-subject variance (skull thickness, impedance) swamps within-subject Rest/Task effect |
| **Per-subject (this work)** | Removes baseline differences; preserves within-subject contrast |

### Why Without Labels?
- Uses **all epochs** (Rest + Task) → no label information used
- If we used only Rest to compute $\mu, \sigma$, we'd be "peeking" at condition structure

### Deployment Caveat
A real system needs **both conditions from a new user** to compute this normalization. Single-condition inference would require population statistics (introduces bias). This is a known limitation (Section 6).

---

## 3.5 Classification Pipeline (Zero Leakage)

```python
Pipeline([
    ('imputer', SimpleImputer(strategy='median')),      # Fit on train fold only
    ('var_filter', VarianceThreshold(threshold=1e-8)), # Remove constant features
    ('scaler', StandardScaler()),                        # Fit on train fold only
    ('selector', SelectKBest(f_classif, k=150)),        # Fit on train fold only
    ('clf', Classifier())                                # Fit on train fold only
])
```

### Why This Pipeline Design?

| Step | Why This Choice | Why Not Alternative |
|------|-----------------|---------------------|
| Median imputation | Robust to outliers (failed entropy = NaN) | Mean imputation skewed by outliers |
| VarianceThreshold(1e-8) | Removes numerical constants that break scaler | No filter → scaler warns on zero-variance |
| StandardScaler | SVM, LR, ANOVA need unit variance | RobustScaler: median/IQR better for outliers but SVM prefers z-score |
| SelectKBest(ANOVA, k=150) | Reduces 1,076 → 150; ANOVA F-test is fast, linear | Mutual info: nonlinear but slower; PCA: loses interpretability |
| k=150 | ~10% of features; heuristic: N_train ≈ 1000, k < N/5 | k=50: too aggressive; k=500: too noisy |

### Critical: All Steps Fit on Training Fold Only
- **No data leakage**: Test subject never influences imputer median, scaler mean/std, or feature selector scores.
- **ANOVA inside CV**: Feature ranking recomputed per fold using only training subjects.

---

## 3.6 Models Evaluated (17 Total)

### Why These 17?
We cover the full spectrum: linear, kernel, single trees, bagging, boosting, ensembles.

| Category | Models | Rationale |
|----------|--------|-----------|
| **Linear** | Logistic Regression (L2, C=0.05), SVM-Linear (C=0.03) | Baseline; interpretable; fast |
| **Kernel** | SVM-RBF (C=30, γ=0.0003) | Nonlinear boundary; strong in EEG literature |
| **Single Trees** | DecisionTree (max_depth=5), ExtraTree (max_depth=6) | Show overfitting baseline |
| **Bagging** | Bagging (300×), RandomForest (tuned), ExtraTrees (tuned) | Variance reduction via averaging |
| **Boosting** | AdaBoost (300×), GradBoost (tuned), HistGradBoost (tuned), XGBoost (tuned) | Sequential error correction |
| **Ensembles** | 5 soft-voting combinations | Combine diverse predictions |

### Why These Hyperparameters?
- **Tuned via grouped 5-fold CV** on training subjects only (GroupKFold, 12 random draws)
- **Fixed seeds (SEED=42)** for reproducibility
- **class_weight='balanced'** for all (harmless since classes balanced)

### Why Not Deep Learning?
| Reason | Detail |
|--------|--------|
| **N=36 subjects** | DL needs 100s–1000s of subjects for cross-subject generalization |
| **~1,000 epochs** | Too few for CNN/RNN; would memorize subjects |
| **Interpretability required** | SHAP on DL is approximate (KernelSHAP); trees/SVM have exact SHAP |
| **Prior work** | DL on EEGMAT [@ajra2022mental] used within-subject CV; no LOSO DL baseline exists |

---

## 3.7 Evaluation Protocol

### Primary: Leave-One-Subject-Out (LOSO)
- **36 folds**: Train on 35 subjects, test on 1
- **Per-subject metrics**: Balanced accuracy, AUC computed per test subject, then averaged
- **Why per-subject mean?** Treats each subject equally (deployment-relevant). Pooled metrics weight subjects by epoch count.

### Secondary: Fixed Hold-Out (Subjects 01, 22, 35)
- **Chosen a priori** (before any training/tuning)
- **Never used** in training, hyperparameter tuning, or model selection
- **Final model**: SVM-RBF (all features) chosen by grouped CV on 33 training subjects

### Why Both?
| Aspect | LOSO | Hold-Out |
|--------|------|----------|
| **N_test** | 36 subjects | 3 subjects |
| **Variance** | Low (36 estimates) | High (3 estimates) |
| **Bias** | Slight optimism (tuning on same pool) | Unbiased (truly unseen) |
| **Use case** | Primary result | Sanity check |

### Metrics
| Metric | Formula | Why |
|--------|---------|-----|
| Balanced Accuracy | $\frac{1}{2}(\text{Recall}_0 + \text{Recall}_1)$ | Primary; equals accuracy here (balanced classes) |
| ROC-AUC | $\int \text{TPR}(FPR) dFPR$ | Threshold-independent |
| Precision, Recall, F1 | Standard | Per-class detail |
| MCC | $\frac{TP\cdot TN - FP\cdot FN}{\sqrt{(TP+FP)(TP+FN)(TN+FP)(TN+FN)}}$ | Single correlation-style score |

---

# 4. Results

## 4.1 Exploratory Data Analysis

### 4.1.1 Theta Topography: No Universal Pattern

![Theta Topomaps](results/01_subject00_theta_topomap_rest_task_diff.png)
**Figure 1:** Subject 00 — Rest (left), Task (middle), Task−Rest difference (right). Log theta power (4–8 Hz). Red = Task > Rest. Color scale shared across Rest/Task for direct comparison. Difference map centered at zero (blue = Rest > Task, red = Task > Rest).

![All 36 Subjects Theta Grid](results/02_all_36_subjects_theta_topomap_grid.png)
**Figure 2:** Task−Rest theta difference for all 36 subjects (6×6 grid). Each panel: one subject. Shared color scale. **No consistent spatial pattern** — some show frontal red, some parietal, some mostly blue.

**Subject 00 Statistics (Welch t-test, Bonferroni ×5 regions):**
| Region | Mean Δ (Task−Rest) | p (uncorrected) | p (Bonferroni) |
|--------|-------------------|-----------------|----------------|
| Occipital | -0.179 | 0.156 | 0.779 |
| Frontal | -0.054 | 0.668 | 1.000 |
| Temporal | -0.047 | 0.638 | 1.000 |
| Central | +0.034 | 0.731 | 1.000 |
| Parietal | +0.016 | 0.885 | 1.000 |

**Why Bonferroni?** 5 regions tested; family-wise error rate control. Without correction, 1 false positive expected per subject by chance.

**Why Welch t-test?** Does not assume equal variance between Rest/Task epochs.

**Implication for modeling:** No single "theta hotspot" generalizes across subjects. Classifier must integrate distributed, subject-varying patterns — favors high-dimensional features + regularization over single-channel approaches.

### 4.1.2 Feature Redundancy (Fz Heatmap)

![Feature Correlation Heatmap](results/03_shap_top20_features.png)
*Note: Figure 3 shows SHAP; correlation heatmap in notebook Section 7. Fz band powers and logs correlate >0.95; ratios are inverses (θ/α vs α/θ ≈ -1).*

**Why this matters:** 1,076 features contain massive redundancy. Without regularization (L2, tree splits) or selection (ANOVA), models overfit to duplicated noise.

### 4.1.3 Group-Level Frontal Theta Effect

![Frontal Theta by Counter Quality](results/06_frontal_theta_violin_good_vs_bad_counters.png)
**Figure 3:** Frontal theta relative power (per-subject mean) by condition and counter quality. **Task > Rest** (median +0.0102, paired Wilcoxon $p=0.0105$, $n=36$). **Good vs. Bad counters: no difference** (Mann-Whitney $p=0.54$).

**Why paired Wilcoxon?** Same subjects in both conditions; non-parametric (outliers present).

**Why Mann-Whitney for counters?** Independent groups (different people), unequal N (26 vs 10), non-normal.

**Why no counter-quality effect?** Small N for bad counters (10); Task may reflect *attempted* effort, not success. Also: Rest always precedes Task → fatigue confound.

---

## 4.2 LOSO Model Comparison

### Full Results (Per-Subject Mean ± SD)

| Rank | Model | Family | Bal. Acc. | AUC | SD (Acc) |
|------|-------|--------|-----------|-----|----------|
| 1 | **SVM-RBF** | Kernel | **0.737** | **0.820** | 0.166 |
| 2 | AdaBoost | Boosting | 0.736 | 0.812 | 0.192 |
| 3 | XGBoost | Boosting | 0.735 | 0.816 | 0.198 |
| 4 | LogReg-L2 | Linear | 0.733 | 0.817 | 0.168 |
| 5 | SVM-Linear | Linear | 0.732 | 0.812 | 0.161 |
| 6 | Ensemble: SVM+Boosting | Ensemble | 0.738 | 0.822 | 0.194 |
| 7 | Ensemble: Top-3 (post-hoc) | Ensemble | 0.743 | 0.826 | 0.177 |
| 8 | ExtraTrees | Tree Ensemble | 0.724 | 0.788 | 0.206 |
| 9 | GradBoost | Boosting | 0.728 | 0.809 | 0.193 |
| 10 | HistGradBoost | Boosting | 0.727 | 0.807 | 0.194 |
| 11 | Bagging | Tree Ensemble | 0.708 | 0.782 | 0.206 |
| 12 | RandomForest | Tree Ensemble | 0.702 | 0.784 | 0.215 |
| 13 | ExtraTree (single) | Tree | 0.654 | 0.715 | 0.165 |
| 14 | DecisionTree (single) | Tree | 0.641 | 0.703 | 0.174 |

**Table 2:** LOSO Results (17 models). *Post-hoc ensemble chosen after seeing LOSO → optimistic.*

### Why SVM-RBF Leads (Analysis)

| Factor | SVM-RBF | Trees/Boosting |
|--------|---------|----------------|
| **High-dim, small-N** | Margin maximization in 150-D space | Split on 1 feature at a time; unstable |
| **Correlated features** | L2 regularization handles naturally | Trees pick one correlated feature arbitrarily |
| **Weak distributed signal** | Uses all 150 features smoothly | Must select few; discards weak cues |
| **Person-specific noise** | Regularization suppresses | Deep trees memorize; ensembles average but not fully |

**Why single trees fail (64–65%):** A depth-5 tree has ≤31 splits. With 150 features and distributed signal, 31 splits cannot capture the pattern. Each split fits training-subject idiosyncrasies → high variance.

**Why boosting helps (+7–9%):** Sequential error correction approximates additive model of many weak learners — closer to linear combination.

**Why top models within 1%?** Per-subject SD ≈ 0.17 → standard error of mean ≈ 0.17/6 ≈ 0.028. Differences < 0.01 are noise.

**Grouped CV tuning (33 training subjects) gave higher:** SVM-RBF 74.5%, XGBoost 74.2%, GradBoost 74.7% → confirms ~0.5–1% optimism from tuning on same pool.

### ROC & Confusion (SVM-RBF)

![ROC & Confusion](results/04_ROC_RF_XGB_and_XGB_confusion_matrix.png)
**Figure 4:** Left: ROC curves — pale lines = 36 subjects, dark line = mean. Wide spread (SD ≈ 0.2 AUC) confirms high inter-subject variability. Right: Confusion matrix — balanced errors (no Rest/Task bias).

---

## 4.3 Hold-Out Test (Subjects 01, 22, 35)

| Subject | Rest Epochs | Task Epochs | Bal. Acc. | AUC |
|---------|-------------|-------------|-----------|-----|
| 01 | 22 | 22 | 82.1% | 0.92 |
| 22 | 22 | 22 | 96.7% | 0.99 |
| 35 | 22 | 22 | 96.7% | 0.99 |
| **Pooled** | **66** | **66** | **92.0%** | **0.974** |
| **Smoothed (w=3)** | — | — | **97.7%** | — |

**Table 3:** Hold-out results (88 epochs total). Confusion: 40/44 Rest correct, 41/44 Task correct. All 6 recordings classified correctly at recording level.

### Critical Caveats (Why Not to Cite 92%)
1. **N=3 subjects only** — binomial 95% CI for 92% (81/88) ≈ [84%, 97%]
2. **SVM used all features** (including gamma/Fp1/Fp2 which boost separability on these subjects)
3. **Smoothing (w=3)** exploits temporal autocorrelation — not strict per-epoch performance
4. **Subject selection bias** — these 3 may be "easier" by chance

**LOSO 73.7% is the reliable generalization estimate.**

---

## 4.4 SHAP Interpretability (Hold-Out Subjects)

![SHAP Summary](results/05_shap_summary_xgboost_top10.png)
**Figure 5:** SHAP beeswarm (XGBoost, hold-out, trained on 33 subjects). Each dot = one epoch. Red = high feature value, right = pushes toward Task.

### Top 20 Features (KernelSHAP on SVM-RBF)

| Rank | Feature | Channel | Family | Direction (High →) |
|------|---------|---------|--------|-------------------|
| 1 | `higuchi_fd` | T6 | Nonlinear | **Task** |
| 2 | `gamma_power` | T5 | Spectral | **Task** |
| 3 | `wavelet_energy_L4` | C3 | Wavelet | **Task** |
| 4 | `wavelet_energy_L3` | P3 | Wavelet | **Task** |
| 5 | `theta_power` | Fz | Spectral | **Task** |
| 6 | `gamma_power` | Fp2 | Spectral | **Task** |
| 7 | `gamma_power` | Fp1 | Spectral | **Task** |
| 8 | `katz_fd` | T6 | Nonlinear | **Task** |
| 9 | `petrosian_fd` | T6 | Nonlinear | **Task** |
| 10 | `beta_power` | C4 | Spectral | **Task** |
| 11 | `beta_power` | C3 | Spectral | **Task** |
| 12 | `line_length` | Fp1 | Time | **Task** |
| 13 | `spectral_edge_95` | Fp2 | Spectral | **Task** |
| 14 | `higuchi_fd` | T5 | Nonlinear | **Task** |
| 15 | `wavelet_energy_L4` | C4 | Wavelet | **Task** |
| 16 | `gamma_power` | F4 | Spectral | **Task** |
| 17 | `permutation_entropy` | Fz | Nonlinear | **Rest** |
| 18 | `wavelet_entropy` | P3 | Wavelet | **Rest** |
| 19 | `alpha_power` | Fz | Spectral | **Rest** |
| 20 | `theta_alpha_ratio` | Fz | Spectral | **Task** |

**Table 4:** Top 20 SHAP features (hold-out, SVM-RBF).

### Key SHAP Observations

| Observation | Evidence | Interpretation |
|-------------|----------|----------------|
| **Frontal/temporal dominance** | 45% of top 20 from frontal/temporal (42% of scalp) | Workload networks are fronto-temporal |
| **Complexity > power** | Higuchi FD (T6) #1; 3 FD in top 10 | Signal irregularity > band power |
| **Gamma prominent** | 5 gamma features in top 20 (T5, Fp1, Fp2, F4) | **Artifact concern** (see Section 4.5) |
| **Wavelet beta/alpha** | C3 L4 (16–32 Hz), P3 L3 (8–16 Hz) | Time-frequency localization helps |
| **Connectivity absent** | No connectivity in top 50 | Weak contribution vs. single-channel |
| **Alpha → Rest** | Fz alpha power pushes toward Rest | Consistent with alpha desynchronization |

### Why SHAP on Hold-Out (Not LOSO)?
- **Computational**: KernelSHAP on 36×SVM-RBF too slow
- **Interpretability**: Single model on fixed test set = stable explanations
- **Caveat**: Feature importance may not generalize to other test subjects

---

## 4.5 Robustness Checks: Neural vs. Artifact Contributions

| Exclusion | Features Removed | LOSO Bal. Acc. | Δ vs. Full |
|-----------|------------------|----------------|------------|
| **Full** | — | **0.737** | — |
| Fp1/Fp2 channels | 112 (56×2) | 0.712 | -0.025 |
| All gamma-band | 95 (5×19) | 0.701 | -0.036 |
| **Fp1/Fp2 + gamma** | **187** | **0.678** | **-0.059** |

**Table 5:** Robustness: Performance degradation when artifact-prone features removed.

### Why These Exclusions?
| Feature Group | Artifact Risk | Evidence |
|---------------|---------------|----------|
| **Fp1/Fp2** | Blinks, eye movements | Frontal poles closest to eyes; no EOG channels |
| **Gamma (30–45 Hz)** | Scalp/jaw muscle (EMG) | Muscle power peaks >30 Hz; ICA imperfect |

### Interpretation
- **5.9% drop** when both removed → substantial non-neural contribution
- **67.8% still > 50% chance** → genuine neural signal exists
- **Gamma drop > Fp1/Fp2 drop** → muscle more pervasive than blinks
- **SHAP top features include gamma/Fp1/Fp2** → model uses artifacts for separation

**Why not just remove them upfront?** Would discard potentially useful neural gamma (e.g., [@ishii2014frontal] shows frontal gamma during calculation). Better to **quantify** the contribution.

---

## 4.6 Per-Subject Variability

- **Range**: ~0.35 to ~0.95 balanced accuracy (SVM-RBF, LOSO)
- **SD**: 0.166 across 36 subjects
- **Correlation with artifacts**: Subjects with more ICA components rejected → lower accuracy
- **Correlation with counter quality**: $r = 0.08$, $p = 0.64$ (no relation)

**Why high variability?**
1. **Anatomical differences**: Skull thickness, CSF volume affect EEG amplitude
2. **Artifact burden**: Some subjects blink more, clench jaw
3. **Strategy differences**: "Bad counters" may not engage working memory similarly
4. **Small epoch count**: ~15/condition → noisy per-subject estimates

---

# 5. Discussion

## 5.1 Why LOSO ~74% and Not Higher?

| Factor | Impact | Evidence |
|--------|--------|----------|
| **Effective N = 36** | High-dimensional (1,076 feat) but only 36 independent samples | Per-subject SD 0.166 >> 0 |
| **No universal topography** | Fig. 2: 36 different patterns | Classifier learns distributed weak cues |
| **Condition order confound** | Rest always before Task | Fatigue/drift mimics workload [@berka2007eeg] |
| **19-ch, no EOG/EMG** | ICA with Fp1/Fp2 proxies limited | Robustness drop 5.9% |
| **Task difficulty variance** | 12/36 "bad counters" | Their Task ≠ high effort |

**Why not higher with more features?** 1,076 features → 150 selected. Adding more (e.g., connectivity, PAC) increases noise faster than signal.

**Why not higher with deep learning?** DL needs more subjects; no LOSO DL baseline on EEGMAT exists.

## 5.2 Why Hold-Out 92% ≠ True Performance?

| Reason | Detail |
|--------|--------|
| **Tiny test set** | 3 subjects = 88 epochs; binomial variance huge |
| **Feature selection bias** | SVM used all features (gamma/Fp1/Fp2 help these 3) |
| **Smoothing artifact** | w=3 rolling mean exploits epoch adjacency |
| **Selection bias** | 3 subjects chosen randomly may be "easy" |

**Never cite 92% as generalization.** It is a sanity check only.

## 5.3 Literature Alignment

| Finding | This Work | Prior Work |
|---------|-----------|------------|
| **Frontal θ ↑** | Fz theta #5 SHAP; group median +0.0102 | [@chikhi2022eeg] meta-analysis: θ best index |
| **Alpha ↓** | Fz alpha pushes → Rest (SHAP #19) | [@berka2007eeg] engagement = α↓ |
| **Gamma ↑** | 5 gamma in top 20 SHAP | [@ishii2014frontal] MEG: gamma during calculation |
| **Complexity ↑** | Higuchi FD #1 SHAP | [@aslam2025electroencephalograph] EMD+entropy |
| **LOSO ~74%** | SVM-RBF 73.7% | [@sibisanu2026high] LOSO AUC 0.66–0.71 |
| **Subject-dep ~99%** | N/A (we don't report) | [@nath2020eeg] 97.5% (random split) |

**Key conclusion:** Our 73.7% is **consistent with the only rigorous prior LOSO study** and **inconsistent with subject-dependent claims** — confirming evaluation design drives reported performance.

## 5.4 Limitations (Honest)

| # | Limitation | Impact | Mitigation / Future Work |
|---|------------|--------|--------------------------|
| 1 | **Rest always before Task** | Fatigue/drift confounded with workload | Counterbalance order; within-session design |
| 2 | **No dedicated EOG/EMG** | ICA with Fp1/Fp2 proxies imperfect | 64+ ch cap with EOG/EMG electrodes |
| 3 | **Per-subject z-score needs both conditions** | Not deployable for single-condition inference | Population-normative z-scoring (with bias) |
| 4 | **Hold-out N=3** | Unreliable estimate | Larger hold-out (e.g., 10 subjects) |
| 5 | **Tuning optimism** | Grouped CV on same 36 subjects | Nested CV (expensive: 36×5×12 = 2,160 fits) |
| 6 | **Binary only (Rest vs Task)** | Workload *levels* not addressed | Regression on counter quality / RT |
| 7 | **Young adults only (18–35)** | Generalizability unknown | Test on older / clinical populations |
| 8 | **4-s epochs, non-overlapping** | Fewer samples; ignores temporal continuity | Overlapping epochs; temporal models (HMM) |

---

# 6. Conclusion

We presented a leak-free EEG classification pipeline for Mental Arithmetic vs. Rest on the EEGMAT dataset. Under rigorous leave-one-subject-out evaluation, the best model (SVM-RBF) achieves **73.7% balanced accuracy** (AUC 0.820), with boosting ensembles (XGBoost, AdaBoost) statistically indistinguishable. A fixed hold-out test yields 92.0% but is unreliable due to small sample size. SHAP reveals frontal/temporal complexity and gamma features as top discriminators, but robustness checks show **5.9% accuracy drop** when artifact-prone features are excluded, confirming mixed neural/non-neural origins. The result is consistent with the only prior LOSO study on this dataset and substantially lower than subject-dependent claims (90–99%), underscoring that **evaluation design—not model cleverness—drives reported performance**. Future work: within-session condition counterbalancing, higher-density EEG with dedicated artifact channels, and workload-level regression.

---

# Acknowledgment

This work was completed as part of the UG3 BTP 4th Evaluation (Group B28SSR04) at IIIT Sri City. Dataset courtesy of Zyma *et al.* via PhysioNet. Computational resources: local workstation. Code and data: https://github.com/PoornaSri26/EEG

---

# References