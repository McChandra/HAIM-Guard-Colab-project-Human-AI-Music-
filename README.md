# HAIM-Guard-Colab-project-Human-AI-Music-
Cybersecurity Issues in Music Where AI is Used  The integration of AI in music creation has led to several cybersecurity issues, including data privacy concerns, manipulation of AI-generated content, and ethical hacking risks. These issues arise from the need to protect sensitive data and ensure the integrity of AI models
HAIM-Guard

A threat-aware AI/ML project for detecting AI-generated and hybrid music

HAIM-Guard is a Google Colab-ready research project for identifying fully synthetic music, outputs from unseen generators, post-processed synthetic audio, and hybrid human–AI production. It is designed around a cybersecurity principle that ordinary random-split accuracy is insufficient: a useful detector must also be tested against generator shifts, audio laundering, calibration failure, data leakage, and false-positive risk.


Important: HAIM-Guard is a forensic triage tool, not proof of copyright infringement, authorship, impersonation, missing consent, or illegal conduct. Its scores should support human review rather than automatic punitive action.

Project files

File
Description
HAIM_Guard_AI_Music_Cybersecurity.ipynb
Main Google Colab notebook
PROJECT_GUIDE.md
Detailed methodology, threat analysis, and benchmark guide
VALIDATION_REPORT.md
Structural validation and real-data smoke-test results
requirements.txt
Python dependencies for local execution
validation_report.json
Machine-readable notebook validation record




Primary dataset

The project uses HAIM—Human–AI Music as its primary dataset. HAIM was selected because it represents more of the real music-production chain than datasets limited to fully generated songs or synthetic vocals. It contains fully human and fully AI music, AI mastering of human tracks, human mastering or mixing of AI tracks, AI vocal covers, AI-generated variations and edits, human lyrics with AI generation, and temporal human–AI mixtures.

The current public release describes 153,686 track records, including 67,000 hosted audio tracks and 86,686 link-based records. The hosted files require approximately 245 GB, so HAIM-Guard uses official public preview subsets by default and provides an optional selective-download mode for larger experiments.

Dataset resource
Link
Official HAIM dataset
huggingface.co/datasets/mippia/HAIM
Official repository
github.com/Mippia/HAIM_dataset
Research paper
arxiv.org/abs/2606.01686




HAIM data are released under CC BY-NC 4.0 for non-commercial research. Review the dataset card and the terms governing linked third-party material before use.

Security objectives

HAIM-Guard addresses several practical failure modes in AI-music detection.

Threat or vulnerability
Project control
Previously unseen generator
Generator-disjoint train/test protocol
Dependence on known fake examples
Real-only anomaly-detection branch
MP3 or signal-processing evasion
Eleven-condition corruption benchmark
Speed and pitch manipulation
Log-frequency forensic features and explicit tests
Human–AI hybrid production
Separate HAIM hybrid and temporal challenge sets
Track leakage between partitions
Tracks are split before crop extraction
Poorly calibrated confidence
Validation-only isotonic calibration
False accusations
Target-FPR threshold and human-review grey zone
Benchmark cherry-picking
Worst-generator and worst-corruption reporting
Reproducibility failure
Fixed seed, configuration hash, cached features, and model card




Model architecture

The default system is intentionally compact enough for an ordinary Colab session. It combines three complementary branches:

1.
A standardized logistic-regression baseline provides interpretable linear evidence.

2.
A histogram gradient-boosting classifier learns nonlinear combinations of forensic features.

3.
A real-only anomaly model uses standardized features, PCA, and Ledoit–Wolf covariance to measure deviation from human training music.

The final score fuses calibrated supervised probabilities with the real-only anomaly score. This design follows MusicDET’s principle of modeling the real-music distribution, while remaining a lightweight educational implementation rather than a reproduction of MusicDET’s normalizing-flow architecture.

The feature representation includes log-mel summaries, MFCCs, chroma, spectral centroid, bandwidth, rolloff, flatness, zero-crossing rate, RMS energy, and a compact log-frequency fingerprint. Three deterministic crops are extracted from each track and aggregated back to a track-level score.

Quick start in Google Colab

1.
Download HAIM_Guard_AI_Music_Cybersecurity.ipynb from this project.

