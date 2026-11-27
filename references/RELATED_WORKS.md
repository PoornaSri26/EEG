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
    subject identity and inflate accuracy.

## BibTeX

Machine-readable versions of the above are in `references.bib`.
