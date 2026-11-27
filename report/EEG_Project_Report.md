EEG Mental Arithmetic: Rest vs Task Classification
Complete Presentation Script, Slide-by-Slide Deep Report
Companion to: EEG_Mental_Arithmetic_Presentation.pptx (10 slides)
Source: your Jupyter notebook (notebookf5d0ec746d__1_.ipynb)
Suggested talk length: about 25 to 30 minutes (about 2.5 minutes per slide, plus questions)

How to use this document
Every slide section has the same parts, in this order:

What is on the slide. Every line and every picture on that slide, listed so you can point at it.
Script. Words to say out loud, written in plain speech. Read it as written or paraphrase it.
Picture-by-picture explanation. How to read each figure, axis by axis, plus what it proves and what it does not prove.
How, Why, Why not. The reasoning behind each design choice and the alternatives that were rejected.
Deep analysis. Numbers, statistics, caveats and what a skeptical reviewer would say.
Likely questions and answers.
Honesty rules used throughout this document

Every number comes from your notebook's saved outputs. Numbers read off a picture (not printed as text) are marked (approx., read from the plot).
The notebook never draws a tree diagram, and the Section 16 robustness-check cell has no saved output. This document says so rather than inventing results.
Where the notebook is explicit about a limitation (Section 18), the script repeats it. A presentation that admits its limits is more credible than one that hides them.
PART A. ABBREVIATIONS AND TERMS (READ THIS FIRST)
Use this as a reference when someone asks "what does X mean?" Terms are grouped by topic.

A1. Brain-signal and recording terms
Term	Full form	Meaning in this project
EEG	Electroencephalography	Recording of tiny voltage changes on the scalp caused by brain activity. Measured in microvolts (µV).
EDF	European Data Format	The file format of each recording (Subject00_1.edf = Rest, Subject00_2.edf = Task).
10-20 system	International 10-20 electrode placement	Standard map of where electrodes sit on the head. Electrodes sit at 10% or 20% steps of the head's measured distances.
Channel	One electrode's signal	19 channels are used: Fp1, Fp2, F3, F4, Fz, F7, F8, C3, C4, Cz, T3, T4, T5, T6, P3, P4, Pz, O1, O2.
Fp / F / C / T / P / O	Frontal-pole / Frontal / Central / Temporal / Parietal / Occipital	The lobe of the brain under the electrode.
Odd / even / z numbers	Left / right / midline	Odd numbers (Fp1, F3, C3...) are on the left, even (Fp2, F4, C4...) on the right, "z" (Fz, Cz, Pz) on the midline.
A1, A2	Auricular (earlobe) electrodes	Reference electrodes on the ears. The channel "A2-A1" is not a brain channel, so it is dropped.
ECG	Electrocardiogram	Heart signal. Dropped because it is not EEG.
EOG	Electrooculogram	Eye-movement and blink signal. The dataset has no EOG channel, so Fp1/Fp2 are used as stand-ins ("proxies").
EMG	Electromyogram	Muscle signal (jaw clenching, scalp tension). A contaminant for EEG.
Artifact	Non-brain signal	Blinks, eye movements, jaw/neck muscle, mains hum, electrode movement.
Reference	The baseline the voltages are measured against	Here: average reference (subtract the mean of all channels from each).
Epoch	A fixed time window	Here: 4 seconds = 512 samples at 128 Hz.
Rest / Background	Condition 0	Subject sits quietly (recording 1).
Task	Condition 1	Subject does serial mental subtraction (recording 2).
Counting quality	Good / bad counter	Subject metadata: whether the person counted well (1) or poorly (0).
Hz	Hertz	Cycles per second.
Sampling rate (sfreq)	Samples per second	Resampled to 128 Hz.
Nyquist frequency	Half the sampling rate	64 Hz at 128 Hz sampling. You can only represent frequencies below this.
A2. The five brain-rhythm bands
Band	Range in this notebook	Typical association (general literature, not proven by this project)
Delta (δ)	0.5 to 4 Hz	Deep sleep, slow drifts; also eye-movement artifact leaks in here.
Theta (θ)	4 to 8 Hz	Mental effort and working memory, especially frontal midline.
Alpha (α)	8 to 13 Hz	Relaxed wakefulness; tends to drop with concentration.
Beta (β)	13 to 30 Hz	Active thinking, alertness; muscle can leak in at the top.
Gamma (γ)	30 to 45 Hz	Fast activity; also where scalp/jaw muscle noise lives, so it is suspect.
A3. Signal-processing terms
Term	Full form	Meaning
FIR	Finite Impulse Response	A type of digital filter. With zero-phase it does not shift the timing of the signal.
Band-pass	Filter that keeps a frequency range	1 to 45 Hz here.
Notch filter	Filter that removes one frequency	50 Hz here (mains electricity hum).
High-pass / low-pass edge	1 Hz and 45 Hz	1 Hz removes slow drift, 45 Hz removes fast noise.
Resample	Change sampling rate	128 Hz here.
ICA	Independent Component Analysis	Splits the 19 channels into independent source signals so blink and muscle sources can be removed.
FastICA	A fast ICA algorithm	The specific ICA algorithm used.
Component	One ICA source	15 are computed; at most 4 are removed.
AutoReject	Automated epoch rejection (a library)	Learns per-channel amplitude thresholds using cross-validation, then repairs or drops bad epochs.
PSD	Power Spectral Density	How much signal power sits at each frequency.
Welch method	Averaged-segment PSD estimate	Splits the signal into overlapping segments, computes the spectrum of each, averages. Here the segments are 256 samples (2 s), giving 0.5 Hz resolution.
Band power	PSD integrated over a band	Area under the PSD between the band edges (trapezoid rule).
Relative power	Band power / total power	Share of total power in that band (a number between 0 and 1).
Log power	Natural log of power	Makes the very skewed power values closer to bell-shaped.
Wavelet / DWT	(Discrete) wavelet transform	Splits the signal into frequency levels while keeping time information. Here: Daubechies-4 (db4) wavelet, 5 levels.
L0 to L5	Wavelet levels	L0 = 0-2 Hz, L1 = 2-4, L2 = 4-8, L3 = 8-16, L4 = 16-32, L5 = 32-64 Hz (effectively 32-45 Hz after filtering).
Spectral edge 95	Frequency below which 95% of power lies	A "how fast is the signal" summary.
Peak frequency	Frequency with maximum power	
Spectral entropy	Flatness of the spectrum	High = power spread over many frequencies; low = one dominant rhythm.
Engagement index	β / (α + θ)	A classic mental-engagement ratio.
Theta/alpha, theta/beta, alpha/theta	Band-power ratios	Compare two rhythms.
A4. Complexity ("non-linear") features
Term	Meaning (plain words)
Hjorth parameters	Three time-domain descriptors: activity (= variance), mobility (typical frequency), complexity (how much the shape differs from a pure sine wave).
Line length	Sum of absolute differences between consecutive samples. Large for jagged, high-amplitude, fast signals.
Zero-crossing rate	How often the signal crosses zero (a rough frequency measure).
Peak-to-peak	Maximum minus minimum value in the epoch.
RMS	Root mean square: the typical amplitude.
Skewness / kurtosis	Asymmetry / "tailedness" of the amplitude distribution. Kurtosis spikes with blinks and muscle bursts.
Permutation entropy	Randomness measured from the ordering patterns of consecutive samples.
SVD entropy	Entropy of the singular values of the delay-embedded signal (a complexity measure).
Approximate entropy (ApEn) / Sample entropy (SampEn)	How unpredictable the signal is. More regular = lower value.
FD (fractal dimension)	How "rough" or space-filling the waveform is. Three versions: Petrosian, Katz, Higuchi (HFD / higuchi_fd).
DFA	Detrended Fluctuation Analysis: long-range correlation (memory) in the signal.
LZ complexity	Lempel-Ziv complexity: how compressible the binarised signal is.
A5. Connectivity features
Term	Meaning
Connectivity	How strongly two channels' signals are linked.
Correlation (corr)	Broadband linear similarity between two channels.
PLV	Phase-Locking Value: how consistently two signals keep the same phase relationship (0 = random, 1 = perfectly locked). Here computed for 4-13 Hz (theta-alpha).
Coherence	Frequency-specific correlation between two channels (theta-alpha, 4-13 Hz).
Pairs used	Fp1-Fp2, F3-F4, Fz-Pz, F7-F8 (4 pairs x 3 measures = 12 features).
A6. Machine-learning terms
Term	Full form	Meaning
Feature	An input number	One measurement, e.g. T6_higuchi_fd = Higuchi fractal dimension at channel T6.
Feature name pattern	<channel>_<feature>	e.g. Fz_theta_power, P3_line_length.
Label	The answer	0 = Rest, 1 = Task.
Z-score	Standardised value	(value - mean) / standard deviation. Here computed within each subject.
Imputation	Filling missing values	Median imputation: missing values are replaced by the training median.
Variance filter / VarianceThreshold	Removes constant features	Drops any feature with variance below 1e-8.
StandardScaler	Z-score scaler fitted on training data	
ANOVA F-test / f_classif	Analysis of variance	Scores each feature by how well it separates the two classes.
SelectKBest / TOP_K = 150	Keep the k best features	Keeps the 150 highest-scoring features, chosen inside each training fold only.
Pipeline	Chain of steps	impute, variance filter, scale, select, classify, all fitted on training rows only.
Data leakage	Test information influencing training	The main trap this notebook is built to avoid.
CV	Cross-validation	Repeatedly training on part of the data and testing on the rest.
Grouped CV (GroupKFold)	CV that keeps each subject in one fold	Prevents the same person appearing in train and test.
LOSO / LOGO	Leave-One-Subject-Out (Leave-One-Group-Out)	Train on 35 subjects, test on the 36th; repeat 36 times.
Hold-out	Data set aside	Subjects 01, 22 and 35, never used for any training or choice.
OOF	Out-of-fold	A prediction for a sample made by a model that did not see it in training.
Hyperparameter	A setting chosen by the human/search, not learned	e.g. number of trees, C, learning rate.
RandomizedSearchCV	Random hyperparameter search	Tries TUNE_ITERS = 12 random settings.
Class weight "balanced"	Reweights classes	Compensates for class imbalance (here classes are already equal).
SEED = 42	Random seed	Makes results reproducible.
Overfitting	Memorising noise	High training accuracy, poor test accuracy.
Variance (model)	Sensitivity to the specific training data	Single decision trees have high variance.
Ensemble	Combination of models	Averaging probabilities of several models.
Soft vote	Average predicted probabilities	Used in the VotingClassifier.
Bagging	Bootstrap aggregating	Train many models on random samples and average.
Boosting	Sequential error-correcting ensemble	Each new tree focuses on the previous mistakes.
Smoothing (w=3)	Rolling mean of neighbouring epochs' probabilities	Only valid if epochs of a recording come in time order. Not a strict per-epoch result.
A7. Models
Abbreviation	Full name	One-line description
LogReg / LR (L2)	Logistic Regression (L2-regularised)	Linear model giving probabilities; L2 penalty shrinks weights. C = 0.05.
SVM	Support Vector Machine	Finds the widest-margin boundary between classes.
SVM-Linear	SVM with straight boundary	C tuned to 0.03 in LOSO.
SVM-RBF	SVM with Radial Basis Function kernel	Curved boundary; controlled by C and gamma. Tuned: C = 30, gamma = 0.0003.
DT / DecisionTree	Decision Tree	One tree of yes/no questions (max depth 5, min leaf 10).
ExtraTree (single)	Extremely Randomised Tree	One tree with random split thresholds (max depth 6).
Bagging (trees)	Bagged decision trees	300 trees of depth 6, 80% of samples, 50% of features each.
RF	Random Forest	Many de-correlated trees voting.
ET / ExtraTrees	Extremely Randomised Trees	Forest with random thresholds, smoother than RF.
AdaBoost	Adaptive Boosting	300 weak trees, each reweighting the mistakes.
GB / GradientBoosting	Gradient Boosting	Each tree fits the remaining error.
HistGB / HistGradBoost	Histogram Gradient Boosting	Fast gradient boosting on binned features.
XGB / XGBoost	eXtreme Gradient Boosting	Regularised gradient boosting (depth 3, L2 reg_lambda 3, column sampling 0.5).
SHAP	SHapley Additive exPlanations	Splits each prediction into per-feature contributions using game-theory (Shapley) values.
A8. Metrics
Metric	Meaning	Chance level
Accuracy	Fraction correct.	0.50 (balanced classes)
Balanced accuracy (bal_acc)	Average of Rest recall and Task recall. Same as accuracy here because the classes are equal.	0.50
Precision	Of the epochs predicted "Task", the fraction that truly are Task.	
Recall (sensitivity)	Of the true Task epochs, the fraction found.	
F1 / f1_task	Harmonic mean of precision and recall for Task.	
ROC	Receiver Operating Characteristic curve: true-positive rate vs false-positive rate across all thresholds.	diagonal
AUC	Area Under the ROC Curve: probability a random Task epoch scores higher than a random Rest epoch. 1.0 perfect, 0.5 chance.	0.50
MCC	Matthews Correlation Coefficient: a single correlation-style score from -1 to +1; 0 = chance.	0
Confusion matrix	2x2 table of true vs predicted classes.	
Pooled vs per-subject mean	Pooled: all epochs together. Per-subject mean: score each person, then average the 36 scores.	
SD (per-subj std)	Standard deviation of the 36 per-subject scores. Shows person-to-person spread.	
Median-split/subj	A transductive rule: within one subject, call the top half of probabilities "Task". Needs the whole recording; not a strict accuracy.	
A9. Statistics terms
Term	Meaning
p-value	Probability of seeing a result at least this extreme if there were truly no effect. Small (< 0.05) = unlikely by chance.
Welch t-test	Compares two group means without assuming equal variances.
Wilcoxon signed-rank (paired)	Non-parametric test for paired differences (Task vs Rest for the same subject).
Mann-Whitney U	Non-parametric test comparing two independent groups (good vs bad counters).
Bonferroni correction	Multiplies p-values by the number of tests (5 regions) to control false positives. p(bonf).
n	Number of samples (here 36 subjects).
PART B. SLIDE-BY-SLIDE SCRIPT
SLIDE 1: Title, "EEG Mental Arithmetic: Rest vs Task"
1. What is on the slide
Element	Text
Title	EEG Mental Arithmetic: Rest vs Task
Subtitle	Can a machine tell a resting brain from a calculating brain? A full pipeline from raw EEG to leave-one-subject-out testing.
Card 1	36 subjects
Card 2	1,076 4-second epochs
Card 3	1,076 features per epoch
Card 4	73.7% LOSO balanced accuracy
2. Script (about 1.5 minutes)
"Good morning everyone. Today I'll show you a complete machine-learning project that answers one question: can a computer look at 4 seconds of brain-wave recording and tell whether the person was resting or doing mental arithmetic?

