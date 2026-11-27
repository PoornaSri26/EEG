# Understanding Your EEG Feature Extraction (Plain-English Guide)

This explains what the feature extraction block does, why each feature exists, and
what it actually tells you about a person's brain — no jargon assumed.

---

## 1. The big picture

Your EEG cap records tiny electrical voltages from the scalp, 500 times per
second, from 23 spots on the head. Raw, that's just a wiggly line — useless
to a computer. **Feature extraction** turns each short chunk of that wiggly
line (an "epoch," here 4 seconds) into a set of numbers that summarize *how*
it wiggles: how strong, how fast, how regular, how complex. A machine
learning model then looks at those numbers — not the raw wiggle — to tell
rest from mental-arithmetic task.

For every 4-second epoch, from every one of your channels, we compute
dozens of numbers. Multiply by all channels and you get **over 1,000 features per
epoch**. That sounds like a lot, but it's actually standard for EEG research
— this is what real published papers on this exact dataset do.

---

## 2. The five families of features

### A) Time-domain features — "what does the raw wave look like?"

These are basic shape statistics, like describing a mountain range by its
average height, how bumpy it is, etc.

| Feature | Plain-English meaning |
|---|---|
| `mean`, `std`, `variance` | How big the signal is on average, and how much it varies |
| `skewness` | Is the wave lopsided (more spikes up than down, or vice versa)? |
| `kurtosis` | Are there occasional extreme spikes, or is it smooth throughout? |
| `rms` | Overall signal "loudness" |
| `peak_to_peak` | Distance between the highest and lowest point |
| `zero_crossing_rate` | How often the signal crosses zero — a rough proxy for how "fast" it oscillates |
| `line_length` | Total up-and-down distance traveled — higher = more erratic signal |
| `hjorth_activity/mobility/complexity` | Three classic EEG descriptors: activity = signal power, mobility ≈ average frequency, complexity ≈ how much the frequency itself changes |

**Why it matters:** during mental effort, brain signals typically get
faster and less "smooth" — these features catch that directly from the raw
waveform, no frequency analysis needed.

---

### B) Frequency-domain features — "which brain rhythms are strongest?"

The brain doesn't just make noise — it oscillates in distinct rhythms
("bands"), and each rhythm is linked to a different mental state:

| Band | Frequency | Loosely means... |
|---|---|---|
| Delta | 1–4 Hz | Deep rest / drowsiness |
| Theta | 4–8 Hz | **Increases with mental effort**, especially at the front of the head |
| Alpha | 8–13 Hz | **Drops when you're actively focused** (it's the "resting" rhythm) |
| Beta | 13–30 Hz | Active thinking, alertness |
| Gamma | 30–45 Hz | High-level processing, but noisier / less reliable |

For each band we compute:
- **`_power`** — raw strength of that rhythm
- **`_relative_power`** — that band's strength as a % of total brain activity
  (accounts for the fact that some people just have "louder" EEG overall)

Plus derived numbers used specifically for mental-workload research:
- **`engagement_index` = beta / (alpha + theta)** — the most-cited single
  number for "how engaged/focused is this person right now." Higher =
  more engaged.
- **`theta_alpha_ratio`**, **`theta_beta_ratio`**, **`alpha_theta_ratio`** —
  alternate versions of the same idea, all shown in published studies to
  track cognitive workload.
- **`peak_frequency`** — the single strongest rhythm in the whole signal
- **`spectral_edge_95`** — the frequency below which 95% of the signal's
  energy sits (a rough "how fast is this signal overall" number)
- **`spectral_entropy`** — how "spread out" vs. "concentrated" the energy is
  across all frequencies (low = signal dominated by one clear rhythm, high =
  messy mix of many rhythms)

**Why it matters:** this is the core evidence for your "frontal lobe worked
harder" claim — if frontal channels show theta power going up and alpha
power going down from rest → task, that's the textbook EEG signature of
increased cognitive engagement in that region.

---

### C) Wavelet features — "how does the mix of rhythms change over time?"

Frequency-domain features above treat the whole 4-second epoch as one
snapshot. But brain activity isn't static within those 4 seconds — a
**wavelet transform** breaks the signal into several time-frequency "levels"
(like a zoom lens that can see both fast wiggles and slow drifts at once).