2.
Open Google Colab.

3.
Select File → Upload notebook and upload the .ipynb file.

4.
Run the notebook from top to bottom.

5.
Review the generated files in /content/haim_guard_outputs.

A GPU is optional for the default compact model. The official SpecTTTra comparison cell can benefit from a GPU.

Local installation

Python 3.10 or later and FFmpeg are recommended.

Bash


python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
jupyter lab HAIM_Guard_AI_Music_Cybersecurity.ipynb



On Ubuntu, FFmpeg can be installed with:

Bash


sudo apt-get update
sudo apt-get install -y ffmpeg



Dataset modes

Quick Mode

Quick Mode is enabled by default. It downloads a configurable sample from the official HAIM audio-preview splits. This mode is suitable for demonstrations, debugging, coursework, and validating the security workflow without downloading the complete corpus.

The most useful configuration values are:

Python


baseline_tracks_per_source = 40
challenge_tracks_per_category = 5
train_generators = ('acestep', 'musicgen')
unseen_generator = 'lyria_pro3'
target_validation_fpr = 0.05
robustness_tracks_per_class = 10



These defaults support executable experimentation, not definitive performance claims. Increase the sample sizes for a formal study.

Research Mode

The disabled Research Mode cell uses selective Hugging Face downloads:

Python


from huggingface_hub import snapshot_download

snapshot_download(
    repo_id='mippia/HAIM',
    repo_type='dataset',
    local_dir='/content/HAIM_selected',
    allow_patterns=[
        'A_full_generation/A1_real/MTG-Jamendo music subset/**',
        'A_full_generation/A2_fake/musicgen/**',
    ],
)



Keep allow_patterns narrowly scoped unless sufficient storage is available. A complete download can be initiated with the following command, but is not recommended for a normal Colab runtime:

Bash


hf download mippia/HAIM --repo-type dataset --local-dir HAIM



Evaluation protocol

The supervised branches train on human music and two synthetic sources. A third source is withheld from all fitting and used as an unseen-generator test. Every crop from a track remains in the same partition. Calibration and metrics are computed at the track level rather than treating correlated crops as independent samples.

The notebook reports:

Metric
Purpose
ROC-AUC
Threshold-independent ranking quality
PR-AUC
Performance under class imbalance
F1 score
Precision–recall balance at the selected threshold
Balanced accuracy
Class-balanced thresholded performance
Equal-error rate
Operating point where false acceptance and rejection are equal
Brier score
Probabilistic accuracy
Expected calibration error
Agreement between confidence and observed outcomes
FPR at 90% TPR
False-positive cost at high sensitivity
Bootstrap interval
Track-level uncertainty estimate




Hybrid HAIM categories are not assigned misleading ordinary binary accuracy. The notebook reports their AI-alert and uncertainty rates by category because HAIM provides role-level AI involvement that cannot always be represented by one whole-track label.

Robustness benchmark

The corruption suite applies every transformation to both human and synthetic tracks, preventing the transformation itself from becoming a class shortcut.

Corruption
Security scenario
Gain reduction
Loudness normalization or deliberate gain change
Additive noise
Environmental noise or laundering
Resampling round trip
Platform conversion or detector evasion
Low-pass filtering
Bandwidth limitation or artifact removal
Mild clipping
Production distortion
Time shift
Crop and alignment instability
Time stretch
Tempo alteration
Pitch shift
Frequency displacement
MP3 at 64 kbps
Low-bitrate distribution
Combined transformation
Multi-step laundering attack




Results are displayed in worst-F1-first order and include ROC-AUC so threshold failure can be distinguished from complete loss of separability.

Published SOTA references

Published numbers below are reference results, not direct leaderboard rankings. The source studies use different datasets, segment lengths, training regimes, and metrics.

Public model
Published setting
Reported result
MusicDET, real-only
FakeMusicCaps cross-generator
Average EER 4.51%
Class-conditional MusicDET
FakeMusicCaps cross-generator
Average EER 0.89%
MusicDET, real-only
SONICS cross-generator
Average EER 2.89%
CLAM
MoM generator-disjoint evaluation
Accuracy 93.1%; F1 92.5%
CLAM
SONICS
F1 99.3%
SpecTTTra-α
SONICS, 120-second segments
F1 97.2%
Fusion Segment Transformer with MERT
SONICS
F1 99.99%
Fusion Segment Transformer with MERT
AIME
F1 98.68%; AUC 99.95%
Speed-invariant log-frequency detector
Suno v5 under speed attack
AUC 99.7%; F1 98.6%