The title says Rest versus Task. 'Rest' means the person sits quietly. 'Task' means the person is mentally subtracting numbers. EEG, which stands for electroencephalography, records tiny electrical signals from the scalp using 19 electrodes.

Let me walk through the four numbers on the slide because they frame the whole talk. We have 36 subjects, each recorded twice, once at rest and once during the task. After cleaning, we have 1,076 epochs: an epoch is a 4-second slice of signal, and we keep exactly the same number of Rest and Task slices, 538 of each. From every slice we compute 1,076 features, which are numbers that describe the signal: its power in different rhythms, its complexity, how connected different brain areas are, and so on. It is a coincidence that the number of features equals the number of epochs.

The last card is the most important: 73.7%. That is the balanced accuracy when the model is tested on a person it has never seen before. That is our honest headline number. Chance is 50%, so the model is clearly learning something real, but it is far from perfect, and I'll explain why in this talk."

3. How, why, why not
Why 4-second epochs? Four seconds contains at least 2 full cycles even of the slowest band edge (0.5 Hz needs 2 s per cycle). It gives enough data for stable spectral and entropy estimates while still giving many epochs per subject. Shorter windows make frequency estimates noisy; longer windows give too few samples.
Why equal Rest and Task counts? If one class had more epochs, a model could score well by guessing the larger class. Truncating Rest to the Task count makes chance exactly 50%, so every percentage is easy to read.
Why report "balanced accuracy"? It averages the success rate on each class. Here it equals ordinary accuracy because the classes are equal, but using the balanced version is the safe habit.
Why a "LOSO" headline instead of the hold-out number? The notebook itself states that LOSO is the headline because the 3-subject hold-out is too small. Slide 9 and Slide 10 explain this.
4. Deep analysis
1,076 epochs from 36 subjects means roughly 30 epochs per subject (about 15 Rest + 15 Task; Subject00 has exactly 15 + 15). The "sample size" for generalising to new people is 36, not 1,076, because epochs from one person are highly similar.
A rough uncertainty on the headline: the per-subject standard deviation is 0.166 for SVM-RBF. The standard error of the mean over 36 subjects is about 0.166 / 6 = 0.028, so a rough 95% interval is about 0.74 ± 0.055, i.e. roughly 0.68 to 0.79. This is an approximation (it treats subjects as independent and ignores tuning optimism) but shows the true accuracy for new people could plausibly be anywhere in that range.
5. Likely questions
"Is 73.7% good?" It is well above 50% chance and consistent with the difficulty of cross-subject EEG classification using only 19 channels, but it is not good enough for a reliable real-world device. It is a research-grade result.
"Why 1,076 features and 1,076 epochs?" Pure coincidence: 56 features x 19 channels = 1,064, plus 12 connectivity features = 1,076; the epoch count is a separate result of cleaning.
SLIDE 2: The project, what, why and how
1. What is on the slide
Left column, three cards

Card	Text
WHAT	Label each 4-second EEG window as Rest or Mental-arithmetic Task.
WHY	Reading mental workload from the brain supports workload monitoring and brain-computer interfaces.
DATA	36 subjects, 19 scalp electrodes, one Rest and one Task recording each.
Right column, six numbered pipeline steps

Preprocess: filter, ICA, AutoReject
Extract 1,076 features per epoch
Z-score within each subject
LOSO test of 17 models and ensembles
Explain the model with SHAP
Hold-out test and robustness check
2. Script (about 2 minutes)
"This slide is the map of the whole project. On the left: what, why, and the data. On the right: the six steps.

What. The task is binary classification. Each 4-second window gets a label, Rest or Task.

Why. Mental workload leaves a signature in the brain's electrical rhythms. If we can detect it automatically, we could build systems that monitor how hard someone's brain is working, for example for safety-critical jobs, or use it as a control signal in a brain-computer interface, which is a system where brain activity controls a computer.

Data. 36 volunteers, 19 electrodes placed using the international 10-20 system, one recording at rest, one during mental arithmetic.

Now the six steps. One, preprocessing: raw EEG is full of non-brain noise, so we filter it, remove blink and muscle components with ICA, and drop bad windows with AutoReject. Two, feature extraction: we turn each window into 1,076 numbers. Three, we z-score each subject separately, which removes the fact that some people just have bigger signals. Four, we test 17 models and ensembles using leave-one-subject-out validation, meaning we train on 35 people and test on the 36th, and repeat for everyone. Five, we open the black box with SHAP to see which features matter. Six, we check on three completely untouched subjects, and we ask the hardest question: is the model reading the brain, or reading eye and muscle artifacts?"

3. How, why, why not (design decisions)
Decision	Why	Why not the alternative
Features + classical ML instead of deep learning on raw EEG	With only 36 subjects and about 1,000 epochs, deep networks would overfit; hand-crafted features are interpretable and work with small data.	Deep nets need far more data; they are also harder to explain with SHAP.
LOSO instead of random train/test split	Epochs of one person are near-duplicates; a random split lets the model "recognise the person", inflating accuracy.	Random splits give unrealistically high scores that do not predict performance on a new person.
Caching with a config hash (Section 12)	The hash of preprocessing settings + band definitions + feature version names each cache file, so changing any setting recomputes automatically.	Plain file names could silently reuse stale features after a setting change.
Parallel processing (ProcessPoolExecutor, one worker per subject)	ICA + AutoReject + non-linear features are slow per subject; subjects are independent.	Serial processing would take much longer.
Hold-out of subjects 01, 22, 35 fixed in advance	A fixed, never-touched test set cannot be tuned against.	Choosing hold-out subjects after seeing results would be cherry-picking.
4. Deep analysis
The notebook is organised to defeat leakage. Imputation, variance filtering, scaling and feature selection are all inside the pipeline and fitted on the training subjects of each fold. The one step that is not fitted on training only is the per-subject z-scoring, which uses a subject's own rows (both Rest and Task, no labels). The notebook flags this explicitly: no label leakage, but a deployed system could only use it on a recording that contains both conditions.
The "optimism" caveats are listed in the notebook: hyperparameters in Section 14 were tuned with grouped 5-fold CV on the same 36 subjects that LOSO then evaluates, and the "top-3 ensemble" was chosen after looking at LOSO AUC. Section 15 (hold-out) avoids both.
5. Likely questions
"Why not just use more subjects?" The dataset has 36; that is the limit of this public dataset. More subjects is the single biggest improvement available.
"Could this run in real time?" The feature extraction (especially entropy and fractal measures) is computationally heavy, so a real-time version would need a trimmed feature set. This was not tested here.
SLIDE 3: Cleaning the signal and building features
1. What is on the slide
Left card: "PREPROCESSING, 6 STEPS"

Keep 19 real electrodes. Drop A2-A1 and ECG channels.
Average reference. Removes noise shared by all channels.
1-45 Hz band-pass + 50 Hz notch. Drops drift, muscle and mains hum.
Resample to 128 Hz. Faster, keeps all brain bands.
ICA, 15 components. Removes blink and muscle parts (max 4).
4 s epochs + AutoReject. Drops bad windows, balances Rest and Task.
Right: bar chart "Features per family (total 1,076)"

Family	Count
Frequency	418
Wavelet	247
Time-domain	209
Nonlinear	190
Connectivity	12
Right text: "56 features per channel x 19 channels = 1,064, plus 12 connectivity features." and "Then every feature is z-scored within each subject to remove person-to-person baseline."

2. Script (about 3 minutes)
"Raw EEG is like a recording of a conversation in a noisy cafe. Before we can analyse the speech, we have to remove the noise. The left card shows six cleaning steps.

Step 1. The dataset's file headers label two non-brain channels as EEG: the ear reference A2-A1 and a heart channel. We match every channel name against the standard 10-20 montage and drop anything that does not match. That leaves 19 true scalp electrodes.

Step 2. Average reference. EEG voltages are always measured relative to something. We subtract the average of all channels from each channel, so no single electrode acts as the reference and noise common to all electrodes is cancelled.

Step 3. A band-pass filter from 1 to 45 Hz. Below 1 Hz is slow drift from sweat and movement. Above 45 Hz is mostly muscle noise. We use a zero-phase FIR filter so the timing of events is not shifted. We also add a notch at 50 Hz because the electrical mains hum at 50 Hz is not fully removed by the 45 Hz low-pass edge.

Step 4. Resample to 128 Hz. Since we only keep up to 45 Hz, we need only above 90 samples per second; 128 is a safe, efficient choice that still represents up to 64 Hz.

Step 5. ICA. Independent Component Analysis unmixes the 19 channels into 15 independent sources. Some sources are clearly eye blinks, because they dominate Fp1 and Fp2, the forehead electrodes. Others look like muscle. We remove the blink-like ones first, then the most muscle-like ones, but never more than four. We fit one ICA on Rest and Task together so both recordings are cleaned in exactly the same way.

Step 6. We cut the signal into 4-second epochs, then AutoReject throws out or repairs windows with excessive noise. Finally we truncate so each subject has the same number of Rest and Task epochs.

Now the chart. From each channel in each epoch we compute 56 features in four families: time-domain, frequency-domain, wavelet and non-linear. Times 19 channels gives 1,064. We add 12 connectivity features between four electrode pairs. Total, 1,076. Finally, each feature is z-scored within each subject. That removes person-to-person differences like skull thickness and electrode contact that have nothing to do with the task."

