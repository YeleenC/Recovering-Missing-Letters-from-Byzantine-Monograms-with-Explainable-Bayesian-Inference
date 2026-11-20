# Recovering-Missing-Letters-from-Byzantine-Monograms-with-Explainable-Bayesian-Inference
Explainable Bayesian engine to predict missing letters in Byzantine monograms (SUMAC '26). Documentation only, data not public.

Predicting missing letters in partially legible Byzantine monograms, using an explainable Bayesian scoring model built on corpus statistics.

Published at **SUMAC '26** (8th Workshop on analySis, Understanding and proMotion of heritAge Contents), Rio de Janeiro, November 10-14, 2026.
Paper: https://doi.org/10.1145/3840472.3841738

Authors: Yeleen Canonge, Soukaina Bouhaid, Saranga Kingkor Mahanta, Victoria Eyharabide (Sorbonne Université, Laboratoire STIH / UMR 8167 Orient & Méditerranée).
Work carried out within the ANR BHAI project (Byzantine Hybrid Artificial Intelligence), grant ANR-21-CE38-0001.

## Context

Byzantine lead seals are a major source for the social and institutional history of the Byzantine Empire. Their inscriptions often include monograms, which condense the letters of a personal name into a compact graphical composition. When a seal is damaged, reading a monogram means choosing among several possible readings from incomplete evidence.

This project addresses one task: given the letters already identified on a partially legible monogram, rank the most probable missing letters. It works on symbolic transcriptions (letter identity and position, as recorded by expert sigillographers), not on images.

## Contributions

1. **A dataset of 350 Byzantine monograms**, automatically extracted from the sigillographic catalog *The Seals of the Collection Adolfo Eidelstein: Caesarea Maritima* (675 seals), encoded in XML (one file per seal) and normalized with transposition tables that reduce 99 attested graphical variants to 24 canonical Greek letters.
2. **A Bayesian inference engine** that predicts missing letters by combining five sources of evidence computed from the corpus.

## Method

### Dataset construction
- Python extraction pipeline (regular expressions and structural analysis of the paragraphs) turning catalog entries into structured XML.
- Annotated fields: seal identifier, associated name, monogram type (block, cruciform, invocative), dating, letters and their positions, Greek reading, structural relations.
- Selection: monograms rated sufficiently legible in the catalog (three highest of four quality tiers), so that the catalog reading can serve as ground truth.
- Normalization in three representations: corrected (Unicode), palaeographic, compositional. The compositional mapping (ligatures decomposed, variants merged) is the reference vocabulary for the engine.

### Inference engine
For each candidate letter, a composite score is computed as a linear combination (linear opinion pool) of five probability estimates:

| Component | Evidence | Weight |
|---|---|---|
| P1 | Letter frequency | 0.15 |
| P2 | Co-occurrence with observed letters (pairs, triplets, quadruplets) | 0.30 |
| P3 | Position | 0.25 |
| P4 | Monogram type | 0.20 |
| P5 | Dating period | 0.10 |

- Weights were tuned by a two-stage grid search on the calibration set only (80% of the corpus), maximizing Top-3 accuracy. The search is approximate and does not guarantee a global optimum.
- Missing context (unknown position, type or dating) is handled by redistributing weights or falling back on raw frequency.
- A combinatorial filter eliminates letter pairs never attested together in the corpus and penalizes rare pairs.
- Each prediction can be decomposed by source of evidence, which makes the model explainable.

## Evaluation

- Stratified split by monogram type: 80% calibration / 20% test.
- One or two letters masked per test monogram: 471 cases (1 letter) and 1390 cases (2 letters), reported separately.
- Baseline: letters ranked by frequency conditioned on monogram type.

| Metric | 1 masked: baseline | 1 masked: full engine | 2 masked: baseline | 2 masked: full engine |
|---|---|---|---|---|
| Top-1 accuracy | 0.454 | 0.563 | 0.219 | 0.381 |
| Top-3 accuracy | 0.679 | 0.773 | 0.556 | 0.735 |
| MRR | 0.589 | 0.695 | 0.429 | 0.583 |

Over the full test set (1861 cases): Top-3 = 74.5% and MRR = 0.611 for the full engine, against Top-3 = 58.7% and MRR = 0.469 for the baseline. Top-5 = 86.8% (baseline 74.8%). Calibration gap = 0.037 (baseline 0.001).

By monogram type (full engine): block (n = 760) Top-1 47.6%, Top-3 79.9%, MRR 0.650; cruciform (n = 1101) Top-1 39.2%, Top-3 70.8%, MRR 0.584.

## Limitations

- Performance varies strongly by letter. Ten of the 24 letters (Beta, Chi, Delta, Eta, Gamma, Iota, Kappa, Lambda, Theta, Xi) get no correct first-position prediction, while Upsilon (90.7%), Nu (82.6%) and Omicron (68.3%) are well predicted.
- For Upsilon, Omicron and Rho, the composite score does slightly worse than frequency alone.
- For several poorly predicted letters (Delta, Lambda, Kappa, Gamma), the calibration gap is negative: the engine is on average more confident when wrong than when right. The confidence score should not be trusted for these letters.
- The corpus is small (350 monograms) and the results have not been validated by sigillographers on new cases.
- The engine is a decision-support tool, not an authority for seal identification.

## Perspectives

Coupling the engine with a retrieval-augmented pipeline over a prosopographic knowledge base to propose attested names; an interface for sigillographers with per-component score breakdown and explicit warnings; adaptive weighting per candidate letter; a larger annotated corpus with expert validation.

## My contribution

- Collected the data and created the correction tool.

## Data availability

The dataset is not publicly released. This repository documents the method, the evaluation protocol and the results reported in the paper.

## Citation

```bibtex
@inproceedings{canonge2026monograms,
  author    = {Canonge, Yeleen and Bouhaid, Soukaina and Mahanta, Saranga Kingkor and Eyharabide, Victoria},
  title     = {Recovering Missing Letters from Byzantine Monograms with Explainable Bayesian Inference},
  booktitle = {The 8th workshop on analySis, Understanding and proMotion of heritAge Contents (SUMAC '26)},
  year      = {2026},
  address   = {Rio de Janeiro, Brazil},
  publisher = {ACM},
  doi       = {10.1145/3840472.3841738}
}
```
