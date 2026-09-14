---
name: benchmark-baseline-table
description: >-
  Turns a domain survey into a maintainable SOTA and baseline
  comparison table: scoping the authoritative benchmarks, locking
  anchor values, sweeping the full method corpus, extracting numbers
  from source PDFs with baseline reconciliation, building literature
  and experiment sheets, and selecting a reproducible baseline. Use
  when the user wants to build or maintain a SOTA table, find
  reproducible baselines in a field, or learn which methods were
  evaluated on a benchmark and with what numbers.
license: CC-BY-4.0
---

# Benchmark Baseline Table

## Overview

A baseline table decides what you reproduce and what you compare
against. It is not a literature review and not a screenshot of a
leaderboard: it is a structured, long-lived data table in which every
number is traceable to a page and every row carries an explicit
verdict about whether its measurement caliber matches the main table.

This skill runs the full pipeline: scope a domain into its
authoritative benchmarks, lock anchor values, sweep the method corpus
in complementary directions, acquire and validate source PDFs, extract
numbers with a mandatory baseline reconciliation, build the two-sheet
table structure, select a reproducible baseline, then verify every
number that entered the table.

The deliverable is a pair of files: a Markdown constraint document
that holds calibers, rules and the change log, and an Excel workbook
that holds the numbers.

## When to use this skill

- "Build a SOTA/baseline table for domain X", "set up a folder to
  maintain the baseline table for X long term".
- "Which baselines in X are reproducible right now?"
- "Which methods have been run on benchmark Y, and what numbers did
  they report?"
- A literature sweep is finished and the result must become structured
  data rather than prose.
- A reproducibility baseline must be frozen before the author's own
  experiments start.

## When NOT to use this skill

- Writing a survey or a related-work section. Use `paper-writer`, or
  `deep-research` for the literature map.
- Deep-reading a single paper for its mechanism. Reading one paper
  does not need a table.
- Reproducing or re-implementing a method. This skill stops at
  selecting and freezing the baseline.
- Designing a new benchmark or evaluation suite. Use
  `benchmark-paper-template`.

## Hard rules (read first)

These five rules determine the quality of everything downstream. Read
them before starting any stage.

1. **Caliber before numbers.** A method name can mean different
   implementations, different base models and different metric
   definitions across papers. If a paper's reproduction of a public
   baseline (NPO, RMU, and similar) does not match the authoritative
   reference value, demote every number in that paper to an
   **isolated row** and forbid direct comparison with the main table.
2. **One writer per file.** Constraints and data live in separate
   files. The constraint document (`.md`) holds calibers, rules and
   the change log; the data workbook (`.xlsx`) holds numbers only.
   Editing data means editing the workbook; editing rules means
   editing the Markdown file.
3. **Missing stays missing.** Write `not reported` rather than an
   approximate value. Never fill a cell from a blog post, a
   leaderboard screenshot, or a remembered number.
4. **Every row is traceable.** Each row must be able to answer "which
   table on which page did this number come from". Record the source
   as `Original Table 1 (p.6)` or `third-party reproduction (XX,
   Table 2)`.