3. Picture explanation: the bar chart
Axes. Horizontal bars; each bar = number of features in that family. Total = 418 + 247 + 209 + 190 + 12 = 1,076.
Frequency (418) = 22 per channel x 19. Absolute, log and relative power in 5 bands (15), plus spectral edge 95, peak frequency, spectral entropy (3), plus engagement index and three ratios (4).
Wavelet (247) = 13 per channel x 19. Energy and relative energy for 6 wavelet levels (12) + wavelet entropy (1).
Time-domain (209) = 11 per channel x 19. Mean, std, variance, skewness, kurtosis, RMS, peak-to-peak, zero-crossing rate, line length, Hjorth mobility, Hjorth complexity.
Nonlinear (190) = 10 per channel x 19. Permutation, SVD, approximate and sample entropy; Petrosian, Katz and Higuchi fractal dimension; DFA; number of zero crossings; Lempel-Ziv complexity.
Connectivity (12) = 4 pairs x 3. Correlation, theta-alpha PLV, theta-alpha coherence for Fp1-Fp2, F3-F4, Fz-Pz, F7-F8.
Take-away of the chart. The feature set is dominated by frequency-based and wavelet descriptions of single channels; connectivity is deliberately tiny (12 of 1,076, about 1%). That matters later: the SHAP charts show connectivity contributes almost nothing.
4. How, why, why not (every preprocessing choice)
Choice	Why	Why not the alternative
Drop A2-A1 and ECG	They are not scalp EEG; keeping them would inject heartbeat and reference signals.	Keeping them would contaminate the average reference and the features.
Average reference	With a 19-electrode cap and no clean single reference, the common average is standard and unbiased.	A single-electrode reference makes one channel's noise appear in all.
1 Hz high-pass	ICA works much better on data without slow drift.	A lower cutoff (0.1 Hz) keeps drift that disturbs ICA.
45 Hz low-pass	Matches the gamma band edge (30-45 Hz) and the recording's own 45 Hz low-pass.	Going to 70+ Hz would add more muscle noise.
50 Hz notch	The 45 Hz FIR transition band does not fully remove mains hum.	Skipping it leaves leaks near the top of the band.
128 Hz	Nyquist = 64 Hz > 45 Hz, with fewer samples to process.	Higher rates add cost without information.
ICA 15 components	"< 18 = rank after average reference of 19 channels." Average referencing removes one degree of freedom (rank falls to 18), and asking for as many components as the rank can make ICA unstable.	19 components would exceed the data's rank; far fewer would not separate sources.
Fp1/Fp2 as EOG proxies	The dataset has no eye channel; forehead electrodes see blinks best.	No proxy would mean no automatic blink detection.
Max 4 components removed	A noisy recording must not lose all its signal.	Unlimited removal could delete real brain activity.
One ICA on Rest + Task together	Both recordings are cleaned by the same unmixing, so the cleaning cannot create a Rest/Task difference.	Separate ICAs per recording could remove different things and introduce a spurious difference.
AutoReject	Data-driven thresholds per channel instead of one fixed microvolt cut-off.	Fixed thresholds are too strict for some people and too loose for others.
Truncate Rest to Task length	Equal class counts, chance = 50%.	Unequal classes complicate every metric.
Z-score per subject	Removes baseline differences between people (skull thickness, electrode contact, arousal).	Without it, between-person variation swamps the Rest/Task signal.
5. Deep analysis
ICA with 19 electrodes is a limited cleaner. The notebook says so itself. Real artifact removal usually uses 64+ channels and dedicated EOG/EMG electrodes. That is why Section 16 exists and why Slide 10 says "not proven pure brain signal".
The wavelet level labels need care. With 128 Hz sampling and wavedec(level=5), the coefficient sets are: approximation 0-2 Hz (L0), and detail levels 2-4 (L1), 4-8 (L2), 8-16 (L3), 16-32 (L4), 32-64 Hz (L5). So C3_wavelet_energy_L4 (a top hold-out feature) means energy in the roughly 16-32 Hz (beta) range at C3, and P3_wavelet_energy_L3 means roughly 8-16 Hz (alpha/low-beta) at P3.
Welch settings. 256-sample segments at 128 Hz = 2 s windows, giving 0.5 Hz frequency resolution. That is adequate for delta (0.5-4 Hz) but means band edges are coarse.
Feature redundancy. Many of the 1,076 features are near-duplicates (power vs log power vs relative power, ratios and their inverses). This is why the pipeline has variance filtering, scaling, regularised or tree-based models, and (in the top-k variants) ANOVA feature selection.
6. Likely questions
"Why did you not use deep learning / CNNs?" Too little data (36 people); features + classical models are the right tool and are explainable.
"Is 15 ICA components enough?" It is a compromise forced by the rank (18). It cannot fully clean 19-electrode data, which is why artifact robustness is tested.
SLIDE 4: EDA, where does theta power change?
1. What is on the slide
Element	Content
Left image	Grid of topomaps: "Task - Rest log power (theta band)" for all 36 subjects
Left caption	All 36 subjects: Task minus Rest theta power
Right image (top)	Subject 0 topomaps: Rest (theta), Task (theta), Task - Rest (theta)
Right caption	Subject 0: Rest, Task, Task minus Rest
Bullet 1	Red means more theta in Task, blue means less.
Bullet 2	Subject 0: no region is significant after Bonferroni correction (all p of 0.78 or more).
Bullet 3	The pattern differs from person to person, so there is no single universal brain map.
2. Script (about 2.5 minutes)
"Before modelling, always look at the data. Here we ask a simple question: which parts of the head show more or less theta activity during the arithmetic task compared with rest? Theta is the 4-8 Hz rhythm often linked to mental effort.

A topomap is a top-down view of a head, nose at the top. Colour is interpolated between the 19 electrodes. In the single-subject picture on the right, the first two heads show Rest and Task on the same colour scale, so you can compare them directly. The third head shows the difference: Task minus Rest, on a blue-to-red scale centred on zero. Red means more theta during the task, blue means less.

This is Subject 0. You can see some red and blue patches, but when we test each brain region statistically, with a Bonferroni correction because we test five regions, nothing is significant. The smallest corrected p-value is 0.78. So for this one person, the visible pattern could easily be random.

Now the big grid on the left: all 36 subjects. If everybody's brain responded the same way, all 36 maps would look alike, say all red at the front. They do not. Some subjects show red at the front, some at the back, some mostly blue. That tells us something crucial for the modelling: there is no single universal spatial pattern, so the classifier cannot rely on one 'theta hot spot'. It has to combine many weak, person-varying cues."

3. Picture-by-picture explanation
Picture 1: Subject 0 topomaps (3 heads + 2 colour bars)

Head 1 (Rest, theta) and Head 2 (Task, theta): log band power (log V²) on a shared viridis (purple-low to yellow-high) scale. Same scale means colours are directly comparable.
Head 3 (Task - Rest): difference of the log powers on a diverging red-blue scale. Zero = no change (white-ish).
What it shows: this subject's task and rest maps look broadly different in some areas, but the statistics (below) show the differences are within noise.
What it does not show: causation, or anything about other subjects.
Subject 0 regional statistics (Welch t-test, Bonferroni, printed in the notebook)

Region	Mean diff (Task - Rest)	p	p (Bonferroni)
Occipital	-0.179	0.1558	0.7791
Frontal	-0.054	0.6683	1.0000
Temporal	-0.047	0.6382	1.0000
Central	+0.034	0.7306	1.0000
Parietal	+0.016	0.8850	1.0000
None reaches 0.05. Sample size is only 15 Rest and 15 Task epochs for this subject, so power is low.

Picture 2: 6 x 6 grid of Task - Rest theta maps (36 small heads, each labelled by subject, shared colour bar)

Each small head is one subject's difference map.
What it shows (approx., read from the plot): both red and blue regions appear in many subjects, in different locations; no consistent front-to-back or left-to-right pattern is obvious across the grid.
What it implies: group-level theta effects exist (shown on Slide 5), but they are small and spatially inconsistent. The notebook's own limitations section warns: "avoid claiming a universal spatial pattern".
4. How, why, why not
How the map is made: compute_band_power_per_channel computes Welch PSD per epoch and channel, integrates over 4-8 Hz, averages epochs per condition; plot_subject_topomap takes log(power) for Rest and Task and subtracts. head_sphere_for sizes the head outline from electrode positions so the interpolation is clipped exactly at the drawn head.
Why log power? Power is strongly right-skewed. Log makes differences symmetric (a doubling and a halving look equal and opposite).
Why a shared colour scale for Rest and Task? Different scales would hide real differences.
Why Bonferroni? Five regions were tested; without correction, you expect about 1 false positive in 20 tests.
Why Welch t-test? It does not assume equal variances in Rest and Task epochs.
Why not draw conclusions from one subject? 15 vs 15 epochs is too few; epochs are also not independent (neighbouring windows are correlated).
5. Deep analysis
The topomap is an interpolation. With only 19 electrodes, the smooth colours between electrodes are estimates. Fine detail is not real.
A visible colour difference is not evidence. Subject 0's third head looks dramatic, yet corrected p-values are 0.78 to 1.0. This is a good teaching moment: statistical testing prevents over-reading pretty pictures.
Why heterogeneity matters for modelling. If the effect location differs per person, then a model trained on other people's "theta at the front" cannot generalise. This partly explains why cross-subject accuracy (about 74%) is far below what within-subject classifiers often achieve.
6. Likely questions
"Why theta?" It is the band most commonly linked to working memory and mental effort, especially frontal. The notebook uses it for EDA; the models use all bands.
"Is red 'more brain activity'?" Red means more theta power. Higher theta power is not simply "more activity"; it is one specific rhythm.
SLIDE 5: EDA, feature redundancy and a small theta effect
1. What is on the slide
Element	Content
Left image	Heatmap: "Correlation of Fz band powers and ratios"
Left text	Heatmap (Fz). Band powers and their logs move together, while ratios move opposite. Many features are redundant, so we regularise and select.
Right image	Box plot: "Frontal theta: Rest vs Task (one point per subject, n=36)" with Bad counter and Good counter groups
Right text	Frontal theta (one dot per subject). Rises slightly in Task: median +0.0102, paired Wilcoxon p = 0.0105 (n = 36). Good vs bad counters do not differ (p = 0.54).
2. Script (about 2.5 minutes)
"Two more exploratory pictures, answering two questions. First: how much do our features overlap? Second: is there any simple, group-level difference between Rest and Task?

The heatmap. Take one electrode, Fz, on the midline of the forehead. We have many features for it: power in each band, the log of that power, relative power, and band ratios. The heatmap shows how strongly each pair is correlated. Dark red means they move together, dark blue means they move in opposite directions. The strong red squares show, for instance, that delta power and log delta power are almost the same information. The blue patches show, for example, that theta/alpha and alpha/theta are mirror images. This tells us the feature set is highly redundant. That is not a mistake; it is a reason the models need regularisation or feature selection, so they do not chase duplicated noise.

The box plot. Now the group-level question. For each subject we averaged frontal theta relative power separately for Rest and Task, so each dot is one person. There are 36 dots per condition, one per subject, because epochs from the same person are not independent. The boxes show the spread. Task is slightly higher than Rest. The paired Wilcoxon test, which compares each person with themselves, gives p equals 0.0105, so the increase is statistically significant. But look at the size: the median change is only about 0.01 in relative power, tiny compared with the person-to-person spread. This is a small effect.

Finally, we split subjects into good and bad counters, based on whether they actually counted well. If the arithmetic effort were driving the effect, you might expect good counters to show a larger change. The Mann-Whitney test gives p equals 0.54: no evidence of a difference. So we cannot say the effect depends on counting quality."

3. Picture-by-picture explanation
Picture 1: Correlation heatmap of Fz power and ratio features

Rows/columns: Fz delta/theta/alpha/beta/gamma power, log power, relative power, and the ratios theta/alpha, theta/beta, alpha/theta (about 18 variables).
Colour: Pearson correlation from -1 (blue) to +1 (red); the upper triangle is masked to avoid showing everything twice.
Pattern (approx., read from the plot): each band's power and log-power form a strong red pair; the ratio features at the bottom show blue (negative) relations with each other and with some band powers, exactly as expected because alpha/theta is the inverse of theta/alpha.
Implication: high multicollinearity.
Picture 2: Box plot with dots, "Frontal theta relative power" Rest vs Task by counting quality