- **`wavelet_energy_L0...L5`** — how much signal strength is in each level
  (each level roughly corresponds to a frequency range)
- **`wavelet_relative_energy_L0...L5`** — same, as a percentage
- **`wavelet_entropy`** — how evenly spread the energy is across levels

**Why it matters:** this is what the highest-accuracy published EEGMAT paper
in your literature review used (wavelet transform + statistical features,
99%+ accuracy) — it captures patterns that a single-snapshot frequency
analysis can miss.

---

### D) Nonlinear / entropy features — "how complex or predictable is the signal?"

The brain isn't a simple repeating wave — it's chaotic and irregular by
nature. These features quantify *how* irregular:

| Feature | Plain-English meaning |
|---|---|
| `permutation_entropy` | How unpredictable the up/down pattern is |
| `svd_entropy` | How many independent "directions" of variation the signal has |
| `approximate_entropy`, `sample_entropy` | How repetitive vs. novel the signal's patterns are over time |
| `petrosian_fd`, `katz_fd`, `higuchi_fd` | "Fractal dimension" — roughly, how jagged/rough the line is (imagine measuring a coastline: rougher = higher number) |
| `detrended_fluctuation` | Whether the signal has long-range structure or is just random noise |
| `num_zero_crossings` | (same idea as zero-crossing rate above) |

**Why it matters:** mental effort tends to make brain signals *more*
complex/irregular in certain bands. These features are a strong, less
commonly-used complement to plain band power, and several top studies on
this dataset used exactly these (entropy + fractal dimension) as key
features.

---

### E) Connectivity features — "are different brain regions talking to each other?"

Unlike A–D (computed per channel, independently), these look at **pairs of
channels together** — e.g., is the front-left region doing the same thing as
the front-right region at the same time?

- **`corr_X_Y`** — simple correlation: do these two spots rise and fall
  together?
- **`plv_X_Y`** (Phase Locking Value) — do the two signals stay in a
  consistent rhythm relationship, even if their raw amplitude differs?
- **`coherence_theta_alpha_X_Y`** — specifically, how synchronized the two
  regions are in the theta/alpha bands (the cognitive-load bands)

**Why it matters:** the highest-accuracy multi-domain stress study found
functional connectivity *decreases* under stress — so this isn't just
"extra" features, it's shown in the literature to be independently
informative from power/entropy features.

---

## 4. Research backing (what this is built on)

- **Wavelet + statistical features** — a published EEGMAT-specific study
  using discrete wavelet transform and statistical features for mental
  stress classification during the arithmetic task.
- **Entropy + time-frequency features** — a stacked LSTM study on this exact
  dataset combining nonlinear (entropy) features with time-frequency
  features across 21 channels, reporting a high-accuracy ensemble result.
- **Convolutional Spiking Neural Networks on this dataset** — used Discrete
  Wavelet Transform features, reaching 98.75% accuracy in 10-fold CV.
- **Multi-domain functional connectivity + spectral features** — a 22-subject
  study combining Phase Locking Value connectivity with time/frequency
  features (210 features total), reaching 93.2% accuracy with SVM, and
  showing connectivity *decreases* under stress.
- **Engagement index & band ratios** — beta/(alpha+theta), theta/alpha,
  and theta/beta ratios are established mental-workload indices from
  psychophysiology research (Mikulka et al. 2002; Chikhi et al. 2022
  meta-analysis confirming frontal theta as the most reliable single
  workload marker).
- **PyEEG / AntroPy / eeglib** — the open-source Python libraries this
  module's entropy, fractal-dimension, and Hjorth-parameter functions are
  based on, used broadly across EEG research for exactly this kind of
  feature set.

---

## 5. Quick summary for your project write-up

> "We extracted a comprehensive feature set per EEG epoch, following
> established EEG/cognitive-workload research: time-domain shape statistics,
> frequency-band power and workload ratios (engagement index,
> theta/alpha), wavelet time-frequency energy, nonlinear complexity/entropy
> measures, and cross-channel functional connectivity (correlation, phase
> locking, coherence). This mirrors the feature sets used in published
> EEGMAT stress-classification studies achieving 93–99% accuracy."

That single paragraph, plus the feature-family table above, is enough for a
non-technical reader (or your professor, if they're not an EEG specialist)
to understand what you built and why each piece is there.