The notebook includes an optional adapter for the official public SpecTTTra-α 5-second checkpoint, allowing a same-sample comparison on HAIM. It is disabled by default to keep the main workflow lightweight.

Generated outputs

After execution, the notebook writes the following artifacts to /content/haim_guard_outputs:

Artifact
Description
haim_guard_model.joblib
Serialized models, feature schema, calibrator, and thresholds
metrics_clean.csv
In-domain, unseen-generator, and generator-wise results
metrics_corruptions.csv
Corruption-level robustness metrics
metrics_hybrid.csv
Hybrid and temporal challenge results
predictions.csv
Track-level probabilities and policy decisions
predictions_corruptions.csv
Per-track corruption scores
threshold_policy.json
Binary and abstention thresholds
config.json
Reproducible experiment configuration
model_card.md
Intended use, results, limitations, and governance guidance
clean_roc_pr_calibration.png
ROC, precision–recall, and calibration plots
hybrid_category_scores.png
HAIM hybrid-category score distributions
corruption_f1.png
Thresholded and threshold-independent robustness chart




Decision policy

The model produces one of three review states:

State
Meaning
Recommended action
likely_human
Score is below the lower review threshold
Continue normal processing while retaining audit metadata
uncertain_human_review
Score falls inside the calibrated grey zone
Escalate to a qualified reviewer
ai_indicators_detected
Score is above the upper threshold
Review provenance and supporting evidence; do not treat the score as proof




The binary threshold is selected on validation data subject to a target human false-positive rate. Thresholds must be recalibrated when the deployment population changes.

Limitations

HAIM-Guard does not solve training-data privacy, artist identity verification, consent, copyright adjudication, or security of the underlying music-generation service. Its default preview sample is small, public datasets may contain collection artifacts or duplicates, and future generators may overlap with the human-music distribution. Heavy mastering, unusual genres, damaged recordings, codecs, and adaptive adversaries can cause errors.

High clean accuracy should not be mistaken for resilience. Published MusicDET results show substantial degradation under pitch shifting, additive noise, and low-bitrate codecs despite strong clean cross-generator performance. The newer speed-invariant detector improves one attack class but does not claim universal robustness across codecs, equalization, dynamics, and arbitrary future generators.

Recommended deployment controls

Use HAIM-Guard alongside signed provenance metadata, original-file hashing, upload-account abuse detection, duplicate and acoustic-fingerprint analysis, rate limiting, royalty-fraud monitoring, documented data governance, and a human appeal process. Log the model version, configuration hash, threshold policy, input-file hash, score, and reviewer outcome for every escalated case.

Validation

The notebook passed automated JSON and Python-syntax validation and was executed end to end in a reduced smoke configuration using real audio retrieved from the official HAIM preview service. The run validated download, decoding, feature extraction, training, calibration, unseen-generator testing, hybrid analysis, all corruption paths, visualization, and artifact export. See VALIDATION_REPORT.md for details.

Ethical use

Use this project for defensive research, platform integrity, provenance support, and transparent content review. Do not use it to harass artists, infer protected personal attributes, make unsupported public accusations, or automatically remove or demonetize content without appropriate evidence and review.

References

[1] HAIM: Human-AI Music Datasets for AI Music Production Tracking Benchmark
[2] HAIM dataset card and downloads
[3] SONICS dataset, SpecTTTra code, checkpoints, and benchmark
[4] Melody or Machine: Detecting Synthetic Music with Dual-Stream Contrastive Learning
[5] Fusion Segment Transformer for AI-Generated Music Detection
[6] A Fourier Explanation of AI-Music Artifacts—code
[7] MusicDET: Zero-Shot AI-Generated Music Detection
[8] Improved Robustness in AI-Generated Music Detection
[9] FakeMusicCaps dataset
[10] SingFake: Singing Voice Deepfake Detection
