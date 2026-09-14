# Case study: one complete run in machine unlearning

Real parameters and artefacts from a full run of this skill. Scale the
workload proportionally when moving to another domain.

## Domain setup

- **Benchmarks**: TOFU (fictional-author question answering), MUSE
  (news and books, copyright and privacy), WMDP (bio and cyber
  hazardous knowledge).
- **Additional caliber**: OpenUnlearning, a unified framework and
  evaluation caliber on a Llama-3.2-1B base.
- **Anchor values**: TOFU-5% SimNPO 0.99 / 0.58; TOFU-10% SimNPO 0.45
  / 0.62; MUSE-News SimNPO 47.09 / 40.31 / 12.90 / 11.90; MUSE-Books
  SimNPO 0.00 / 48.27 / 0.00 / -19.82; WMDP RMU (Cut) 29.3 / 57.0 /
  24.9.

## Workload and output, with real numbers

| Stage | Approach | Output |
|---|---|---|
| S0 scoping | 3 benchmarks plus the OpenUnlearning caliber; anchors and holders fixed | caliber section of the constraint document |
| S1 search | 4 parallel sub-agents (lower-tier workshops / IJCAI, KDD, CVPR / two-week increments plus venue upgrades / pending verification) | 90 papers listed, plus 20 incremental methods and 10 preprints |
| S1-E full-text scan | SimNPO as the keyword across the whole PDF archive | 11 papers with direct numeric comparisons (7 method papers, 4 evaluation papers) |
| S2 archiving | batch download plus three-gate validation | 24 PDFs, all passing |
| S3 extraction | 4 parallel sub-agents at 3 to 4 papers each | complete numbers plus baseline reconciliation for 11 method papers |
| S3.5 notes | 6 parallel sub-agents at 4 papers each | 24 deep-read notes, 9 to 15 KB each |
| S4 tables | Markdown constraints plus a workbook with 22 sheets | A sheets plus 19 rows, B sheets plus 2 rows, marker held |
| S5 baseline | three-condition filter | TOFU to NPO; MUSE to TPO_GDR; WMDP to RMU (Cut) |
| S6 verification | PyMuPDF re-check plus column check | 22 of 22 numbers hit, zero column anomalies |

## What caliber isolation actually caught

This is the part that justifies the pipeline.

- **BLADE**: a custom harmonic-mean caliber whose reproduced SimNPO
  moves in the opposite direction to the original (probability 0.848
  against the official forget quality 0.99). Isolated row.
- **BalDRO**: the TOFU main table uses forget 1 percent while this
  table maintains only 5 and 10 percent, the MUSE numbers are
  self-reproduced on a 0 to 1 decimal scale, and the SimNPO baseline
  does not reconcile. Isolated row.
- **ALTER**: Zephyr base model with the standard bio, MMLU and cyber
  metrics, and RMU compared directly in the same table (bio 24.4
  against 30.2, cyber 24.0 against 27.3, but MMLU 56.4 against 57.0).
  The only candidate that could enter as a consistent caliber, and not
  a strict dominance, so the marker did not move.
- **SPUL and GRACE**: fully non-standard, self-built datasets and
  metrics. Not loaded into the table at all, recorded in the risk list
  only.

## Baseline reasoning from the same run

- SimNPO reproduction failed locally, so it was excluded as a
  baseline.
- TOFU went to **NPO**: formally published at COLM 2024, implemented
  twice inside OpenUnlearning, and a loss-level method that an
  evolutionary modification can be attached to.
- MUSE went to **TPO_GDR**: AAAI 2026 oral with code, the same target
  caliber as the official numbers, and first place on the aggregate
  ranking.
- WMDP kept **RMU (Cut)**: it holds the marker and reproduced cleanly
  on the local machine.
- **GROM** was explicitly excluded: closed-form and training-free, so
  there is no loss function to evolve, and it shares an author with
  prior work, which also affects how related work must be cut.

## Reusable judgement calls

1. Compare baselines before comparing numbers. A baseline that does
   not reconcile demotes the whole paper.
2. A better number under an incommensurable caliber does not earn a
   column; it earns a note.
3. The marker moves only when all three conditions hold. Otherwise
   hold the marker and file the challenger as a candidate.
4. Select a baseline by asking whether your contribution can be
   attached to it, not by asking who scores highest.
