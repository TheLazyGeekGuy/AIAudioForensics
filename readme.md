# 🧠 AI Audio Forensics — Detection Methods & Perturbation Chains

> **Project:** AI Music & Vocal Generation Detection  
> **Version:** 1.1  
> **Author:** TheLazyGeekGuy Research  
> **Last update:** 2025-10-24

## 1. Introduction

With the massive rise of AI-generated music via platforms like Suno, Udio, Boomy and distribution services such as DistroKid, audio forensic detection methods are becoming essential.

AI-generated audio can be identified by:
- **Metadata patterns**
- **Temporal regularities**
- **Spectral artifacts**
- **Prosodic anomalies**
- **Distribution fingerprinting**

This article documents forensic detection pipelines, common generator traces, and the kinds of processing chains that may **degrade detection confidence**.

---

## 2. Metadata & Distribution Fingerprints

### 2.1 Typical Metadata Fingerprints

| Field                    | Indicator                                                                 |
|--------------------------|----------------------------------------------------------------------------|
| ISRC / UPC               | Batch patterns, identical prefixes, auto-issued series                     |
| Encoder String           | Same `LAME 3.100` / FFmpeg signatures across “different artists”           |
| Encoding Date            | Creation timestamp ≈ Upload timestamp                                     |
| Label / Producer Fields  | Missing or repeated                                                        |
| Duration vs File Size    | Abnormal compression ratio (AI output is often size-optimized)            |

**Tools:** `ffprobe`, `exiftool`, `mediainfo`, `mutagen`.

### 2.2 Cross-Release Analysis
- Correlating ISRCs can reveal single origin accounts.
- Recurrent encoder hashes = same generation pipeline.