5. **Domain rules outrank generic rules.** A domain may add stricter
   rules (for example "every sheet must contain a Vanilla baseline
   row", or "a row enters the table only after passing a Stage-1
   memorization gate"). List domain-specific rules in their own block
   in the constraint document and state their precedence explicitly.

## Stage map

Stage IDs are deliberately stable across versions, including the
`S-1` label that runs before `S0`, so that existing maintenance logs
and change entries stay valid. Read the *Order* column, not the label,
when you sequence the work.

| Order | Stage | Goal | Who runs it | Typical fan-out |
|---|---|---|---|---|
| 1 | `S-1` | Lock folder, delivery depth, benchmark scope | Main agent, with the user | ask, do not guess |
| 2 | `S0` | Define benchmarks, anchor values, current best | Main agent, optionally 1 sub-agent | 1 |
| 3 | `S1` | Sweep the full method corpus, deduplicate | Sub-agents, complementary directions | 3 to 5 parallel |
| 4 | `S2` | Acquire PDFs and pass three validation gates | Main agent plus a script | scripted |
| 5 | `S3` | Extract numbers and reconcile baselines | Sub-agents | 3 to 4 papers each |
| 6 | `S3.5` | Produce structured deep-read notes | Sub-agents | 3 to 4 papers each |
| 7 | `S4` | Build the A sheets and B sheets | Main agent only | 1 |
| 8 | `S5` | Freeze the reproduction baseline | Main agent | 1 |
| 9 | `S6` | Verify numbers, structure and links | Fresh round, main agent plus a script | scripted |

## Core procedure

### S-1 Preflight: lock three parameters

When the user says "create a folder to maintain the baseline table for
domain X", ask for these three parameters before writing anything.
Skipping them means S0 to S6 are rerun from scratch.

1. **Folder location and name.** Inside the current workspace next to
   the report, or beside an existing table for the same domain?
2. **Delivery depth for this round.** (a) skeleton only: constraint
   document plus sheet structure, data to be filled later; (b)
   skeleton plus a first data batch; (c) the complete S0 to S6 run.
   Option (c) costs hours of parallel retrieval. Unless the user
   explicitly asks for the full run, default to (a) and stand the
   container up first.
3. **Main caliber scope.** A domain usually has several benchmark
   families. Let the user multi-select. Different families get their
   own sheets and are never ranked against each other.

The skeleton delivery is complete only when all of the following
exist:

| Artefact | Role |
|---|---|
| `README.md` | Directory map, two-file convention, common operations, stage progress table |
| `<name>_table.md` | Rule source: calibers, anchors, best-method rule, pitfall list, field spec, iron rules, change log |
| `<name>_table.xlsx` | Data source: one index sheet plus, per benchmark, one A sheet and one B sheet |
| `scripts/` | S2 three-gate validation, S6 numeric re-check, S6 column-count check (all three can be in place before data exists) |
| Empty subfolders | `papers/`, `notes/`, `intermediate/`, each with a `.gitkeep` |

**Workbook creation channel.** Creating an `.xlsx` from scratch is an
Office file-creation task and must go through the workbook-generation
path (openpyxl direct write), not through the editor SDK. The
`sheet_set_range_value` and `sheet_insert_dimension` calls documented
in references/table-schema.md belong to the **maintenance** channel
used to insert rows into an existing workbook. Do not mix the two.

See: references/staged-prompts.md for the P0 prompt.

### S0 Domain and benchmark scoping

This stage sets the workload for everything that follows. Produce
three things:

- The **authoritative benchmarks** of the domain, each with its main
  caliber: base model, data split, metric definitions, value
  direction.
- The **anchor value** of each benchmark: the public-baseline number
  reported by the official paper or by the most authoritative
  reproduction. This is the ruler every later paper is measured
  against.
- The **current holder**, that is the best method per benchmark under
  the main caliber.

See: references/staged-prompts.md for the P0 prompt.

### S1 Full-corpus method search

Dispatch three to five sub-agents along **complementary** axes, never
along the same keyword repeated:

- Axis A: formally published work, scanned venue by venue.
- Axis B: preprints, recent increments, and venue-upgrade checks for
  preprints already in the list.
- Axis C: framework and benchmark reverse lookup (who cites the
  benchmark) plus survey cross-referencing.
- Axis D: lower-tier venues, workshops and journals, where recall is
  usually worst.
- Axis E, optional but high value: take one core method name as a
  keyword and full-text scan the local PDF archive to find every paper
  that compares against it numerically. This produces a **competitor
  profile** and routinely surfaces papers that keyword search misses.

**Deduplicate** against the existing list with fuzzy title matching.
Locally archived PDFs often carry truncated filenames, so exact
matching will produce false "new" hits.

See: references/staged-prompts.md for the P1 and P7 prompts.

### S2 Acquisition and three-gate validation

Download PDFs into one directory, named `<index> <title>` or
`<method> <title>`. Every file passes three gates:

1. Magic number `%PDF-`.
2. File size within a plausible range (above 30 KB).
3. First-page text extracted with PyMuPDF contains the expected title
   keywords.

For paywalled work with no arXiv version: look for the same-title
arXiv version first and cross-confirm it with the bibtex in the
authors' official repository; only then fall back to marking the item
as `not acquired`.

See: references/staged-prompts.md for the P2 prompt.

### S3 Value extraction and baseline reconciliation

The most expensive stage. Always parallelise. Each sub-agent owns
three to four papers and emits a fixed four-part record:

- **A. Experimental setup**: dataset and split, base model quoted
  verbatim, metric definition and whether it matches the standard,
  stopping criterion, full fine-tuning or LoRA.
- **B. Main numbers**: copied verbatim, original precision preserved,
  each row annotated with `Table X (p.Y)`.
- **C. Baseline reconciliation** (mandatory): the public baselines the
  paper reproduces, compared item by item against the S0 anchor
  values. Conclude `consistent`, `half-consistent` with the exact
  divergence named, or `inconsistent` and isolate the row.
- **D. Hyperparameters and code**: batch size, learning rate, epochs
  or steps, GPU model and count, repository link.

See: references/staged-prompts.md for the P3 prompt.

### S3.5 Deep-read notes

Optional but strongly recommended. Before numbers enter the table,
produce one structured note per paper. Two benefits: each number is
understood independently, which lowers transcription error, and the
notes are directly reusable when writing related work.

- **Parallelise**: three to four papers per sub-agent, several agent
  calls in one message.
- **Naming**: `<index> <method> <title>.md`, where the index matches
  the PDF archive index exactly.
- **Heading sync**: the first line of the note must equal the
  filename stem. Renaming a file without renaming its H1 breaks the
  directory note-to-file mapping.
- **Note skeleton**, adapted per domain: domain pre-check, core
  positioning and mechanism, mechanism diagram and full pipeline,
  formulas with per-variable meaning and a plain-language physical
  reading, evaluation system and reviewer attack surface,
  limitations, cross-domain inspiration, and two questions that can
  only be answered by going back to the source.

Attack and evaluation papers need their core-mechanism section
rewritten as an audit or attack pipeline. Do not force the method-paper
template onto them.

See: references/staged-prompts.md for the P8 prompt.

### S4 Table construction

Build two sheet families, following references/table-schema.md:

- **A sheets, literature numbers**: one per benchmark. Rows are
  methods; columns are base model, best-method marker, code link,
  method, one column per metric, full name, venue and year, data
  source, and setup consistency.
- **B sheets, own experiments**: one per benchmark. The first row
  aggregates the best value per metric from the A sheet, the rows
  after it are the comparison baselines from S5 with their
  hyperparameters, and the last rows are placeholders for the
  author's own runs.

**Never add a column name.** Non-standard metrics reported by a new
method go into the setup-consistency or notes column. A domain in
which every benchmark has a genuinely different headline metric is the
one sanctioned exception: add a single dedicated-metric column with a
constrained format (`name=value`, semicolon separated) and declare the
deviation explicitly in the constraint document.

See: references/staged-prompts.md for the P4 prompt.

### S5 Baseline selection

Filter candidates on three conditions, all of which must hold:

1. **Caliber match**: passes the S0 anchor reconciliation.
2. **Reproducible**: official code or a framework implementation
   exists, ideally already running on the local machine.
3. **Modifiable**: the author's contribution can be attached to it.
   A project about evolving a loss function cannot start from a
   closed-form or training-free method, because there is no loss to
   evolve.

Record the numbers, hyperparameters, reproducibility evidence and
modifiability verdict for every candidate. Every exclusion needs a
stated reason: prior work, no code, incommensurable caliber, or not
modifiable.

See: references/staged-prompts.md for the P5 prompt.

### S6 Verification

Not optional, and it must run as a fresh round that does not reuse the
S3 extraction results.

- **Numeric re-check**: for every key number already in the table, grep
  the full PDF text through PyMuPDF. Emit a hit and miss list. A miss
  means manual review, not silent acceptance.
- **Structural check**: scan every table block and require zero blocks
  with an inconsistent column count.
- **Link spot check**: batch-probe newly added official links.

See: references/staged-prompts.md for the P6 prompt.

## Parallelisation budget

- **Foreground, not background.** Background sub-agents are not
  recoverable after a session interruption; a long retrieval fan-out
  must be issued as several agent calls inside one message.
- **One axis per agent.** Three to five agents on S1, three to four
  papers per agent on S3 and S3.5.
- **Main agent owns S4, S5 and the venue and numeric verdicts.** Sub
  agents may retrieve and transcribe; they may not be trusted for
  "is this method better than the state of the art" style judgements.

## Pitfalls

- **Sub-agent summaries are not evidence.** A summarised verdict is not
  a verified fact; one measured run reported a preprint as an EMNLP
  2026 paper when it was never accepted anywhere. Every venue and
  numeric assertion must be re-checked by the main agent against the
  original PDF or the official page.
- **Background sub-agents are unrecoverable.** After an interruption
  their output is gone. Use foreground fan-out for long retrieval.
- **The Read tool cannot open binary PDFs.** Extract with PyMuPDF
  through a shell call.
- **The best value in a row is not automatically the state of the
  art.** A per-row best may come from different methods (most
  thoroughly forgotten versus highest utility). Promote a marker only
  when all three conditions hold: identical caliber, strict dominance
  on every main metric, and reproducibility. Otherwise keep the marker
  where it is and record the challenger as a candidate.
- **Writing a value through the editor SDK requires a type.** Each
  item needs `value_type` plus the matching typed field; a bare
  `value` key raises `key 'value_type' not found`.
- **New rows require an insert, not a write.** Use the dimension
  insert call to make room; writing directly overwrites user data.
- **Turn wrapping off on data sheets**, scoped to each sheet's real
  used range.
- **Second-hand conclusions need the same treatment as sub-agent
  output.** Turn each assertion into a string that can be grepped in
  the PDF full text and run the check. Below a 100 percent hit rate,
  the assertion is not adopted.
- **A renamed note file must have its H1 renamed too**, or the note
  and the directory disagree.
- **An empty anchor or a deliberately absent marker is a valid
  conclusion, not unfinished work.** When a domain has no shared
  public baseline, split the anchor table into a verified block and a
  to-be-filled block and declare the gap as a domain fact. When
  cross-comparison is impossible, ship version 1 with no global marker
  at all, writing only the promotion rule. Inventing a marker or
  filling a plausible value is worse than leaving the cell empty.
- **Column-count checks and header rows interact badly.** When the
  lint rule is "no text wrapping in the data region", start the scan
  below the header row. Headers often keep wrapping enabled so that
  multi-line labels render, and a naive scan reports every sheet as a
  violation.
- **Respect the stable stage IDs.** Do not renumber stages that
  existing maintenance logs and change entries already reference.

## Integrity gate

Five bullets, all **[inspection]** class: the agent verifies each
directly from the artefacts it produced.

Before reporting the delivery as complete:

1. **[inspection]** Every benchmark has a stated anchor value or an
   explicit declaration that the domain lacks one.
2. **[inspection]** Every newly added method was checked against all
   rows already in the table, including earlier batches.
3. **[inspection]** Every isolated row states its divergence in the
   setup-consistency column and carries a do-not-compare warning.
4. **[inspection]** Numeric re-check hit rate on table-bound numbers
   is 100 percent, or every miss is listed for the user.
5. **[inspection]** Column-count anomalies equal zero, and the
   workbook is saved with wrapping disabled on data sheets.

For the skeleton delivery, the minimum bar is: per-sheet header column
counts match the constraint document, data-region column anomalies
equal zero, no unexpected wrapping in the data region, and every
benchmark has a Vanilla baseline row.

## Output format

**Skeleton delivery**: the five artefacts listed in the S-1 table,
plus a short handover note naming what is still empty on purpose.

**Full run**: the updated workbook, the updated constraint document
with a change-log entry, a numeric re-check report, a structure-check
report, and the candidate list for any marker that was challenged but
not promoted.

## References

- [`references/staged-prompts.md`](references/staged-prompts.md):
  copy-paste prompt templates P0 to P8, one per stage.
- [`references/table-schema.md`](references/table-schema.md):
  A and B sheet column templates, the three promotion conditions,
  eight maintenance iron rules, and spreadsheet operation notes.
- [`references/case-study.md`](references/case-study.md): one complete
  worked run in machine unlearning, with real workloads, isolation
  outcomes and baseline reasoning.

## Related skills

- `deep-research`: when the domain map itself is missing and the
  survey has to come before any table.
- `benchmark-paper-template`: when the goal is to design a benchmark
  rather than to compare against existing ones.
- `pre-submission-reviewer`: when the table has to be defended in a
  submitted manuscript.
- If a dedicated paper-inventory or venue-grading skill is installed
  in the environment, hand bulk inventory and venue classification to
  it rather than reimplementing the search.
