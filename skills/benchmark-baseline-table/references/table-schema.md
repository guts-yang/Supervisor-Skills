# Comparison table schema

Domain-independent column templates and maintenance rules. Bracketed
slots are replaced per domain.

## Contents

- [1. Two-file convention](#1-two-file-convention)
- [2. A sheets: literature numbers](#2-a-sheets-literature-numbers)
- [3. B sheets: own experiments](#3-b-sheets-own-experiments)
- [4. Promotion conditions for the best-method marker](#4-promotion-conditions-for-the-best-method-marker)
- [5. Maintenance iron rules](#5-maintenance-iron-rules)
- [6. Spreadsheet operation notes](#6-spreadsheet-operation-notes)

## 1. Two-file convention

| File | Role | Holds | Does not hold |
|---|---|---|---|
| `<name>_table.md` | Constraint and rule source | caliber definitions, benchmark descriptions, grading rules, marker promotion conditions, risk list, change log | no data rows (only a pointer once data moves out) |
| `<name>_table.xlsx` | Data source | all A sheet and B sheet values | rules and logs |

Order of change: editing data means editing the workbook; editing
rules means editing the Markdown file. The two files never both hold
the same content, which is what keeps them from drifting apart.

## 2. A sheets: literature numbers

One A sheet per benchmark. The column count is fixed; new methods
never add a column.

| Base model | Best | Code | Method | [metric 1] | [metric 2] | ... | [metric n] | Full name | Year and venue | Data source | Setup consistency |
|---|---|---|---|---|---|---|---|---|---|---|---|

### Column semantics

- **Base model**: the model the paper actually used, quoted verbatim,
  including whether it is the chat or instruct variant and whether it
  was fine-tuned further. This is the primary caliber witness.
- **Best**: only the benchmark's current holder carries the marker.
  When the aggregation scheme differs, write `marker (scheme name)` so
  the isolation is visible.
- **Code**: the official repository, or `not released / to verify`.
- **Method**: the method's short name; multiple variants of one paper
  occupy separate rows.
- **Metric columns**: fixed order per benchmark. Leave a dash when the
  paper does not report the metric.
- **Full name**: the method's full English name.
- **Year and venue**: official page first; mark preprints with a
  `preprint` tag.
- **Data source**: where the number actually came from, precise to
  table and page, for example `Original Table 1 (p.6)` or
  `third-party reproduction (XX, Table 2)`.
- **Setup consistency**: one of three values plus a reason.
  - `consistent`: base model, split, metric and stopping criterion all
    match.
  - `half-consistent`: some match, but a reproduced baseline or the
    metric reading does not. The note must say **do not compare
    directly**.
  - `isolated`: the base model or the metric definition differs. The
    note says the row is an isolated reference, and any non-standard
    metric value lives in the note rather than in a column.

### Row types

1. **Baseline rows**: Original, Retrain, Target and similar. They do
   not participate in ranking.
2. **Main rows**: consistent caliber. They participate in the marker
   and the ranking.
3. **Isolated rows**: inconsistent caliber. Qualitative reference
   only, with values in the note field.

## 3. B sheets: own experiments

One B sheet per benchmark.

| Base model | Best | Code | Method | [metric 1] | ... | [metric n] | Full name | Year and venue | Batch | LR | Epoch | GPU model x count |
|---|---|---|---|---|---|---|---|---|---|---|---|---|

### Row order, fixed

1. **Best-value row**: the per-metric best from the A sheet,
   emphasised. This is the target line.
2. **Comparison baseline rows**: the methods selected in S5, with the
   best column marking them as `baseline`. Every hyperparameter must
   be filled; where it cannot be found, leave it empty and say so.
3. **Own experiment rows**: placeholders such as `MY-OURS-run1`,
   filled after the runs.

A B sheet holds only the methods you will run or compare against. The
full literature list belongs to the A sheet.

## 4. Promotion conditions for the best-method marker

All three must hold before a marker moves.

1. The method uses **exactly** the main caliber of the sheet and
   passes the anchor reconciliation.
2. It **strictly dominates** on the main metrics: no main metric may
   get worse.
3. It is formally published, or it is a preprint with **reproducible
   code plus third-party reproduction evidence**.

On promotion, the old row's marker becomes a dash and the note records
`marker moved to XX on YYYY-MM-DD`. If the conditions are not met,
keep the marker where it is and file the challenger in the
candidate and risk list.

## 5. Maintenance iron rules

Copy this list into the constraint document.

1. **Decide the caliber before inserting a row.** Consistent goes to
   the main region; anything inconsistent becomes an isolated row with
   the divergence written in the setup-consistency column.
2. **Source priority when filling a cell**: original paper's main
   table, then original appendix, then a credible third party with the
   same setting (name the reproducing party), then a preprint's own
   number tagged as preprint.
3. **Forbidden sources**: blog restatements, leaderboard screenshots,
   numbers with no provenance.
4. **Fill column by column**: hyperparameters come from the
   implementation details section or the appendix; when absent, leave
   the cell empty.
5. **Refresh the date and the change log** whenever a row is inserted
   or edited.
6. **Compare baselines before comparing numbers.** The same baseline
   disagreeing across two tables is hard evidence of a protocol
   mismatch.
7. **The aggregation scheme is part of the conclusion.** If a caliber
   uses a harmonic mean or similar aggregate, a different aggregate
   can invert the whole ranking; any citation must state the scheme.
8. **Scope of maintenance**: only rows of the form method by benchmark
   are maintained. Tables whose rows are the metrics themselves, such
   as metric meta-evaluations, are not loaded into the workbook; their
   conclusions degrade to a note.

## 6. Spreadsheet operation notes

This section describes the maintenance channel used on an existing
workbook. Creating a workbook from scratch uses the generation path
instead.

1. Present the file, then read the real file identifier. The path and
   the value returned by presentation are both insufficient.
2. Probe the real boundary of each sheet before writing. A structural
   addition must insert a dimension first; writing directly over an
   occupied cell destroys the user's data.
3. Every written item needs a `value_type` (string, number, boolean,
   formula) plus the matching typed field. Sending only a bare value
   raises `key 'value_type' not found`.
4. Disable text wrapping across the data sheets, with the range taken
   from each sheet's real used range. Start the verification scan
   below the header row, because headers usually keep wrapping enabled
   for multi-line labels.
5. Save the file explicitly. Do not close the file proactively; that
   closes the view the user is reading.
