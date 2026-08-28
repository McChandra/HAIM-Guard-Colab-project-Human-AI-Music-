# HAIM-Guard Validation Report

**Date:** 28 August 2026  
**Author:** Manus AI

## Validation outcome

The delivered notebook passed automated JSON and Python-syntax validation. It contains 27 cells: 10 Markdown cells and 17 code cells, with nine numbered primary-source references. All required security components were found, including the generator-disjoint test, real-only anomaly model, corruption suite, EER and calibration metrics, false-positive-constrained threshold, abstention policy, optional public SpecTTTra adapter, and artifact export.

## End-to-end smoke run

A reduced copy of the notebook was executed from its first runtime cell through final export using real audio obtained from the official HAIM preview API. The test downloaded baseline, hybrid, and temporal tracks; decoded audio; extracted crop-level features; fitted supervised and real-only models; calibrated track scores; evaluated in-domain and unseen-generator subsets; ran all eleven corruption conditions; produced the hybrid-category analysis; and saved the complete model bundle and documentation.

| Check | Result |
| --- | --- |
| Public HAIM preview retrieval | Passed |
| Audio decoding and deterministic cropping | Passed |
| Feature extraction and cache writing | Passed |
| Track-safe model fitting and calibration | Passed |
| In-domain, unseen-generator, and generator-wise evaluation | Passed |
| Hybrid and temporal challenge evaluation | Passed |
| Eleven-condition corruption benchmark | Passed |
| MP3 64 kbps round-trip through FFmpeg | Passed |
| Model, metrics, predictions, policy, and model-card export | Passed |
| Plot generation and visual inspection | Passed |

The smoke run produced all twelve expected artifacts: the serialized model, three primary metric tables, two prediction tables, configuration and threshold files, a model card, and three PNG figures. The figures were visually inspected for legibility.

## Interpretation limit

The reduced run used eight baseline tracks per source, two challenge tracks per category, and two robustness tracks per class. Its numerical scores are therefore **not model-quality claims**. The smoke test validates executable code paths and reproducibility only. Users should run the notebook’s larger defaults—or Research Mode with raw HAIM folders—and report bootstrap intervals and worst-group results before drawing performance conclusions.

## Structural validation record

The machine-readable validation record is available in `validation_report.json`. Its final error list is empty.