**Reference:**  
- [DistroKid Distribution FAQ](https://distrokid.zendesk.com/)  
- [ISRC Handbook (IFPI)](https://isrc.ifpi.org/en/)

---

## 3. Temporal Domain Analysis

### 3.1 Envelope Coherence
- Synthetic tracks display overly smooth ADSR envelopes.
- No micro-dynamics (room tone, breath noise).

### 3.2 Transient Jitter
- Transients are perfectly aligned on the grid.
- Real human performance shows timing drift.

### 3.3 Loop Signatures
- Seams every 4/8/16 bars are common in AI outputs.
- Abrupt or seamless looping is detectable in cumulative energy plots.

**Tools:** `librosa`, `essentia`, `Sonic Visualiser`.

**Reference:**  
- [Librosa Documentation](https://librosa.org/doc/latest/)  
- [Bittner et al., 2022 - Temporal Analysis of AI Music](https://arxiv.org/abs/2207.04652)

---

## 4. Frequency & Spectral Domain Analysis

### 4.1 High Frequency Roll-Off
- Sudden energy drop above 18–20 kHz indicates AI-generated content.
- Studio recordings typically exhibit analog noise, air band content.

### 4.2 Harmonic Fingerprinting
- AI models produce **mathematically clean** harmonic series.
- Missing detuning and chaotic upper partials.

### 4.3 Aliasing / Interpolation Patterns
- Detectable “stairs” or smooth bands in spectrograms due to neural vocoder interpolation.

**Tools:** `Sonic Visualiser`, `Audacity`, `matplotlib + librosa`.

**Reference:**  
- [Fourier Analysis in Synthetic Audio](https://arxiv.org/abs/2305.09356)  
- [SONICS Project Whitepaper, 2024](https://sonics.audio/whitepaper)  
- [OpenAI Audio Watermarking Proposal](https://openai.com/research/audio-watermarking)

---

## 5. Structural & Musical Pattern Analysis

| Feature                    | AI Music Pattern                                 | Human Pattern                                      |
|----------------------------|--------------------------------------------------|----------------------------------------------------|
| Rhythm                     | Perfect quantization                             | Swing, groove, irregularity                         |
| Chords                     | Standard 4-chord pop loop                         | Unique variations, bridges                          |
| Stereo Imaging             | Static                                           | Dynamic                                            |
| BPM                        | Fixed, grid-perfect                              | Micro-fluctuations                                 |

**Reference:**  
- [“Statistical Patterns in Neural Music Generation” – Huang et al., 2024](https://arxiv.org/abs/2401.04201)

---

## 6. Prosody & Voiceprint Analysis

### 6.1 Vocal Fingerprints
- Flat emotional prosody
- Regular breathing gaps or none at all
- Lack of room signature
- Unnatural formant behavior

### 6.2 Voiceprint Matching
- MFCC and formant vector clustering can identify the same vocal model across different releases.

**Tools:** `praat`, `pyworld`, MFCC clustering libraries.

**Reference:**  
- [Bozkurt et al., “Voiceprint Detection of Synthetic Vocals”, 2023](https://arxiv.org/abs/2310.03788)

---

## 7. Perturbation Testing (Forensic)

| Technique                       | Expected AI Reaction                                  | Real Audio Reaction                                 |
|----------------------------------|------------------------------------------------------|-----------------------------------------------------|
| +0.5% time-stretch              | Reveals seam artifacts                                | Minimal perceptual change                           |
| High shelf @ 20kHz              | Exposes roll-off                                     | Subtle brightness shift                             |
| MP3 re-encode                   | Amplifies generation noise                           | Stable structure                                    |
| Phase inversion / jitter       | Reveals layered synthesis artifacts                   | Natural degradation                                 |

---

## 8. Cross-Release Signature Matching

- **Spectral similarity clustering** between multiple “artists” can unmask single generator sources.
- Same mastering chain → identical noise floor, harmonic curve.
- Same vocal model → identical MFCC centroid.

**Reference:**  
- [Transformer-based Audio Fingerprinting, 2025](https://arxiv.org/abs/2503.01845)

---

## 9. ML-Based Automated Detection

- **Features**: MFCCs, spectral flux, chroma vectors, transient maps  
- **Models**: CNN, Transformer encoder, Siamese for clustering
- **Labels**: AI vs Real, or Generator Type A/B/C

```mermaid
flowchart LR
A[Metadata] --> B[Temporal Analysis]
B --> C[Spectral Analysis]
C --> D[Pattern Clustering]
D --> E[Perturbation Tests]
E --> F[ML Classification]
F --> G[Decision & Risk Scoring]
```

---

## 10. VST / FX Chains That Degrade Detectability (⚠️ To Avoid)

| Plugin / FX Chain Type                             | Effect on Detection                                                    | Why it Matters                                                                                              |
|----------------------------------------------------|-------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------|
| Heavy mastering limiters (e.g. FabFilter Pro-L 2, Waves L2) | Reduces dynamic range and spectral variance                             | Flattens transients and high-frequency fingerprints                                                         |
| Exciters / Harmonic Enhancers (e.g. Soundtoys Decapitator, Aphex Exciter) | Adds synthetic harmonics that blur natural roll-off                      | Creates false “analog-like” cues                                                                            |
| Stereo Widener Chains (e.g. Ozone Imager, Melda MStereoExpander) | Artificial stereo correlation masking generator patterns                 | Breaks spatial detection patterns used in clustering                                                        |
| Multi-band compressors with auto gain          | Normalizes AI signature inconsistencies                                 | Reduces classifier confidence                                                                               |
| Tape/Analog Emulation (e.g. Waves J37, UAD Ampex ATR-102, Soundtoys Radiator) | Adds low-level noise floor + HF bloom that mimics real studio chains     | Makes forensic separation between synthetic and analog less reliable                                       |
| Transient Shapers                              | Smoothens or exaggerates transient irregularities                       | Obscures the temporal quantization traces typical of AI audio                                               |
| Auto-Mastering Chains / Presets (e.g. iZotope Ozone 11 AI Mastering) | Homogenizes tracks                                                      | Removes generator-specific spectral identity                                                                |

🛑 *These plugins are not malicious. But their use can significantly reduce the reliability of forensic AI audio detection by introducing artifacts that mimic analog or human performance characteristics.*

👉 **Recommendation:**  
- Keep forensic reference stems **pre-FX**.  
- Document FX chains separately.  
- Prefer minimal processing during detection.

---

## 11 – SubmitHub AI Song Checker

### Overview

The **AI Song Checker** is an experimental classifier developed by **Jason Grishkoff** (SubmitHub) that estimates whether a music track is AI-generated or human-made.  
It is built using a **Random Forest Classifier**, trained on a balanced dataset (~4,000 samples) combining both AI-generated and human-composed songs.

**Accuracy:** ~90% on the test set.  
**Limitations:** quality-dependent, genre bias, and reduced efficiency on post-mastered or mixed (AI + human) tracks.

---

### ⚙️ Model Architecture

#### Classifier
- **Type:** Random Forest (binary classification)
- **Target:** “AI-Generated” vs “Human-Made”
- **Input:** Statistical audio descriptors
- **Output:** Probability score (0–1)

#### Training Data

| Type | Quantity | Source | Comment |
|------|-----------|---------|----------|
| AI-Generated | ~2,000 | Various (Suno, Udio, etc.) | Includes instrumental + vocal tracks |
| Human-Made | ~2,000 | Publicly released songs | Balanced dataset |

Balanced dataset ensures the model avoids class bias and generalizes better across sources.

---

### 🎛 Feature Extraction Pipeline

The system extracts **21 low- to mid-level descriptors** across three analytical domains:

#### 1. Spectral Features

| Feature | Description |
|----------|--------------|
| Spectral Flatness | Differentiates tonal vs. noise-like signals |
| Spectral Rolloff | Frequency threshold containing most spectral energy |
| RMS Energy | Average loudness level |
| Zero Crossing Rate | Waveform polarity change frequency |
| MFCC Statistics | Mel-frequency cepstral coefficients summarizing timbre |

#### 2. Harmonic Analysis

| Feature | Description |
|----------|-------------|
| Phase Coherence | Temporal stability of phase relationships |
| Frequency Band Ratios | Energy distribution across low/mid/high bands |
| Harmonic Consistency | Persistence of harmonic structure |
| Harmonic Stability | Time-based steadiness of harmonic content |
| Pitch Transition Rate | Velocity of pitch movement |
| Harmonic Complexity | Density of harmonic variations |
| Vocals/Music Ratio | Energy ratio between vocal and instrumental sources |

#### 3. Long-Range Temporal Patterns

- Cross-sectional correlation over **1, 2, and 3-minute windows**
- Computes:
  - **Variance**, **standard deviation**, **correlation coefficients**
  - Long-term structure stability and section coherence

These parameters approximate the *temporal logic* that human composition tends to follow, as opposed to the statistically homogeneous textures often observed in AI-generated material.

---

### 🧩 Limitations & Biases

| Limitation | Impact |
|-------------|--------|
| Short or long tracks (<32s or >3min) | Inconsistent features |
| Low bitrate (<196 kbps) | Unreliable spectral data |
| Genre bias | Some genres (jazz, ambient) deviate from training set patterns |
| Mastered AI tracks | Human mastering masks artefacts |
| Mixed sources (AI vocals + human instrumental) | Confuses classifier |
| Instrumental-only tracks | Fewer vocal-related metrics → less accurate |

---

### 🔬 Observations & Relevance

- The **AI Song Checker** uses a **feature-based deterministic classifier**, unlike deep models (CNN, spectrogram Transformers) used by **SONICS**, **Deezer DetectAI**, or **Udio internal filters**.  
- Its design confirms that *AI fingerprinting* is primarily detectable via:
  - **Spectral uniformity**
  - **Phase alignment regularities**
  - **Reduced harmonic entropy**
  - **Unnatural coherence over time windows**

These patterns match those observed in other academic works (e.g., *SONICS*, 2023) — especially the spectral smoothness and low-variance dynamics typical of AI generation pipelines.

---

### 🧠 Future Improvements (v4.0 ideas)

- Larger dataset (×10)  
- GPU-based inference  
- Separate model for vocals vs. instruments  
- Persistent user history tracking  
- Retraining with new AI models (Udio v3, Suno v4, Mubert, etc.)


---

## 11. References

- Bittner, R. et al. (2022) *Temporal Analysis of AI Music*. [arXiv:2207.04652](https://arxiv.org/abs/2207.04652)  
- Huang, C. et al. (2024) *Statistical Patterns in Neural Music Generation*. [arXiv:2401.04201](https://arxiv.org/abs/2401.04201)  
- Bozkurt, S. et al. (2023) *Voiceprint Detection of Synthetic Vocals*. [arXiv:2310.03788](https://arxiv.org/abs/2310.03788)  
- SONICS Project (2024). *Synthetic Audio Fingerprinting Whitepaper*. [sonics.audio](https://sonics.audio/whitepaper)  
- OpenAI (2024). *Audio Watermarking Proposal*. [openai.com](https://openai.com/research/audio-watermarking)  
- IFPI (2023). *ISRC Handbook*. [isrc.ifpi.org](https://isrc.ifpi.org/en/)  
- Librosa Docs: [https://librosa.org](https://librosa.org)  
- Transformer Audio Fingerprinting (2025). [arXiv:2503.01845](https://arxiv.org/abs/2503.01845)
- Jason Grishkoff, “AI Song Checker,” *SubmitHub Blog* (2024).  
  [https://www.submithub.com/ai-song-checker](https://www.submithub.com/ai-song-checker)  
- Ben Jordan (YouTube): “AI Music Detector Experiment,” 2024.  

---

## 12. License

This repository is provided for **research, educational and defensive purposes only.**  
**Do not** use this information to conceal, obfuscate, or distribute synthetic audio maliciously.