x-axis: Condition (Rest, Task). Colour: Bad counter (orange/brown) vs Good counter (green). y-axis: frontal theta relative power (subject mean), values roughly 0.08 to 0.25 (approx., read from the plot).
Pattern: medians in both groups are a little higher in Task; boxes overlap heavily; some dots are far above the others (outlier subjects).
Statistics printed in the notebook:
Paired Wilcoxon, Task vs Rest, n = 36: median change = +0.0102, p = 0.0105.
Mann-Whitney, good (n = 26) vs bad (n = 10) counters on the Task-Rest change: p = 0.5365.
4. How, why, why not
Why one point per subject? Treating all epochs as independent ("pseudo-replication") would give hugely inflated significance. Averaging per subject keeps n = 36 honest.
Why a paired test? Task and Rest come from the same person, so subtracting each person's own baseline removes between-person variation.
Why Wilcoxon not a t-test? It does not assume the differences are normally distributed, and with n = 36 and visible outliers that is safer.
Why Mann-Whitney for counters? The two groups are different people (independent), unequal sizes (26 vs 10), and non-normal.
Why show the heatmap? To justify regularisation, the variance filter and feature selection.
5. Deep analysis
Effect size in context. The box plot shows typical frontal theta relative power around 0.14 to 0.17. A change of +0.0102 is therefore roughly a 6-7% relative increase (approx., from the plot's scale). Statistically significant, but small compared with differences between people.
Significance does not mean usefulness. A single feature with this overlap could not classify well; the classifier needs many features.
The counter-quality null result (p = 0.54, with only 10 bad counters) has low power; it is "no evidence of a difference", not "evidence of no difference".
Possible confound. Theta also rises with fatigue, drowsiness and time-on-task. Because Rest and Task are separate recordings, the theta increase could partly reflect time-in-session. The notebook says so in its limitations.
6. Likely questions
"If theta is only slightly higher, why can the model get 74%?" Because it combines hundreds of weak cues (other bands, complexity, wavelet energies) across many channels; no single feature needs to be strong.
"Why is the correlation heatmap only for Fz?" For readability; Fz is a classic frontal-midline site for theta effects. The same redundancy exists at every channel.
SLIDE 6: Model comparison, all trees vs the rest
1. What is on the slide
Element	Content
Left image	Horizontal bar chart "LOSO: model comparison" (balanced accuracy, error bars = SD across subjects)
Left caption	LOSO balanced accuracy, all 17 models (bars) with SD across subjects
Table	Tree models: DecisionTree, ExtraTree, Bagging, RandomForest, ExtraTrees, GradBoost, HistGradBoost, XGBoost, AdaBoost with type, balanced accuracy and AUC
Bullet 1	Single trees overfit (64-65%). Averaging or boosting adds 7-9 points.
Bullet 2	Best non-tree: SVM-RBF 0.737, AUC 0.820.
Bullet 3	All top models sit within about 1 point.
Table values (all from the notebook)

Tree model	Type	Bal. acc	AUC
DecisionTree	single tree	0.641	0.703
ExtraTree	single tree	0.654	0.715
Bagging	ensemble	0.708	0.782
RandomForest	ensemble	0.702	0.784
ExtraTrees	ensemble	0.724	0.788
GradBoost	boosting	0.728	0.809
HistGradBoost	boosting	0.727	0.807
XGBoost	boosting	0.735	0.816
AdaBoost	boosting	0.736	0.812
2. Script (about 3.5 minutes)
"Now the main experiment. We compared 12 individual models and 5 ensembles, 17 in total, using leave-one-subject-out testing. Every model sits inside exactly the same pipeline: fill missing values with the median, remove constant features, standardise, select the 150 features that best separate the classes, then classify. All of those steps are fitted on the training subjects only, inside each fold, so the test subject never influences anything.

The bar chart on the left sorts models from best at the top to worst at the bottom. Notice two things. First, the top bars are almost equal, around 0.73 to 0.74. Second, the black error bars, which are the standard deviation across subjects, are very long, about 0.17 to 0.2. People differ enormously.

You asked about trees, so here is the full tree family on the right. I'll explain each in plain words.

A Decision Tree is a single flowchart of yes/no questions on features, for example 'is T6 fractal dimension above this value?' It is easy to read but unstable; change the training data slightly and you get a different tree. It scores 0.641. An Extra Tree is the same, but split points are chosen randomly. 0.654.

Bagging trains 300 trees, each on a random 80% of the epochs and half the features, and averages their votes. Averaging cancels each tree's quirks. 0.708. A Random Forest adds extra randomness: at each split only a random subset of features is considered. 0.702. Extra Trees, the forest of randomised-threshold trees, reaches 0.724.

Then the boosting family. Instead of independent trees, each new tree concentrates on the mistakes of the previous ones. AdaBoost reweights misclassified epochs: 0.736. Gradient Boosting fits each new tree to the remaining error: 0.728. Histogram Gradient Boosting is a faster binned version: 0.727. XGBoost is a regularised gradient booster with shallow trees: 0.735.

The lesson: single trees are clearly worst at 64 to 65%. Averaging or boosting trees adds about 7 to 9 points. But the best non-tree model, the SVM with an RBF kernel, still ties or slightly beats them at 0.737. And look at the spread: the top models are all within about a point of each other, which is much smaller than the person-to-person spread. So we should not claim one clearly beats another."

3. Picture explanation: LOSO model comparison bar chart
Y-axis: 17 models (sorted by per-subject mean balanced accuracy). X-axis: balanced accuracy 0.3 to 1.0 with a dashed line at 0.5 (chance).
Teal bars: mean over 36 subjects. Grey whiskers: standard deviation across subjects.
Order from the printed table (per-subject mean balanced accuracy):
Rank	Model	Family	Bal. acc	AUC (pooled)	SD
1	Ensemble: top-3 by LOSO AUC (post-hoc, optimistic)	Ensemble	0.743	0.826	0.177
2	Ensemble: SVM + boosting	Ensemble	0.738	0.822	0.194
3	SVM-RBF	SVM	0.737	0.820	0.166
4	AdaBoost	Boosting	0.736	0.812	0.192
5	Ensemble: SVMs	Ensemble	0.735	0.816	0.166
6	XGBoost	Boosting	0.735	0.816	0.198
7	LogReg-L2	Linear	0.733	0.817	0.168
8	SVM-Linear	SVM	0.732	0.812	0.161
9	Ensemble: all models	Ensemble	0.732	0.814	0.196
10	GradientBoosting	Boosting	0.727	0.809	0.193
11	HistGradBoost	Boosting	0.727	0.807	0.194
12	Ensemble: tree models	Ensemble	0.726	0.807	0.202
13	ExtraTrees	Tree ensemble	0.724	0.788	0.206
14	Bagging (trees)	Tree ensemble	0.708	0.782	0.206
15	RandomForest	Tree ensemble	0.701	0.784	0.215
16	ExtraTree (single)	Tree	0.654	0.715	0.165
17	DecisionTree	Tree	0.641	0.703	0.174
(Pooled balanced accuracy values differ in the third decimal for some models; for example RandomForest pooled is 0.702 vs per-subject mean 0.701.)

4. The tree family in depth: how each works, why it behaves as it does, why not
Settings printed or tuned in the notebook

Model	Settings	How it works	Why it performed that way
DecisionTree	max_depth = 5, min_samples_leaf = 10, class_weight balanced	One flowchart. Each node picks the feature and threshold that best separates classes.	Fits the training subjects' idiosyncrasies; high variance; with ~1,000 noisy epochs, a single tree is unstable. Lowest score, 0.641.
ExtraTree (single)	max_depth = 6, min_samples_leaf = 10	One tree whose thresholds are drawn at random.	Slightly smoother than a CART tree but still one model; 0.654.
Bagging (trees)	300 trees, depth 6, 80% samples, 50% features	Train many trees on random subsets; majority vote.	Averaging reduces variance: +6.7 points over one tree. 0.708.
RandomForest	default 500 trees, leaf 3, sqrt features; tuned 300 trees, leaf 2, sqrt features	Bagging + random feature subset at each split (de-correlates trees).	Many correlated, noisy features: random subsets help, but deep independent trees still fit person-specific noise. 0.702.
ExtraTrees	tuned 600 trees, leaf 2, max_features 0.3	Forest with random thresholds.	Extra randomness acts like regularisation; best pure forest, 0.724.
AdaBoost	300 estimators, lr 0.5	Sequentially re-weights epochs that earlier weak learners got wrong.	Focuses on hard epochs; 0.736, tied for best single model on bal. acc.
GradientBoosting	tuned 400 trees, depth 4, lr 0.1, subsample 0.6	Each tree fits the gradient of the loss (remaining error).	0.728; subsampling provides regularisation.
HistGradBoost	tuned depth 4, lr 0.05, L2 reg 0.1	Gradient boosting on binned (histogram) features; fast.	0.727; similar to GB.
XGBoost	tuned 200 trees, depth 3, lr 0.1, colsample 0.5 (also reg_lambda 3, subsample 0.8 in hold-out builds)	Regularised gradient boosting with shallow trees.	0.735; shallow trees plus L2 and column sampling resist overfitting.
Important note: the notebook has no tree diagrams. No plot_tree, no Graphviz export. "All trees" in this deck means all tree-based models. If someone asks to see one tree, say: "The notebook does not render a tree diagram; I can add one with sklearn.tree.plot_tree on the DecisionTree."

5. Why trees lose and why the SVM leads (analysis)
High dimensionality, small sample. Roughly 1,000 epochs but 1,076 features (150 after selection). Trees split on one feature at a time and are very sensitive to which subjects are in training.
Many weak, correlated signals. The information is spread across hundreds of small effects. A margin-based model (SVM, logistic regression) uses all features smoothly. A tree must pick a few and discards the rest.
Person-specific noise. Deep trees memorise training subjects' quirks. Ensembles average that away, which explains the +7 to +9 point jump from single to ensemble.
Boosting with shallow trees (depth 3 to 4) behaves like an additive model of many small contributions, which is why it reaches the SVM's level.
But the differences between the top models are not meaningful. With SD about 0.17 to 0.2 across 36 subjects, the standard error of each mean is about 0.03. A 0.01 difference between SVM-RBF and XGBoost is far inside that. The honest statement is "SVM, linear models and boosting perform equivalently; single trees and bagged/random forests perform somewhat worse".
Tree models vs "all non-tree models". The ensemble of tree models (0.726) is lower than the ensemble of SVMs (0.735). The best overall is the top-3 ensemble (0.743), but it was chosen after seeing LOSO AUC, so it is optimistic.
6. How, why, why not (general modelling decisions)
Decision	Why	Why not the alternative
Median imputation	Robust to outliers; missing values arise from failed entropy computations.	Mean imputation is pulled by outliers; dropping rows loses data.
Variance filter 1e-8	Removes constant columns that carry no information and break scaling.	Keeping them wastes computation.
StandardScaler	SVM, logistic regression and ANOVA need comparable feature scales.	Unscaled features dominate by magnitude.
ANOVA top-k = 150	Cuts noise and speeds training, done inside each fold.	Selecting features on all data before cross-validation causes leakage.
class_weight='balanced'	Safe default even though classes are equal.	Unneeded but harmless here.
probability=True for SVM	Needed for AUC and for ensemble averaging.	Without it, no ROC curve or probability averaging.
Grouped tuning (5-fold GroupKFold, 12 random draws)	Keeps subjects together while tuning hyperparameters.	Ordinary CV would leak subjects across folds.
Report both pooled and per-subject-mean	Pooled shows overall epoch accuracy; per-subject-mean treats each person equally (the deployment-relevant view).	One number can hide big between-subject differences.
Tuned grouped-CV balanced accuracies (printed): SVM-Linear 0.738 (C = 0.03), SVM-RBF 0.745 (gamma 0.0003, C 30), RandomForest 0.733, ExtraTrees 0.733, GradientBoosting 0.747, HistGradBoost 0.736, XGBoost 0.742. The notebook warns this tuning is on the same subjects LOSO later evaluates, leaving a small optimism.

7. Likely questions
"Why is Decision Tree the worst but easiest to explain?" Interpretability and accuracy trade off: a depth-5 tree uses at most 31 splits, far too few to capture hundreds of weak cues.
"Why SVM and not XGBoost as the final model?" The final hold-out model was chosen by grouped CV on training subjects only: SVM-RBF (all features) had the highest cross-validated balanced accuracy (0.746) and AUC (0.837), ahead of XGBoost (0.739 / 0.804). Not chosen by hold-out score.
"Is boosting better than bagging?" Here, yes by about 2-3 points, but see the standard-error caveat.
SLIDE 7: LOSO results, ROC curves and confusion matrices
1. What is on the slide
Element	Content
Left image	4-panel figure: top row "RandomForest: ROC per held-out subject" and "XGBoost: ROC per held-out subject"; bottom row "RandomForest: pooled confusion matrix (LOSO)" and "XGBoost: pooled confusion matrix (LOSO)"
Table	Model, Rest P / R, Task P / R, Accuracy
Row RF	RandomForest: 0.70 / 0.70, 0.70 / 0.70, 0.70
Row XGB	XGBoost: 0.74 / 0.72, 0.73 / 0.75, 0.74
Text 1	ROC: pale lines are single subjects, the dark line is the mean. The wide spread shows people differ a lot (SD about 0.2).
Text 2	Confusion matrix: errors are balanced, no bias toward Rest or Task.
Text 3	XGBoost beats RandomForest by about 4 points.
2. Script (about 3 minutes)
"The previous slide gave one number per model. This slide opens it up for two models, Random Forest and XGBoost, so you can see how they are right and wrong.

ROC curves, top row. An ROC curve answers: if I move the decision threshold from strict to lenient, how does the catch rate trade off against false alarms? The vertical axis is the true-positive rate: the share of real Task epochs correctly flagged. The horizontal axis is the false-positive rate: the share of Rest epochs wrongly flagged as Task. A perfect model hugs the top-left corner. A useless model follows the diagonal. Each pale, stepped line is one held-out subject, the dark blue line is the mean, and the shaded band is plus or minus one standard deviation. Notice how widely the pale lines fan out: some subjects are decoded almost perfectly, a few are barely better than chance. That fan is the person-to-person variation, with a standard deviation of about 0.2.

Confusion matrices, bottom row. These pool all 1,076 epochs. Rows are the truth, columns are the prediction. The dark diagonal cells are the correct calls. Both models have errors in both off-diagonal cells in similar numbers, so there is no systematic bias toward calling things Rest or Task.

The table confirms it in numbers. Random Forest: precision and recall are 0.70 for both classes. XGBoost: Rest precision 0.74, recall 0.72, Task precision 0.73, recall 0.75, overall accuracy 0.74. XGBoost is about four points better. Precision is: of those I called Task, how many truly were. Recall is: of all truly Task, how many did I find."

3. Picture explanation
Panels 1 and 2: ROC per held-out subject

Axes: x = false positive rate (0 to 1), y = true positive rate (0 to 1), dashed diagonal = chance.
Pale blue stepped lines: the 36 individual ROC curves (steps because each subject has only about 30 epochs).
Dark blue line: mean ROC across subjects. Legend gives the mean AUC and its SD (the per-subject mean AUCs printed are 0.756 for RandomForest and 0.779 for XGBoost, from the notebook table).
Shaded band: ±1 SD across subjects.
Meaning: AUC 0.78 means that for a random pair of (Task epoch, Rest epoch) from a new person, the model ranks the Task epoch higher about 78% of the time on average.
Why the lines are stepped and jagged: with about 30 epochs per subject, the curve can only change in about 30 steps. Smooth ROCs need many more samples.
Panels 3 and 4: pooled confusion matrices

Layout: rows = true label (Rest, Task); columns = predicted label (Rest, Task). Dark blue = high count.
Pooled support: 538 Rest and 538 Task epochs, so each row sums to 538.
Reading from the classification report (printed): Random Forest correctly finds 70% of each class; XGBoost finds 72% of Rest and 75% of Task. The exact cell counts appear in the figure (not printed as text in the notebook), so quote percentages from the table instead.
4. How, why, why not
Why show ROC per subject? A single pooled ROC would hide the fact that performance varies greatly by person, which is the central deployment risk.
Why pooled confusion matrices? They show systematic bias, which per-subject matrices (30 epochs) cannot show reliably.
Why both RandomForest and XGBoost? They represent the two main tree-ensemble philosophies, bagging and boosting, so the contrast is educational.
Why not show the SVM here? The SVM is covered in the hold-out analysis (Slide 9). The notebook's plot_loso_roc_cm function was called with these two names.
5. Deep analysis
Why AUC > accuracy. XGBoost AUC (pooled 0.816) is well above its accuracy (0.74). AUC measures ranking quality across all thresholds; accuracy uses a single 0.5 threshold. The gap suggests the probabilities are informative but not perfectly calibrated across subjects, i.e. each person has a different "natural threshold". This is exactly what the optional median-split rule (per-subject, transductive) tries to exploit, and it gives similar numbers (0.734 for XGBoost), so it does not rescue much.
Precision/recall symmetry. Because classes are equal and class_weight='balanced' or equal pooling is used, precision and recall are almost equal, which indicates no strong bias.
F1 and MCC (from the LOSO table). XGBoost f1_task = 0.740, MCC = 0.471; RandomForest f1_task = 0.701, MCC = 0.403. An MCC near 0.47 is "moderate" correlation between prediction and truth.
Per-subject SD. RandomForest 0.215, XGBoost 0.198. Because the SD is around 0.2, a typical new user could be anywhere from near-chance to near-perfect. A product built on this would need per-user calibration.
6. Likely questions
"Why does one subject have a straight diagonal ROC?" That subject is effectively not decodable by that model; their epochs look the same in both conditions to the classifier.
"What would a perfect ROC look like?" A line from (0,0) straight up to (0,1) then right to (1,1). Hold-out Subject22 gets AUC = 1.00 (Slide 9).
SLIDE 8: SHAP, which features drive the predictions?
1. What is on the slide
Element	Content
Image 1	Bar chart "Top 20 features (LOSO; not used for selection)" with red bars (frontal-pole/temporal channel) and teal bars (other); x-axis "mean
Image 2	Bar chart "Mean
Image 3	SHAP beeswarm "Direction and spread of each feature's effect" with colour bar Low to High
Caption 1	Unseen-subject importance. Red = frontal/temporal channels: 45% of the top 20, vs 42% of the scalp.
Caption 2	Explanation only. T6 Higuchi fractal dimension ranks first, then T5 gamma power.
Caption 3	Direction. Each dot is an epoch. Red = high value; right of zero pushes toward Task.
Bar	Complexity, gamma and wavelet features dominate. Gamma can also be muscle, so Section 16 tests brain vs artifact.
2. Script (about 3.5 minutes)
"A model that is right 74% of the time is useful, but a scientist wants to know why. SHAP is a method from cooperative game theory. It splits each prediction among the features, like dividing a prize among team members according to what each contributed. A positive SHAP value pushes the prediction toward Task, a negative one toward Rest. The size of the average absolute SHAP value tells you how important a feature is.

The left chart is the most honest one. For each fold, the model is trained on some subjects, and SHAP is computed on subjects the model has not seen. We then average. So these are the features that matter for new people. The bars are coloured: red means the feature comes from a frontal-pole or temporal electrode. Why highlight those? Because those are the places where eye movements and jaw or scalp muscle tension leave their strongest marks. The notebook counts it: 45% of the top 20 features come from those eight artifact-prone channels, while those channels make up 42% of the scalp. So they are only slightly over-represented. That is mildly reassuring, but it is not proof of clean brain signal.

The middle chart fits XGBoost on all subjects, purely for explanation. The top feature is T6 Higuchi fractal dimension, by a wide margin, followed by T5 gamma relative power, Fp1 Hjorth complexity, and alpha-power features at Fp2. In plain language: how rough and complex the waveform is at the right-back temporal site, and the share of fast gamma activity, are the strongest cues.

The right chart, the beeswarm, shows direction. Each dot is one epoch. Colour is the feature value: blue low, red high. Position left or right is its effect on the prediction. For the top feature, high values, the red dots, sit on the right, meaning a rougher waveform at T6 pushes the model toward Task.

And now the caution, which is the message of the bottom bar. Gamma power, signal roughness and Hjorth complexity measured at temporal and frontal sites can also rise with jaw clenching and scalp muscle tension. A person concentrating on arithmetic may frown or clench slightly. So we cannot say from SHAP alone that the model reads the brain. That is why the notebook has Section 16, which re-runs the model without those channels and without gamma and the fast wavelet levels."

3. Picture-by-picture explanation
Picture 1: Top 20 features, out-of-fold (OOF) mean |SHAP|

Y-axis: feature names, format <channel>_<feature>. X-axis: mean |SHAP| on unseen subjects (0 to about 0.3). Red = artifact-prone channel (Fp1, Fp2, F7, F8, T3, T4, T5, T6); teal = other channels.
Top bars (approx., read from the plot): T6_higuchi_fd (about 0.30, red), Fp2_gamma_power (about 0.28, red), P3_line_length (about 0.23, teal), P4_wavelet_energy_L3 (about 0.21, teal), T5_gamma_relative_power (about 0.19, red), then Cz_beta_power_log, C3_wavelet_energy_L4, and others.
Printed fact: 45% of the top 20 come from artifact-prone channels; 8 of 19 channels = 42% of the scalp.
Interpretation: the top two features are both from artifact-prone channels. The overall share is near what chance would give, but the very top of the ranking is dominated by red bars, so the caution is justified.
Picture 2: Mean |SHAP|, XGBoost fitted on all subjects (explanation only)

Top features (approx., read from the plot): T6_higuchi_fd (about 0.59, far ahead), T5_gamma_relative_power (about 0.35), Fp1_hjorth_complexity (about 0.32), Fp2_alpha_power_log (about 0.28), Fp2_alpha_relative_power (about 0.26), C3_wavelet_energy_L4, P3_line_length, F3_beta_power_log, Fp2_hjorth_complexity, P3_wavelet_energy_L3, T4_approximate_entropy, Cz_beta_power_log, O1_peak_to_peak, P3_hjorth_complexity, T3_approximate_entropy.
Why the numbers are larger than Picture 1: this model is fitted on all data and explained on the same data (in-sample), so the contributions look larger. That is why it is labelled "explanation only".
Feature-type reading: the list mixes complexity (fractal dimension, Hjorth, entropy), frequency-power (gamma, alpha, beta) and wavelet/line-length features. Many are at forehead (Fp1, Fp2) and temporal (T3-T6) sites.
Picture 3: Beeswarm (summary) plot

Rows: features, sorted by importance (same order as Picture 2). x-axis: SHAP value = impact on model output (about -1 to +1.2, approx., read from the plot). Colour: low (blue) to high (pink/red) feature value.
Reading: if red dots are on the right and blue on the left, higher feature value means more "Task". For T6_higuchi_fd, that is what the plot shows (approx., read from the plot).
Width of each swarm = how much that feature's effect varies between epochs.
A feature with mixed colours on both sides has a non-linear or interaction-dependent effect.
4. How, why, why not
How SHAP is computed. shap explains the XGBoost model (tree explainer): for each epoch, it measures how the prediction changes when each feature is added to different subsets of other features, then averages (a Shapley value).
Why SHAP and not plain "feature importance"? Built-in importance only says how often a feature was used in splits, not in which direction it acts, and it is biased toward features with many possible split points. SHAP gives sign, magnitude and per-epoch detail.
Why out-of-fold SHAP (Picture 1)? SHAP values on the training data describe what the model memorised. SHAP on unseen subjects describe what generalises.
Why colour by artifact-prone channel? It turns a vague worry (is this brain or muscle?) into a number you can read off the chart.
Why not use SHAP for feature selection? The notebook states the top-20 plot is "not used for selection". Using it for selection on the same data would leak.
5. Deep analysis
The 45% vs 42% comparison. 45% of the top-20 equals 9 of 20 features. 42% of channels would predict about 8.4 of 20. The excess is under one feature, so by itself there is no strong enrichment of artifact-prone channels. But importance is not uniform: the top-ranked features are heavily frontal/temporal, which matters more than the count.
Why fractal dimension and entropy rank so high. These measure signal roughness/irregularity. Muscle activity adds fast, irregular, broadband noise, which strongly raises fractal dimension and entropy. A genuine brain-state change also alters complexity. The two explanations are not separable without a clean control, which is why Section 16 matters.
Why connectivity is absent. Only 12 connectivity features exist (1% of features), and they use just four electrode pairs. Their importance share is about 1% (Slide 9 family chart).
Interpretation limits. SHAP shows what the model uses, not what the brain does. A feature can be important because it is a confound.
6. Likely questions
"What does T6_higuchi_fd actually mean?" The Higuchi fractal dimension of the signal at electrode T6 (right posterior temporal). Larger value = more irregular, "rougher" waveform.
"Why do the three charts show different feature orders?" Chart 1 is on unseen subjects (5 grouped folds); Charts 2 and 3 are in-sample from one model fitted on everyone. Rankings are similar at the top but differ lower down.
"Does a right-side dot mean Task for every feature?" No. For some features high values push toward Rest. Read colour against position per row.
SLIDE 9: Clean hold-out test, Subjects 01, 22 and 35
1. What is on the slide
Element	Content
Image 1 (3-panel strip)	"Hold-out accuracy (red = chosen by grouped CV)" bar chart; "ROC per hold-out subject, SVM-RBF (all)"; "Confusion matrix (hold-out subjects)"
Image 2 (3-panel strip)	Epoch-by-epoch P(task) for Subject01, Subject22, Subject35 (blue = true Rest, red = true Task, black = smoothed w=3, green dotted = Rest/Task switch)
Image 3 (3-panel strip)	"Top 15 features (mean
Callout 1	92.0% raw balanced accuracy, AUC 0.974
Callout 2	97.7% smoothed over 3 epochs (not strict)
Bullet 1	SVM-RBF chosen by grouped CV on training subjects only.
Bullet 2	Subject01 82.1%, Subject22 96.7%, Subject35 96.7%.
Bullet 3	Confusion: 40 and 41 correct, 4 and 3 wrong.
Bullet 4	All 6 recordings classified correctly.
2. Script (about 4 minutes)
"So far every number came from leave-one-subject-out. Now the cleanest possible test. Three subjects, 01, 22 and 35, were fixed in advance. Nothing about them touched imputation, scaling, feature selection, hyperparameters, or model choice. The model trained on the other 33 subjects, 988 epochs, and was scored once on the 88 epochs from the three held-out people.

How was the model chosen? By grouped cross-validation on the training subjects only. The SVM with RBF kernel using all features won, with a cross-validated balanced accuracy of 0.746 and AUC 0.837. We did not choose it by looking at the hold-out score.

The first strip, left panel: hold-out balanced accuracy for ten models. The red bar is the model chosen by cross-validation. Using all 1,076 features, the SVM scores 0.920, logistic regression, XGBoost and the soft-vote ensemble 0.886, ExtraTrees 0.818, Random Forest 0.807. Models restricted to the top-k selected features score only 0.73 to 0.74.

Middle panel: ROC per hold-out subject. Subject 22 is a perfect 1.00, Subject 35 is 0.996, and Subject 01 is the weakest at 0.87.

Right panel: the confusion matrix: of 44 Rest epochs, 40 correct and 4 wrong; of 44 Task epochs, 41 correct and 3 wrong. That is 81 out of 88, or 92.0%.

The second strip is my favourite. Each dot is one 4-second epoch; blue dots are truly Rest, red dots are truly Task; the vertical green dotted line is where the person switched from Rest to Task. The vertical axis is the model's probability of Task. You can see the blue dots sit low and red dots sit high, with a few misses, mostly for Subject 01. The black line is a three-epoch rolling average, which smooths out the occasional outlier and lifts accuracy to 97.7%. But I must be clear: smoothing assumes you have the neighbouring epochs of the same recording in time order, so it is not a strict per-epoch result.

The third strip shows what the model leaned on in the hold-out. By feature family, frequency features carry about 40% of the importance, non-linear and wavelet about 20% each, time-domain about 18%, and connectivity almost nothing. By scalp region, frontal is largest at about 30%, then temporal at about 23%.

Finally, at the recording level, taking the mean Task probability across a whole recording: every Rest recording averages below 0.27 and every Task recording above 0.71. All six recordings are classified correctly.

Please do not over-claim. Three people and 88 epochs is a tiny sample. The notebook itself says LOSO, 73.7%, is the headline."

3. Picture-by-picture explanation
Strip 1, panel A: Hold-out accuracy by model (red = chosen by grouped CV)

Horizontal bars, x = balanced accuracy from 0.3 to 1.0, dashed line at 0.5.
Values printed in the notebook (raw epoch predictions):
Model	Bal. acc	AUC	F1 (task)	MCC
SVM-RBF (all)	0.920	0.974	0.921	0.841
LogReg (all)	0.886	0.934	0.886	0.773
XGBoost (all)	0.886	0.963	0.889	0.774
Ensemble: soft vote (all)	0.886	0.958	0.889	0.774
ExtraTrees (all)	0.818	0.909	0.830	0.642
RandomForest (all)	0.807	0.903	0.817	0.618
SVM-RBF (top-k)	0.739	0.811	0.753	0.480
RandomForest (top-k)	0.739	0.821	0.753	0.480
LogReg (top-k)	0.727	0.809	0.733	0.455
XGBoost (top-k)	0.727	0.819	0.733	0.455
Smoothed (window = 3) results (printed): SVM-RBF (all) 0.977 / AUC 0.998; LogReg (all) 0.955 / 0.996; soft-vote ensemble 0.955 / 0.993; XGBoost (all) 0.943 / 0.993; ExtraTrees (all) 0.864 / 0.955; RandomForest (all) 0.864 / 0.952; SVM-RBF (top-k) 0.807 / 0.863; LogReg (top-k) 0.784 / 0.857; XGBoost (top-k) 0.761 / 0.849; RandomForest (top-k) 0.750 / 0.864.
Cross-validated selection scores (training subjects only, printed): SVM-RBF (all) 0.746 / 0.837; soft vote 0.744 / 0.829; XGBoost (all) 0.739 / 0.804; LogReg (all) 0.734 / 0.806; LogReg (top-k) 0.729 / 0.808; RandomForest (all) 0.723 / 0.804; ExtraTrees (all) 0.721 / 0.804; XGBoost (top-k) 0.716 / 0.787; RandomForest (top-k) 0.712 / 0.780; SVM-RBF (top-k) 0.699 / 0.787.
Strip 1, panel B: ROC per hold-out subject (SVM-RBF all)

Blue = Subject01 (AUC 0.87), orange = Subject22 (AUC 1.00), green = Subject35 (AUC 0.996), dashed = chance.
Subject22's curve goes straight up the left edge: perfect ranking. Subject01's staircase shows a few Task epochs scored lower than some Rest epochs.
Strip 1, panel C: Confusion matrix

Predicted Rest	Predicted Task
True Rest (44)	40	4
True Task (44)	3	41
Precision/recall printed: Rest 0.93 / 0.91, Task 0.91 / 0.93, accuracy 0.92.
Strip 2: Epoch-by-epoch probability of Task

x-axis: epoch index (each epoch = 4 s). y-axis: model's predicted probability of Task. Blue dots true Rest; red dots true Task; green dotted line boundary between the two recordings; black line smoothed (window 3); dashed line at 0.5.
Subject01 (28 epochs, 82.1%): noticeably noisy: a few blue (Rest) dots above 0.5 early in the recording (some near 0.75 to 0.9) and red (Task) dots below 0.5 near the end (e.g. around 0.25 to 0.35). The smoothed line drifts down at the end of the task recording (approx., read from the plot).
Subject22 (30 epochs, 96.7%): almost perfectly separated; one blue dot at about 0.72 and one red dot around 0.86 near the boundary.
Subject35 (30 epochs, 96.7%): clean separation; one blue dot at about 0.7 and one red around 0.6 (approx., read from the plot).
Why the smoothed curve rises sharply at the boundary: a rolling mean of three neighbouring epochs straddling the Rest/Task switch blends both sides, so a short ramp appears at the switch.
Per-subject numbers printed

Subject	Epochs	Accuracy	Bal. acc	AUC
Subject01	28	0.821	0.821	0.872
Subject22	30	0.967	0.967	1.000
Subject35	30	0.967	0.967	0.996
Recording-level decision (mean P(task) per recording, printed)

Subject	Rest recording	Task recording
Subject01	0.264 → Rest ✔	0.711 → Task ✔
Subject22	0.103 → Rest ✔	0.943 → Task ✔
Subject35	0.146 → Rest ✔	0.926 → Task ✔
Counting quality of the hold-out subjects: Subject01 (age 18, F, good counter = 1), Subject22 (age 19, F, bad counter = 0), Subject35 (age 17, F, good counter = 1). All three are female and aged 17 to 19, a narrow slice of the population.

Strip 3: Hold-out SHAP

Top 15 features (approx., read from the plot): C3_wavelet_energy_L4 (about 0.25), P3_line_length (about 0.25), T6_higuchi_fd (about 0.24), Pz_wavelet_energy_L3 (about 0.24), T5_approximate_entropy (about 0.19), T5_gamma_relative_power, Fp1_gamma_power, P4_katz_fd, P4_higuchi_fd, Cz_theta_beta_ratio, Fp1_hjorth_complexity, O2_higuchi_fd, O1_peak_to_peak, P3_wavelet_energy_L3, O1_theta_power_log.
Share by feature family (approx.): Frequency about 0.40, Nonlinear about 0.20, Wavelet about 0.20, Time-domain about 0.18, Connectivity about 0.01.
Share by scalp region (approx.): Frontal about 0.30, Temporal about 0.23, Parietal about 0.185, Central about 0.16, Occipital about 0.12.
Note: T6_higuchi_fd and T5_gamma_relative_power appear in both the LOSO analysis (Slide 8) and the hold-out analysis, which is a mild sign of consistency.
4. How, why, why not
How the hold-out is protected: extended_holdout_prediction splits by subject, trains on the 33 training subjects, selects among ten candidate models by grouped CV on those 33 only, then fits and predicts the hold-out once.
Why fixed subjects (1, 22, 35)? Constant in HOLDOUT_SUBJECT_IDS, set before running, so nobody can choose "lucky" subjects.
Why "all features" beat "top-k" here? In the hold-out run, models with all 1,076 features scored 0.81 to 0.92, while top-150 versions scored 0.73 to 0.74. With strong regularisation (SVM, logistic regression), extra weak features add information. The top-k selection cuts some of it away. This also differs from the LOSO zoo (which used top-150 for every model).
Why smoothed results are reported but flagged? Real recordings are continuous, so neighbouring epochs do carry information; but a strict test assumes each epoch is classified alone. The notebook says: "report the raw numbers as the strict result".
Why a recording-level decision? If the whole recording is known to be Rest or Task, averaging all epoch probabilities is a powerful vote. It is shown as an extra, not as the headline.
Why not select by hold-out score? That would make the hold-out no longer a test; the notebook explicitly says the model is "never" chosen by hold-out score.
5. Deep analysis: is 92% believable?
Statistical uncertainty. 81 correct out of 88 gives 92.0%. A Wilson 95% confidence interval for a proportion this size is roughly 84% to 96% if epochs were independent; they are not (epochs from one person are correlated and there are only 3 people), so the real interval is wider. Subject01 alone is 23/28 = 82.1%, which means one person's noise can move the pooled score by several points.

Why the hold-out (92%) beats LOSO (73.7%)? Five reasons:

Tiny sample. Three subjects vs 36. Subjects 22 and 35 happen to be highly separable (96.7%). If the other 33 subjects looked like them, LOSO would be above 90%. The LOSO table shows per-subject SD of 0.17, so some subjects are very easy, some hard.
"All features" vs "top-150". The LOSO zoo used in-fold top-150 selection; the hold-out winner used all 1,076 features. Within the hold-out table itself, top-k models score about 0.73 to 0.74, close to LOSO levels.
Training set size. Hold-out trains on 33 subjects; LOSO on 35. Practically identical.
Smoothing. The 97.7% figure borrows information from neighbouring epochs; the strict number is 92.0%.
Selection effects are small but real. The hold-out model was chosen by grouped CV (0.746), consistent with LOSO (0.737), so there is no sign of overfitting in the choice; the gap is mostly the three subjects being easier.
What the Subject01 result teaches. Even among three subjects with good counting quality, accuracy ranges from 82% to 97%. A deployed system would deliver very different experiences to different users.

Demographic narrowness. All three hold-out subjects are female, 17 to 19 years old. Generalising to other ages or sexes is untested here.

6. Likely questions
"Why is the hold-out so much higher than LOSO? Is something wrong?" Nothing is leaking: the hold-out is trained on other subjects only and the model was picked on training subjects. The difference is mostly sample noise (3 people) plus the "all features" model. The honest summary figure is LOSO 73.7%.
"Why did Subject22, a 'bad counter', score 96.7%?" Counting quality did not predict the Task-Rest change in the group analysis (p = 0.54) and the model likely detects the state of effort or arousal, not accuracy of counting. With n = 1 per category in the hold-out, you cannot conclude anything.
"Why do the top-k hold-out models perform so much worse?" Selecting 150 of 1,076 features by ANOVA on the training subjects may drop features that matter for these particular people; strongly regularised models benefit from the full set.
SLIDE 10: Why this accuracy, and what to trust
1. What is on the slide
Element	Content
Card 1	Why SVM and ensembles lead (about 74%): 1,076 features for about 1,000 epochs: a regularised margin model plus per-subject z-scoring copes best. The top ensemble (0.743) was picked after seeing LOSO, so it is optimistic.
Card 2	Why trees score lower (64-72%): A single tree memorises noise. Bagging and boosting average that away (+7-9 points) but still trail the SVM.
Card 3	Why hold-out hits 92%, not 74%: Only 3 subjects, 88 epochs, so very noisy. The SVM also used all features. Smoothing (97.7%) borrows neighbouring epochs. Trust LOSO 73.7%.
Verdict bar	Verdict: SVM-RBF is the best and most stable model; tree ensembles are close but not better.
Image	Scalp map "Channel importance (XGBoost, all features)", colour bar "summed mean
Caption	Channel importance: spread over the whole scalp
Warning text	Not proven as pure brain signal: Rest and Task are separate recordings (fatigue is confounded) and 45% of top features come from eye/muscle-prone channels.
2. Script (about 4 minutes, the closing argument)
"Let me answer the three questions every audience asks: why is this model the best, why aren't the trees better, and why does the hold-out number look so much better than the main result.

Why do the SVM and the ensembles lead? We have about a thousand epochs but over a thousand features, many of them redundant, as the heatmap showed. A regularised margin-based model, like the SVM, uses all of those weak cues smoothly and is hard to overfit. Per-subject z-scoring helps it further by removing baseline differences. The top ensemble, at 0.743, looks best, but we picked it after looking at the LOSO results, so it is optimistic. The SVM at 0.737 is the honest leader, and it was also what grouped cross-validation chose for the hold-out.

Why do trees score lower? A single tree asks one question at a time and memorises the quirks of the training subjects. Averaging many trees, bagging, or correcting errors step by step, boosting, removes most of that noise and gains seven to nine points, but they still trail the SVM by about one to three points, and that gap is smaller than the uncertainty.

Why 92% in the hold-out but 74% in LOSO? Because three people is a very small sample, two of whom happen to be very easy to decode, because the hold-out model used all features, and because smoothing borrows from neighbouring epochs. The trustworthy number is the 36-subject LOSO result.

Verdict: SVM-RBF is the best and most stable choice, and tree ensembles are close but not better.

Now the most important slide message. On the right is the channel-importance map: SHAP importance summed per electrode and drawn on a head. It is spread across the whole scalp, with no single hot spot. That is good: the model is not relying on one electrode. But the warning below it matters. We cannot call this proof of a pure brain signal. First, Rest and Task are separate recordings, so fatigue, drowsiness or time-in-session are mixed up with the label. Second, 45% of the top features come from channels where eye and muscle artifacts are strongest. The notebook includes a robustness test, Section 16, that re-runs the model without those channels, without gamma and fast wavelets, and without the per-subject z-scoring. Its result was not saved in the version of the notebook I received, so I am not reporting it. Running that cell is the single most valuable next step.

To summarise: with a careful, leak-free pipeline, a computer can tell Rest from mental arithmetic at about 74% accuracy on completely new people, 92% on three held-out people, with clear caveats about artifacts and confounds. Thank you. I'm happy to take questions."

3. Picture explanation: channel-importance head map
What it is: for each of the 19 electrodes, the sum of the mean |SHAP| values of all features belonging to that channel, interpolated over a head outline. Colour bar from 0 (white) to about 1.2 (dark red).
Reading (approx., read from the plot): every region is red to some degree; the darkest areas lie toward the lateral/temporal and posterior-lower regions of the head, while central and frontal areas are medium red. There is no blank (white) area.
Meaning: importance is distributed, not concentrated at one electrode.
Limit: a summed-importance map cannot say whether a channel contributes brain signal or artifact; it measures use by the model.
4. Master comparison tables (use as backup slides)
4a. Everything in one table: LOSO vs hold-out

Aspect	LOSO (36 subjects)	Hold-out (3 subjects)
Training subjects	35 per fold	33
Test subjects	1 per fold, 36 folds	3 fixed (01, 22, 35)
Test epochs	1,076	88
Best single model	SVM-RBF 0.737	SVM-RBF (all) 0.920
Features used	top-150 in-fold	all 1,076 (winner)
AUC	0.820	0.974
Uncertainty	Moderate (n = 36, SD 0.17)	Large (n = 3)
Tuning optimism	Slight (tuned on same subjects)	None (tuned on training subjects only)
Role	Headline estimate	Sanity check
4b. Model families

Family	Models	LOSO bal. acc range	Strength	Weakness
Linear / margin	LogReg-L2, SVM-Linear, SVM-RBF	0.732 to 0.737	Smooth use of many weak features, robust	Less interpretable (RBF)
Single trees	DecisionTree, ExtraTree	0.641 to 0.654	Easy to read	High variance, worst accuracy
Bagged / random forests	Bagging, RandomForest, ExtraTrees	0.701 to 0.724	Reduces variance	Deep trees still fit subject noise
Boosting	AdaBoost, GB, HistGB, XGBoost	0.727 to 0.736	Strong, SHAP-friendly	Needs tuning
Ensembles	SVMs, SVM+boosting, all, trees, top-3	0.726 to 0.743	Small gains	Top-3 was chosen post-hoc
4c. Why / why not for the top design choices

Choice	Why it was made	Why not the alternative	Cost / risk
LOSO validation	Realistic for new users	Random split leaks identity	Slower (36 fits per model)
Per-subject z-scoring	Removes baseline differences	Raw features: between-person variation dominates	Needs recordings containing both conditions
In-fold feature selection	No leakage	Global selection inflates results	Slightly lower accuracy
ICA + AutoReject	Remove blink/muscle	Raw data full of artifacts	Imperfect with 19 channels
SHAP on unseen subjects	What generalises	In-sample SHAP shows memorisation	More compute
Fixed hold-out	Honest test	Post-hoc choice is cherry-picking	Only 3 subjects
Report per-subject SD	Shows variability	A mean alone hides it	Reveals sobering spread
5. Deep analysis of limitations (from Section 18 of the notebook, explained)
Rest and Task are separate recordings. Rest is truncated to the first epochs to balance classes. Anything that changes over time (fatigue, boredom, drowsiness, electrode gel drying, posture) is perfectly confounded with the label. No analysis here can separate "doing arithmetic" from "being later in the session". Fix: a design where Rest and Task blocks alternate within one session, and counterbalanced order.
Residual artifacts. ICA with 19 electrodes and no EOG/EMG channels is imperfect. Top SHAP features include forehead and temporal gamma and complexity measures. Fix: run Section 16; collect dedicated EOG/EMG; use more electrodes.
Per-subject z-scoring needs a recording that mixes both conditions. In a real product, you would not have that for a new user without a calibration step. Fix: use a short resting baseline for normalisation, or an adaptive scheme.
Small sample. 36 subjects. Per-subject SD of 0.17 to 0.2. Fix: more subjects.
Light optimism in LOSO. Hyperparameters were tuned on the same subjects; the top-3 ensemble was picked using LOSO AUC. The hold-out avoids both.
Group effects are inconsistent. Topomaps differ between people; do not claim a universal spatial pattern.
Narrow hold-out demographics. Three females aged 17 to 19.
6. Next steps (what a good follow-up looks like)
Run Section 16 and report the five-variant table: all features, without frontal-pole/temporal channels, without gamma and 16-64 Hz wavelet, without both, and without z-scoring. If accuracy stays near 0.7 after removing artifact-prone information, that is strong evidence for a neural signal. If it collapses toward 0.5, the headline was mostly artifact.
Add a nested cross-validation so tuning and evaluation use different subjects.
Draw an actual tree (plot_tree) for the DecisionTree if interpretability is needed.
Test with new data, different ages and sexes, and a within-session alternating design.
Trim features for real-time use (the entropy and fractal measures are the slowest).
7. Likely questions
"So can your model read minds?" No. It distinguishes two coarse states (resting vs arithmetic) at about 74% on new people; it cannot decode what someone is thinking.
"Is the result publishable?" The pipeline is methodologically careful (leak-free, subject-wise validation, hold-out, explainability), but the confounds in the recording design and the unreported robustness check should be resolved first.
"What is the single biggest weakness?" The separate-recordings design, because it confounds time with task.
"What would convince you the signal is neural?" Accuracy staying well above chance when frontal-pole/temporal channels and gamma/fast-wavelet features are removed, ideally replicated on data with alternating Rest and Task blocks.
PART C. ONE-PAGE CHEAT SHEET FOR Q&A
Question	Short answer
What is the task?	Classify 4-second EEG windows as Rest or Mental-arithmetic Task.
How much data?	36 subjects, 19 channels, 1,076 balanced epochs, 1,076 features.
Best honest accuracy?	73.7% balanced accuracy (SVM-RBF, LOSO, AUC 0.820).
Best hold-out?	92.0% raw (97.7% smoothed), SVM-RBF on 3 subjects.
Best tree model?	AdaBoost 0.736, XGBoost 0.735 (boosting); worst: DecisionTree 0.641.
Why are trees lower?	Single trees overfit; forests still fit subject noise; boosting catches up but not beyond the SVM.
Are the differences meaningful?	Top models are within about 1 point; the standard error is about 0.03, so no.
Is it reading the brain?	Unproven: 45% of top-20 features are from artifact-prone channels; Section 16 must be run.
Biggest design flaw?	Rest and Task are separate recordings (time/fatigue confounded).
Why is hold-out higher than LOSO?	Three easy subjects, all-features model, smoothing; sample is tiny.
What is SHAP?	A method that splits each prediction into per-feature contributions (Shapley values).
What is LOSO?	Train on 35 subjects, test on the 36th, repeat for all 36.
What does z-scoring per subject do?	Removes person-specific baseline from every feature.
Top feature?	T6_higuchi_fd (fractal dimension at right posterior-temporal electrode).
Connectivity features useful?	Almost not (about 1% of importance); only 12 of 1,076 features.
PART D. TIMING PLAN
Slide	Topic	Minutes
1	Title and key numbers	1.5
2	Project map	2
3	Preprocessing and features	3
4	Topomaps	2.5
5	Redundancy and theta effect	2.5
6	Model comparison and trees	3.5
7	ROC and confusion matrices	3
8	SHAP	3.5
9	Hold-out	4
10	Why / why not and caveats	4
Total	about 29.5
For a 15-minute version: combine slides 4 and 5 into one sentence each, skip the per-model details on Slide 6 except single tree vs ensemble vs SVM, and shorten Slide 9 to the three callouts and the timeline picture.

End of script.

---

# EEG Mental Arithmetic: Rest vs Task. Speaking Script

## Slide 1. Title, "EEG Mental Arithmetic: Rest vs Task"

Good morning everyone. Today I'll show you a complete machine-learning project that answers one question: can a computer look at 4 seconds of brain-wave recording and tell whether the person was resting or doing mental arithmetic?

The title says Rest versus Task. 'Rest' means the person sits quietly. 'Task' means the person is mentally subtracting numbers. EEG, which stands for electroencephalography, records tiny electrical signals from the scalp using 19 electrodes.

Let me walk through the four numbers on the slide because they frame the whole talk. We have **36 subjects**, each recorded twice, once at rest and once during the task. After cleaning, we have **1,076 epochs**: an epoch is a 4-second slice of signal, and we keep exactly the same number of Rest and Task slices, 538 of each. From every slice we compute **1,076 features**, which are numbers that describe the signal: its power in different rhythms, its complexity, how connected different brain areas are, and so on. It is a coincidence that the number of features equals the number of epochs.

The last card is the most important: **73.7%**. That is the balanced accuracy when the model is tested on a person it has never seen before. That is our honest headline number. Chance is 50%, so the model is clearly learning something real, but it is far from perfect, and I'll explain why in this talk.

## Slide 2. The project, what, why and how

This slide is the map of the whole project. On the left: what, why, and the data. On the right: the six steps.

**What.** The task is binary classification. Each 4-second window gets a label, Rest or Task.

**Why.** Mental workload leaves a signature in the brain's electrical rhythms. If we can detect it automatically, we could build systems that monitor how hard someone's brain is working, for example for safety-critical jobs, or use it as a control signal in a brain-computer interface, which is a system where brain activity controls a computer.

**Data.** 36 volunteers, 19 electrodes placed using the international 10-20 system, one recording at rest, one during mental arithmetic.

Now the six steps. **One**, preprocessing: raw EEG is full of non-brain noise, so we filter it, remove blink and muscle components with ICA, and drop bad windows with AutoReject. **Two**, feature extraction: we turn each window into 1,076 numbers. **Three**, we z-score each subject separately, which removes the fact that some people just have bigger signals. **Four**, we test 17 models and ensembles using leave-one-subject-out validation, meaning we train on 35 people and test on the 36th, and repeat for everyone. **Five**, we open the black box with SHAP to see which features matter. **Six**, we check on three completely untouched subjects, and we ask the hardest question: is the model reading the brain, or reading eye and muscle artifacts?

## Slide 3. Cleaning the signal and building features

Raw EEG is like a recording of a conversation in a noisy cafe. Before we can analyse the speech, we have to remove the noise. The left card shows six cleaning steps.

**Step 1.** The dataset's file headers label two non-brain channels as EEG: the ear reference A2-A1 and a heart channel. We match every channel name against the standard 10-20 montage and drop anything that does not match. That leaves 19 true scalp electrodes.

**Step 2.** Average reference. EEG voltages are always measured relative to something. We subtract the average of all channels from each channel, so no single electrode acts as the reference and noise common to all electrodes is cancelled.

**Step 3.** A band-pass filter from 1 to 45 Hz. Below 1 Hz is slow drift from sweat and movement. Above 45 Hz is mostly muscle noise. We use a zero-phase FIR filter so the timing of events is not shifted. We also add a notch at 50 Hz because the electrical mains hum at 50 Hz is not fully removed by the 45 Hz low-pass edge.

**Step 4.** Resample to 128 Hz. Since we only keep up to 45 Hz, we need only above 90 samples per second; 128 is a safe, efficient choice that still represents up to 64 Hz.

**Step 5.** ICA. Independent Component Analysis unmixes the 19 channels into 15 independent sources. Some sources are clearly eye blinks, because they dominate Fp1 and Fp2, the forehead electrodes. Others look like muscle. We remove the blink-like ones first, then the most muscle-like ones, but never more than four. We fit one ICA on Rest and Task together so both recordings are cleaned in exactly the same way.

**Step 6.** We cut the signal into 4-second epochs, then AutoReject throws out or repairs windows with excessive noise. Finally we truncate so each subject has the same number of Rest and Task epochs.

Now the chart. From each channel in each epoch we compute **56 features** in four families: time-domain, frequency-domain, wavelet and non-linear. Times 19 channels gives 1,064. We add 12 connectivity features between four electrode pairs. Total, 1,076. Finally, each feature is z-scored within each subject. That removes person-to-person differences like skull thickness and electrode contact that have nothing to do with the task.

## Slide 4. EDA, where does theta power change?

Before modelling, always look at the data. Here we ask a simple question: which parts of the head show more or less theta activity during the arithmetic task compared with rest? Theta is the 4-8 Hz rhythm often linked to mental effort.

A topomap is a top-down view of a head, nose at the top. Colour is interpolated between the 19 electrodes. In the single-subject picture on the right, the first two heads show Rest and Task on the same colour scale, so you can compare them directly. The third head shows the difference: Task minus Rest, on a blue-to-red scale centred on zero. Red means more theta during the task, blue means less.

This is Subject 0. You can see some red and blue patches, but when we test each brain region statistically, with a Bonferroni correction because we test five regions, nothing is significant. The smallest corrected p-value is 0.78. So for this one person, the visible pattern could easily be random.

Now the big grid on the left: all 36 subjects. If everybody's brain responded the same way, all 36 maps would look alike, say all red at the front. They do not. Some subjects show red at the front, some at the back, some mostly blue. That tells us something crucial for the modelling: there is **no single universal spatial pattern**, so the classifier cannot rely on one 'theta hot spot'. It has to combine many weak, person-varying cues.

## Slide 5. EDA, feature redundancy and a small theta effect

Two more exploratory pictures, answering two questions. First: how much do our features overlap? Second: is there any simple, group-level difference between Rest and Task?

**The heatmap.** Take one electrode, Fz, on the midline of the forehead. We have many features for it: power in each band, the log of that power, relative power, and band ratios. The heatmap shows how strongly each pair is correlated. Dark red means they move together, dark blue means they move in opposite directions. The strong red squares show, for instance, that delta power and log delta power are almost the same information. The blue patches show, for example, that theta/alpha and alpha/theta are mirror images. This tells us the feature set is highly redundant. That is not a mistake; it is a reason the models need regularisation or feature selection, so they do not chase duplicated noise.

**The box plot.** Now the group-level question. For each subject we averaged frontal theta relative power separately for Rest and Task, so each dot is one person. There are 36 dots per condition, one per subject, because epochs from the same person are not independent. The boxes show the spread. Task is slightly higher than Rest. The paired Wilcoxon test, which compares each person with themselves, gives p equals 0.0105, so the increase is statistically significant. But look at the size: the median change is only about 0.01 in relative power, tiny compared with the person-to-person spread. This is a small effect.

Finally, we split subjects into good and bad counters, based on whether they actually counted well. If the arithmetic effort were driving the effect, you might expect good counters to show a larger change. The Mann-Whitney test gives p equals 0.54: no evidence of a difference. So we cannot say the effect depends on counting quality.

## Slide 6. Model comparison, all trees vs the rest

Now the main experiment. We compared 12 individual models and 5 ensembles, 17 in total, using leave-one-subject-out testing. Every model sits inside exactly the same pipeline: fill missing values with the median, remove constant features, standardise, select the 150 features that best separate the classes, then classify. All of those steps are fitted on the training subjects only, inside each fold, so the test subject never influences anything.

The bar chart on the left sorts models from best at the top to worst at the bottom. Notice two things. First, the top bars are almost equal, around 0.73 to 0.74. Second, the black error bars, which are the standard deviation across subjects, are very long, about 0.17 to 0.2. People differ enormously.

You asked about trees, so here is the full tree family on the right. I'll explain each in plain words.

A **Decision Tree** is a single flowchart of yes/no questions on features, for example 'is T6 fractal dimension above this value?' It is easy to read but unstable; change the training data slightly and you get a different tree. It scores 0.641. An **Extra Tree** is the same, but split points are chosen randomly. 0.654.

**Bagging** trains 300 trees, each on a random 80% of the epochs and half the features, and averages their votes. Averaging cancels each tree's quirks. 0.708. A **Random Forest** adds extra randomness: at each split only a random subset of features is considered. 0.702. **Extra Trees**, the forest of randomised-threshold trees, reaches 0.724.

Then the **boosting** family. Instead of independent trees, each new tree concentrates on the mistakes of the previous ones. **AdaBoost** reweights misclassified epochs: 0.736. **Gradient Boosting** fits each new tree to the remaining error: 0.728. **Histogram Gradient Boosting** is a faster binned version: 0.727. **XGBoost** is a regularised gradient booster with shallow trees: 0.735.

The lesson: single trees are clearly worst at 64 to 65%. Averaging or boosting trees adds about 7 to 9 points. But the best non-tree model, the **SVM with an RBF kernel**, still ties or slightly beats them at 0.737. And look at the spread: the top models are all within about a point of each other, which is much smaller than the person-to-person spread. So we should not claim one clearly beats another.

## Slide 7. LOSO results, ROC curves and confusion matrices

The previous slide gave one number per model. This slide opens it up for two models, Random Forest and XGBoost, so you can see how they are right and wrong.

**ROC curves, top row.** An ROC curve answers: if I move the decision threshold from strict to lenient, how does the catch rate trade off against false alarms? The vertical axis is the true-positive rate: the share of real Task epochs correctly flagged. The horizontal axis is the false-positive rate: the share of Rest epochs wrongly flagged as Task. A perfect model hugs the top-left corner. A useless model follows the diagonal. Each pale, stepped line is one held-out subject, the dark blue line is the mean, and the shaded band is plus or minus one standard deviation. Notice how widely the pale lines fan out: some subjects are decoded almost perfectly, a few are barely better than chance. That fan is the person-to-person variation, with a standard deviation of about 0.2.

**Confusion matrices, bottom row.** These pool all 1,076 epochs. Rows are the truth, columns are the prediction. The dark diagonal cells are the correct calls. Both models have errors in both off-diagonal cells in similar numbers, so there is no systematic bias toward calling things Rest or Task.

**The table** confirms it in numbers. Random Forest: precision and recall are 0.70 for both classes. XGBoost: Rest precision 0.74, recall 0.72, Task precision 0.73, recall 0.75, overall accuracy 0.74. XGBoost is about four points better. Precision is: of those I called Task, how many truly were. Recall is: of all truly Task, how many did I find.

## Slide 8. SHAP, which features drive the predictions?

A model that is right 74% of the time is useful, but a scientist wants to know why. SHAP is a method from cooperative game theory. It splits each prediction among the features, like dividing a prize among team members according to what each contributed. A positive SHAP value pushes the prediction toward Task, a negative one toward Rest. The size of the average absolute SHAP value tells you how important a feature is.

**The left chart** is the most honest one. For each fold, the model is trained on some subjects, and SHAP is computed on subjects the model has not seen. We then average. So these are the features that matter for new people. The bars are coloured: red means the feature comes from a frontal-pole or temporal electrode. Why highlight those? Because those are the places where eye movements and jaw or scalp muscle tension leave their strongest marks. The notebook counts it: **45% of the top 20 features come from those eight artifact-prone channels, while those channels make up 42% of the scalp**. So they are only slightly over-represented. That is mildly reassuring, but it is not proof of clean brain signal.

**The middle chart** fits XGBoost on all subjects, purely for explanation. The top feature is **T6 Higuchi fractal dimension**, by a wide margin, followed by **T5 gamma relative power**, **Fp1 Hjorth complexity**, and alpha-power features at Fp2. In plain language: how rough and complex the waveform is at the right-back temporal site, and the share of fast gamma activity, are the strongest cues.

**The right chart, the beeswarm,** shows direction. Each dot is one epoch. Colour is the feature value: blue low, red high. Position left or right is its effect on the prediction. For the top feature, high values, the red dots, sit on the right, meaning a rougher waveform at T6 pushes the model toward Task.

And now the caution, which is the message of the bottom bar. Gamma power, signal roughness and Hjorth complexity measured at temporal and frontal sites can also rise with jaw clenching and scalp muscle tension. A person concentrating on arithmetic may frown or clench slightly. So we cannot say from SHAP alone that the model reads the brain. That is why the notebook has Section 16, which re-runs the model without those channels and without gamma and the fast wavelet levels.

## Slide 9. Clean hold-out test, Subjects 01, 22 and 35

So far every number came from leave-one-subject-out. Now the cleanest possible test. Three subjects, 01, 22 and 35, were fixed in advance. Nothing about them touched imputation, scaling, feature selection, hyperparameters, or model choice. The model trained on the other 33 subjects, 988 epochs, and was scored once on the 88 epochs from the three held-out people.

**How was the model chosen?** By grouped cross-validation on the training subjects only. The SVM with RBF kernel using all features won, with a cross-validated balanced accuracy of 0.746 and AUC 0.837. We did not choose it by looking at the hold-out score.

**The first strip, left panel:** hold-out balanced accuracy for ten models. The red bar is the model chosen by cross-validation. Using all 1,076 features, the SVM scores 0.920, logistic regression, XGBoost and the soft-vote ensemble 0.886, ExtraTrees 0.818, Random Forest 0.807. Models restricted to the top-k selected features score only 0.73 to 0.74.

**Middle panel:** ROC per hold-out subject. Subject 22 is a perfect 1.00, Subject 35 is 0.996, and Subject 01 is the weakest at 0.87.

**Right panel:** the confusion matrix: of 44 Rest epochs, 40 correct and 4 wrong; of 44 Task epochs, 41 correct and 3 wrong. That is 81 out of 88, or 92.0%.

**The second strip** is my favourite. Each dot is one 4-second epoch; blue dots are truly Rest, red dots are truly Task; the vertical green dotted line is where the person switched from Rest to Task. The vertical axis is the model's probability of Task. You can see the blue dots sit low and red dots sit high, with a few misses, mostly for Subject 01. The black line is a three-epoch rolling average, which smooths out the occasional outlier and lifts accuracy to 97.7%. But I must be clear: smoothing assumes you have the neighbouring epochs of the same recording in time order, so it is **not** a strict per-epoch result.

**The third strip** shows what the model leaned on in the hold-out. By feature family, frequency features carry about 40% of the importance, non-linear and wavelet about 20% each, time-domain about 18%, and connectivity almost nothing. By scalp region, frontal is largest at about 30%, then temporal at about 23%.

Finally, at the recording level, taking the mean Task probability across a whole recording: every Rest recording averages below 0.27 and every Task recording above 0.71. All six recordings are classified correctly.

**Please do not over-claim.** Three people and 88 epochs is a tiny sample. The notebook itself says LOSO, 73.7%, is the headline.

## Slide 10. Why this accuracy, and what to trust

Let me answer the three questions every audience asks: why is this model the best, why aren't the trees better, and why does the hold-out number look so much better than the main result.

**Why do the SVM and the ensembles lead?** We have about a thousand epochs but over a thousand features, many of them redundant, as the heatmap showed. A regularised margin-based model, like the SVM, uses all of those weak cues smoothly and is hard to overfit. Per-subject z-scoring helps it further by removing baseline differences. The top ensemble, at 0.743, looks best, but we picked it after looking at the LOSO results, so it is optimistic. The SVM at 0.737 is the honest leader, and it was also what grouped cross-validation chose for the hold-out.

**Why do trees score lower?** A single tree asks one question at a time and memorises the quirks of the training subjects. Averaging many trees, bagging, or correcting errors step by step, boosting, removes most of that noise and gains seven to nine points, but they still trail the SVM by about one to three points, and that gap is smaller than the uncertainty.

**Why 92% in the hold-out but 74% in LOSO?** Because three people is a very small sample, two of whom happen to be very easy to decode, because the hold-out model used all features, and because smoothing borrows from neighbouring epochs. The trustworthy number is the 36-subject LOSO result.

**Verdict:** SVM-RBF is the best and most stable choice, and tree ensembles are close but not better.

Now the most important slide message. On the right is the channel-importance map: SHAP importance summed per electrode and drawn on a head. It is spread across the whole scalp, with no single hot spot. That is good: the model is not relying on one electrode. But the warning below it matters. **We cannot call this proof of a pure brain signal.** First, Rest and Task are separate recordings, so fatigue, drowsiness or time-in-session are mixed up with the label. Second, 45% of the top features come from channels where eye and muscle artifacts are strongest. The notebook includes a robustness test, Section 16, that re-runs the model without those channels, without gamma and fast wavelets, and without the per-subject z-scoring. Its result was not saved in the version of the notebook I received, so I am not reporting it. Running that cell is the single most valuable next step.

**To summarise:** with a careful, leak-free pipeline, a computer can tell Rest from mental arithmetic at about 74% accuracy on completely new people, 92% on three held-out people, with clear caveats about artifacts and confounds. Thank you. I'm happy to take questions.