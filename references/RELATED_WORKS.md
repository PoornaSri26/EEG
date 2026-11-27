# Related Work & References

Literature review and citations supporting the EEG Mental Arithmetic
(Rest vs Task) classification project. Grouped by topic; each entry notes
how it relates to this work.

## 1. Dataset

**Zyma, I., Tukaev, S., Seleznov, I., Kiyono, K., Popov, A., Chernykh, M.,
and Shpenkov, O. (2019).**
*Electroencephalograms during Mental Arithmetic Task Performance.*
Data, 4(1), 14. https://doi.org/10.3390/data4010014
PhysioNet: https://physionet.org/content/eegmat/1.0.0/

- 36 subjects, 23-channel EEG (10–20, ear reference), Rest (`_1`) and
  Mental Arithmetic (`_2`) recordings per subject.
- Subjects split into "good counters" (Group G, n=24) and "bad counters"
  (Group B, n=12) by subtraction performance — a built-in workload
  manipulation we exploit in the good-vs-bad analysis.
- This is the dataset used throughout the project.

## 2. Prior work on this dataset (EEGMAT)

**Nath, D., Anubhav, Kalra, D., Singh, M., Sethia, D., and Indu, S. (2020).**
*EEG-based Mental Workload Detection in a Mental Arithmetic Task using
Machine Learning.* International Journal of Advanced Science and
Technology, 29(5), 13975–13984.

- Same dataset. Spectral density, relative power, alpha/beta and
  theta/alpha ratio features + RFECV selection; NB, SVM, k-NN.
- Reports ~97.5% with SVM — but uses random splits rather than
  subject-independent evaluation, so subject leakage likely inflates it.
- Our work differs: LOSO evaluation, richer feature set (wavelet,
  non-linear, connectivity), and explicit artifact/leakage analysis.

**Sibisanu, R., Leba, M., and Ionica, A. C. (2026).**
*High Accuracy Is Not Generalization: Evaluation Design in EEG Mental
Arithmetic Classification.* Entropy, 28(11), 1275.
https://doi.org/10.3390/e28111275

- The most directly relevant prior study: systematically re-evaluates
  rest-vs-arithmetic classification on **this same EEGMAT dataset**
  under subject-dependent vs. subject-independent (LOSO) protocols.
- Shows reported 88–99% accuracies collapse under LOSO (kappa 0.19–0.26,
  ROC-AUC 0.66–0.71) — directly motivating our leak-free pipeline:
  in-fold feature selection, per-subject normalization without labels,
  and hold-out subjects. Our honest LOSO result (~74% balanced
  accuracy) is consistent with this picture of modest but real
  subject-independent signal.

**Sanjay, D. A. and Sheethal, P. P. (2025).**
*Detection of Mental Arithmetic Task Using EEG Signals.*
International Journal of Sciences and Innovation Engineering, 2(12),
902–906. https://doi.org/10.70849/IJSCI

- Mental arithmetic vs rest; time/frequency/time–frequency features;
  reports increased theta and frontal midline activity — consistent with
  our top SHAP feature (frontal theta).

**Rambhia, J. and Sutar, R. (2023).** *Classification of Mental
Arithmetic States Using Single Electrode.* 2023 International Conference
on Next Generation Electronics (NEleX), pp. 1–4.
https://doi.org/10.1109/NELEX59773.2023.10421525

- EEGMAT dataset; wavelet alpha-band features from a single occipital
  electrode (O1); reports very high accuracy — included for completeness;
  single-channel results are not comparable to our 19-channel,
  subject-independent evaluation.

**Alhothali, A. S. and Attar, E. T. (2026).** *Validation of EEG mental
workload markers using integrated statistical and machine learning
analyses.* Applied Neuropsychology: Adult, 1–13.
https://doi.org/10.1080/23279095.2026.2686957

- Reanalysis of this same 36-subject EEGMAT dataset: Welch PSD band
  powers + ERPs, Logistic Regression / SVM / Random Forest under
  subject-level cross-validation. Reports theta ↑, alpha ↓ (largest
  effects), RF accuracy 0.92 — the same directional band effects our
  EDA and SHAP analyses find, though their subject-level CV is weaker
  than our LOSO protocol.

## 3. EEG mental workload / cognitive-state literature

**Berka, C., Levendowski, D. J., Lumicao, M. N., Yau, A., Davis, G.,
Zivkovic, V. T., Olmstead, R. E., Tremoulet, P. D., and Craven, P. L.
(2007).** *EEG correlates of task engagement and mental workload in
vigilance, learning, and memory tasks.* Aviation, Space, and
Environmental Medicine, 78(5 Suppl), B231–B244.

