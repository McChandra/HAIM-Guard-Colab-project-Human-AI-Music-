# HAIM-Guard: AI-Generated Music Cybersecurity Project

**Author:** Manus AI  
**Primary deliverable:** `HAIM_Guard_AI_Music_Cybersecurity.ipynb`

## Project summary

HAIM-Guard is a Google Colab-ready research project for detecting **fully synthetic music, unseen generator outputs, post-processed synthetic music, and hybrid human–AI production**. It frames synthetic-music detection as one security layer within a larger provenance and abuse-prevention program. The notebook trains a compact ensemble, evaluates a real-only anomaly branch, calibrates scores, creates a human-review grey zone, and measures degradation under common laundering transformations.

> **Decision boundary:** A model score is a triage signal, not proof of copyright infringement, impersonation, missing consent, or unlawful behavior. High-impact decisions require human review, provenance evidence, and an appeal process.

## Recommended dataset

The most representative public dataset identified is **HAIM (Human–AI Music)**. Unlike benchmarks limited to fully generated songs or synthetic singing voices, HAIM includes fully human and fully AI tracks, AI mastering of human music, human mastering or mixing of AI music, AI vocal covers, human lyrics with AI generation, AI variation/edit/repaint, and human–AI temporal concatenation and crossfades.[1] The current public release describes **153,686 track records**, including 67,000 hosted audio tracks and 86,686 link-based records, with approximately 245 GB of hosted files.[2]