- Landmark study showing EEG workload indices rise with working-memory
  load — including a mental-addition task — motivating theta/alpha
  band-power features.

**Ajra, Z., Xu, B., Dray, G., Montmain, J., and Perrey, S. (2022).**
*Mental arithmetic task classification with convolutional neural network
based on spectral-temporal features from EEG.* 44th Annual International
Conference of the IEEE Engineering in Medicine & Biology Society (EMBC),
pp. 1–4. https://doi.org/10.1109/EMBC48229.2022.9870887
(arXiv:2209.11767)

- Shallow CNN on spectral-temporal EEG features for mental arithmetic;
  ~90.7% accuracy with attention to cross-subject robustness — supports
  both deep and classical approaches to this problem.

**Aslam, M. (2025).** *Electroencephalograph (EEG) based classification
of mental arithmetic using explainable machine learning.*
Biocybernetics and Biomedical Engineering, 45(2), 154–169.
https://doi.org/10.1016/j.bbe.2025.02.002

- EMD features + RF-RFE ranking + RF classifier on mental arithmetic;
  ~98–99% claimed accuracies but evaluated within-subject — highlights
  why subject-independent (LOSO) evaluation matters.

**Chikhi, S., Matton, N., and Blanchet, S. (2022).** *EEG power spectral
measures of cognitive workload: A meta-analysis.* Psychophysiology,
59(6), e14009. https://doi.org/10.1111/psyp.14009

- Meta-analysis of 24 studies / 723 participants: cognitive workload
  robustly increases **theta** (g = 0.68) and **beta** (g = 0.50), with
  **frontal theta the best single index** of workload — the same band
  our SHAP analysis ranks first.

**Katahira, K., Yamazaki, Y., Yamaoka, C., Ozaki, H., Nakagawa, S., and
Nagata, N. (2018).** *EEG Correlates of the Flow State: A Combination of
Increased Frontal Theta and Moderate Frontocentral Alpha Rhythm in the
Mental Arithmetic Task.* Frontiers in Psychology, 9, 300.
https://doi.org/10.3389/fpsyg.2018.00300

- EEG during an experimentally controlled mental arithmetic task;
  frontal theta rises with engagement while frontal alpha stays
  moderate — neural evidence that arithmetic specifically engages
  frontal-midline theta, matching our topographic findings.

**Ishii, R., Canuet, L., Ishihara, T., Aoki, Y., Ikeda, S., Hata, M.,
Katsimichas, T., Gunji, A., Takahashi, H., Nakahachi, T., Iwase, M., and
Takeda, M. (2014).** *Frontal midline theta rhythm and gamma power
changes during focused attention on mental calculation: an MEG
beamformer analysis.* Frontiers in Human Neuroscience, 8, 406.
https://doi.org/10.3389/fnhum.2014.00406

- MEG source analysis localizing frontal-midline theta generators to
  dorsal anterior cingulate / medial prefrontal cortex during mental
  subtraction — the same task as our dataset.

**Dimitriadis, S. I., Sun, Y., Thakor, N. V., and Bezerianos, A.
(2016).** *Causal Interactions between Frontal θ – Parieto-Occipital
α2 Predict Performance on a Mental Arithmetic Task.* Frontiers in Human
Neuroscience, 10, 454. https://doi.org/10.3389/fnhum.2016.00454

- Five workload levels of mental arithmetic; shows frontal theta and
  parieto-occipital alpha are functionally coupled and jointly predict
  performance — supports our theta–alpha connectivity features
  (PLV/coherence) rather than single-band power alone.

**Pušica, M., Mijović, B., Leva, M. C., and Gligorijević, I. (2025).**
*Machine learning performance in EEG-based mental workload
classification across task types: a systematic review.* Frontiers in
Neuroergonomics, 6, 1621309.
https://doi.org/10.3389/fnrgo.2025.1621309

- Systematic review of ML for EEG workload classification; documents
  that reported accuracies depend heavily on task type and evaluation
  design, and that SVM/CNN-type models dominate — context for both our
  model-zoo results and our cautious LOSO framing.

## 4. Methods underpinning the pipeline

- **Preprocessing & artifact removal**
  - Gramfort, A., Luessi, M., Larson, E., Engemann, D. A., Strohmeier, D.,
    Brodbeck, C., Goj, R., Jas, M., Brooks, T., Parkkonen, L., and
    Hämäläinen, M. (2013). *MEG and EEG data analysis with MNE-Python.*
    Frontiers in Neuroscience, 7, 267.
  - Jas, M., Engemann, D. A., Bekhti, Y., Raimondo, F., and Gramfort, A.
    (2017). *Autoreject: Automated artifact rejection for MEG and EEG
    data.* NeuroImage, 159, 361–368.
  - Makeig, S., Bell, A. J., Jung, T.-P., and Sejnowski, T. J. (1997).
    *Independent component analysis of electroencephalographic data.*
    Advances in Neural Information Processing Systems (NIPS), 8.
  - Hjorth, B. (1970). *EEG analysis based on time domain properties.*
    Electroencephalography and Clinical Neurophysiology, 28(3), 306–310.
    (Source of the Hjorth mobility/complexity features used here.)
  - Puce, A. and Hämäläinen, M. S. (2017). *A Review of Issues Related
    to Data Acquisition and Analysis in EEG/MEG Studies.* Brain
    Sciences, 7(6), 58. https://doi.org/10.3390/brainsci7060058 —
    best-practices review covering ICA artifact removal (blinks,
    muscle, ECG) that our preprocessing follows.
  - Bigdely-Shamlo, N., Mullen, T., Kothe, C., Su, K.-M., and Robbins,
    K. A. (2015). *The PREP pipeline: standardized preprocessing for
    large-scale EEG analysis.* Frontiers in Neuroinformatics, 9, 116.
    https://doi.org/10.3389/fninf.2015.00116 — standardized
    referencing/bad-channel handling relevant to our average-reference
    and channel-matching steps.

- **Connectivity features**
  - Lachaux, J.-P., Rodriguez, E., Martinerie, J., and Varela, F. J.
    (1999). *Measuring phase synchrony in brain signals.* Human Brain
    Mapping, 8(4), 194–208. (Phase-locking value used for theta–alpha
    connectivity.)

- **Interpretability & evaluation**
  - Lundberg, S. M. and Lee, S.-I. (2017). *A unified approach to
    interpreting model predictions.* NeurIPS, 30. (SHAP feature
    importance.)
  - Leave-one-subject-out (LOSO) evaluation is standard practice for
    subject-independent EEG decoding; random trial-level splits leak
    subject identity and inflate accuracy (see Sibisanu et al. 2026 and
    the Pušica et al. 2025 review above).

## 5. How this project maps to the literature

| What we did | Supporting / contrasting work |
|---|---|
| Used the public EEGMAT dataset (36 subjects, rest vs serial subtraction) | Zyma et al. 2019 |
| Band-pass 1–45 Hz + 50 Hz notch, resample to 128 Hz, average reference, 4 s epochs | Gramfort et al. 2013 (MNE); Puce & Hämäläinen 2017; Bigdely-Shamlo et al. 2015 |
| ICA removal of blink/muscle components (Fp1/Fp2 as EOG proxies) + AutoReject | Makeig et al. 1997; Jas et al. 2017; Puce & Hämäläinen 2017 |
| Time-domain (incl. Hjorth), spectral band power, wavelet (db4), non-linear features | Hjorth 1970; Chikhi et al. 2022 (theta/alpha/beta sensitivity) |
| Theta–alpha PLV and coherence connectivity features | Lachaux et al. 1999; Dimitriadis et al. 2016 (frontal θ – parieto-occipital α coupling in arithmetic) |
| Frontal theta emerges as the top discriminative feature (SHAP) | Chikhi et al. 2022 meta-analysis; Berka et al. 2007; Katahira et al. 2018; Alhothali & Attar 2026 (same dataset) |
| Per-subject z-normalization without labels | Standard practice; assumption flagged in notebook §10 |
| Leave-one-subject-out evaluation + dedicated hold-out subjects | Sibisanu et al. 2026; Pušica et al. 2025 — contrasts with Nath et al. 2020 (~97.5%) and Aslam 2025 (~99%), which use subject-dependent splits |
| Model zoo (SVM, RF, XGBoost, ensembles) + SHAP interpretability | Lundberg & Lee 2017; Pušica et al. 2025 (SVM/CNN dominate workload ML) |
| Honest ~74% LOSO accuracy with artifact/confound caveats | Consistent with Sibisanu et al. 2026's modest LOSO findings; Section 16 robustness check addresses artifact-prone channels flagged by Puce & Hämäläinen 2017 |

## BibTeX

Machine-readable versions of the above are in `references.bib`.