| Resource | Link | Practical note |
| --- | --- | --- |
| HAIM dataset page | [Hugging Face](https://huggingface.co/datasets/mippia/HAIM) | Official downloads, preview subsets, license, folder structure |
| HAIM repository | [GitHub](https://github.com/Mippia/HAIM_dataset) | Metadata, generation scripts, legal/ethical supplement |
| HAIM paper | [arXiv](https://arxiv.org/abs/2606.01686) | Taxonomy and benchmark methodology |
| Full download command | `hf download mippia/HAIM --repo-type dataset --local-dir HAIM` | Approximately 245 GB; not recommended for ordinary Colab |

The dataset and its hosted audio are provided under **CC BY-NC 4.0** for non-commercial research. Commercial-platform material is represented by URL manifests where required.[2] The HAIM paper’s initial 196,000-track description differs from the current release card’s 153,686-track total; the notebook uses the current public release for download planning and records this version discrepancy.[1] [2]

## Why the notebook defaults to Quick Mode

The official HAIM dataset page warns that its dataset-viewer configurations are small previews and are not the full corpus.[2] The notebook intentionally uses these previews by default because they provide directly accessible audio from multiple baseline, hybrid, and temporal categories without silently consuming hundreds of gigabytes. The configurable default downloads a subset sufficient to demonstrate the full security workflow. An optional, disabled Research Mode cell uses `snapshot_download(..., allow_patterns=...)` for selected raw folders.

## Architecture

HAIM-Guard combines three complementary signals. A standardized logistic regression provides an interpretable baseline. A histogram gradient-boosting classifier captures nonlinear combinations of forensic features. A real-only PCA and shrinkage-covariance model assigns an anomaly distance without using synthetic examples during its fit; this follows the real-distribution principle introduced by MusicDET but is a deliberately compact approximation rather than a reimplementation of MusicDET’s normalizing flows.[7]

The feature vector includes log-mel and MFCC summaries, chroma, spectral centroid, bandwidth, rolloff, flatness, zero-crossing rate, RMS energy, and a log-frequency shift-insensitive fingerprint. The fingerprint maps a time-averaged spectrum to a logarithmic frequency grid, removes a smooth baseline, and summarizes translation-invariant Fourier magnitude. This design is inspired by public Fourier-artifact analyses and the 2026 speed-invariant detector, but the notebook explicitly avoids claiming the source model’s formal robustness guarantee.[6] [8]

| Layer | Purpose | Security rationale |
| --- | --- | --- |
| Track-safe splitting | Prevent crops from one track entering different partitions | Limits leakage and inflated accuracy |
| Generator holdout | Never train on one fully synthetic source | Measures practical unseen-generator behavior |
| Real-only anomaly model | Model human music without requiring every fake generator | Reduces dependence on known attack families |
| Constrained augmentation | Improve resilience to benign and adversarial processing | Reduces brittleness without hiding held-out attacks |
| Calibration | Convert scores into a more interpretable risk scale | Supports threshold governance and auditability |
| Abstention | Return `uncertain—human review` in a grey zone | Reduces false accusations and automatic overreach |
| Hybrid challenge sets | Evaluate B1–B9 and C1–C2 separately | Avoids collapsing role-level AI involvement into a false binary truth |

## Evaluation protocol

The baseline supervised model trains on human music and two synthetic sources. A third source, Lyria Pro 3 by default, is held out as an unseen generator. Human tracks are split deterministically before crop extraction. Three crops from each track are scored and aggregated, so all metrics are track-level.

The notebook reports ROC-AUC, PR-AUC, F1, balanced accuracy, equal-error rate, Brier score, expected calibration error, and false-positive rate at 90% true-positive rate. It additionally reports worst-generator and worst-corruption outcomes. Bootstrap confidence intervals are computed by resampling tracks rather than crops.

The robustness matrix covers gain change, additive noise, resampling round-trip, low-pass filtering, clipping, time shift, time stretch, pitch shift, 64 kbps MP3, and a combined transformation. Both real and synthetic tracks receive each transformation; otherwise the transformation itself could become a class shortcut.

## Published SOTA comparison

Published results are included as references and are never presented as locally reproduced unless the optional adapter actually runs. Different datasets, durations, and split protocols make direct rank ordering invalid.

| Public model | Published evaluation | Reported result | Interpretation |
| --- | --- | --- | --- |
| MusicDET, real-only | FakeMusicCaps cross-generator | Average EER 4.51% | Strong zero-shot result; trained only on real music.[7] |
| Class-conditional MusicDET | FakeMusicCaps cross-generator | Average EER 0.89% | Uses synthetic training data; not zero-shot.[7] |
| MusicDET, real-only | SONICS cross-generator | Average EER 2.89% | Four-second protocol.[7] |
| CLAM | MoM generator-disjoint OOD | Accuracy 93.1%; F1 92.5% | Strong OOD benchmark with multiple unseen generators.[4] |
| CLAM | SONICS | F1 99.3% | Indicates possible benchmark saturation.[4] |
| SpecTTTra-α | SONICS, 120 seconds | F1 97.2% | Long-context architecture with public checkpoint.[3] |
| Fusion Segment Transformer, MERT | SONICS | F1 99.99% | Authors note a possible SONICS resampling shortcut.[5] |
| Fusion Segment Transformer, MERT | AIME | F1 98.68%; AUC 99.95% | Strong in-domain full-audio result.[5] |
| Speed-invariant log-frequency detector | Suno v5 under speed attack | AUC 99.7%; F1 98.6% | Robust to speed scaling by design, but not all corruptions or generators.[8] |

A key conclusion is that **clean in-domain performance is not equivalent to security**. When SONICS-trained SpecTTTra was evaluated on MoM’s unseen-generator subsets, reported F1 scores fell to 53.46% for Riffusion, 68.80% for YuE, 50.94% for voice clones, and 64.58% for Suno 4.[4] MusicDET’s reported average EER on FakeMusicCaps was 4.51% in its clean real-only setting, but rose to 44.73% under pitch shift, 44.11% under white noise, and 41.75% after 64 kbps MP3.[7] The notebook therefore treats corruption and generator holdouts as first-class tests.

## Running the notebook

Open `HAIM_Guard_AI_Music_Cybersecurity.ipynb` in Google Colab and execute cells from top to bottom. A GPU is optional for the compact default model. Quick Mode uses the public HAIM preview files and caches downloads and extracted features under `/content/haim_guard_cache`. Results are written to `/content/haim_guard_outputs`.

The most useful configuration fields are `baseline_tracks_per_source`, `challenge_tracks_per_category`, `train_generators`, `unseen_generator`, `target_validation_fpr`, and `robustness_tracks_per_class`. Increase sample counts cautiously because feature extraction and codec robustness tests are the slowest steps.

The official SpecTTTra adapter is disabled by default. Set `RUN_OFFICIAL_SPECTTTRA=True` to install the public SONICS package, download its 5-second checkpoint, and evaluate it on the same HAIM human and unseen-generator files.[3]

## Outputs

| Artifact | Purpose |
| --- | --- |
| `haim_guard_model.joblib` | Serialized feature-model ensemble and calibration objects |
| `metrics_clean.csv` | In-domain and unseen-generator metrics |
| `metrics_corruptions.csv` | Corruption-by-corruption metrics and F1 degradation |
| `metrics_hybrid.csv` | Hybrid and temporal category sensitivity/uncertainty rates |
| `predictions.csv` | Track-level scores and policy labels |
| `predictions_corruptions.csv` | Per-track corrupted-audio scores |
| `threshold_policy.json` | Binary and grey-zone thresholds |
| `model_card.md` | Intended use, measured results, limitations, and governance |
| `clean_roc_pr_calibration.png` | ROC, precision–recall, and calibration plots |
| `hybrid_category_scores.png` | Score distributions across HAIM hybrid categories |
| `corruption_f1.png` | Worst-first corruption robustness chart |

## Threat reduction and residual risk

HAIM-Guard reduces the chance that a detector passes a laboratory test while failing on simple real-world changes. It explicitly measures unseen generators, hybrid production, post-processing, calibration, and false-positive control. It also encourages an operational policy that preserves original evidence, logs model and configuration versions, and escalates uncertain cases.

It does not prevent training-data theft, secure generative-model infrastructure, enforce copyright, authenticate artist identity, or establish consent. These require additional controls such as signed provenance, access management, protected training pipelines, data-loss prevention, watermarking where appropriate, account-abuse detection, upload rate limits, royalty-fraud analytics, and contractual governance.

## References

[1]: https://arxiv.org/abs/2606.01686 "HAIM: Human-AI Music Datasets for AI Music Production Tracking Benchmark"
[2]: https://huggingface.co/datasets/mippia/HAIM "HAIM dataset card and downloads"
[3]: https://github.com/awsaf49/sonics "SONICS dataset, SpecTTTra code, checkpoints, and benchmark"
[4]: https://arxiv.org/abs/2512.00621 "Melody or Machine: Detecting Synthetic Music with Dual-Stream Contrastive Learning"
[5]: https://arxiv.org/abs/2601.13647 "Fusion Segment Transformer: Bi-Directional Attention Guided Fusion Network"
[6]: https://github.com/deezer/ismir25-ai-music-detector "A Fourier Explanation of AI-Music Artifacts—code"
[7]: https://arxiv.org/abs/2605.18072 "MusicDET: Zero-Shot AI-Generated Music Detection"
[8]: https://arxiv.org/abs/2607.27454 "Improved Robustness in AI-Generated Music Detection"
[9]: https://zenodo.org/records/15063698 "FakeMusicCaps dataset"
[10]: https://singfake.org/ "SingFake: Singing Voice Deepfake Detection"
