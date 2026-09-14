# Staged prompt templates (P0 to P8)

Copy each block into a sub-agent call, replacing the bracketed slots
with your own domain. The numbered prompt IDs are stable; they map to
the stages in SKILL.md.

## Contents

- [How to use these templates](#how-to-use-these-templates)
- [P0 Domain and benchmark scoping](#p0-domain-and-benchmark-scoping)
- [P1 Full-corpus method search](#p1-full-corpus-method-search)
- [P2 Acquisition and three-gate validation](#p2-acquisition-and-three-gate-validation)
- [P3 Value extraction and baseline reconciliation](#p3-value-extraction-and-baseline-reconciliation)
- [P4 Table construction](#p4-table-construction)
- [P5 Baseline selection](#p5-baseline-selection)
- [P7 Method-centric full-text scan](#p7-method-centric-full-text-scan)
- [P8 Deep-read notes](#p8-deep-read-notes)
- [P6 Independent verification](#p6-independent-verification)

## How to use these templates

Each block is a self-contained brief for one sub-agent. Issue several
agent calls inside a single message to run them concurrently. Every
template obeys one rule: **not mentioned means not mentioned, not
found means left empty, fabrication is forbidden.**

## P0 Domain and benchmark scoping

```
You are a literature-scoping sub-agent. Task: define the authoritative
evaluation benchmarks and the current best method for [domain].

1. List the N benchmarks of the domain that top-venue papers actually
   use. For each one give:
   - official paper and link, publication venue
   - main caliber: standard base model, data split, evaluation script
     (quoted verbatim from the official description)
   - metric definitions, including the mathematical meaning of each
     metric and its direction (higher is better, lower is better,
     closer to zero is better)
2. For each benchmark give the anchor value: the public-baseline
   number reported by the official paper or the most authoritative
   reproduction. This becomes the ruler for later reconciliation.
3. Identify the current best method per benchmark and state the rule
   you applied to decide.
4. List the known caliber traps of the domain: which metric and base
   model combinations cannot be compared.

Iron rules: verify every arXiv ID on its abstract page; take the venue
from the official source; mark self-claimed acceptance as
unverified.
Output: Markdown tables plus a caliber-trap list.
```

## P1 Full-corpus method search

```
You are a retrieval sub-agent, route [N], search axis: [formally
published venues / preprints and recent increments / framework and
benchmark reverse lookup / workshops and journals].

Task: find every paper in [domain] whose own experiments use at least
one of [benchmark 1, benchmark 2, benchmark 3].

Known list (do not repeat, report increments only): [paste the
existing method list]

Iron rules:
1. Open every arXiv abstract page and confirm the ID exists with a
   matching title. Never rely on memory.
2. Take the venue from an official source (OpenReview, ACL Anthology,
   DBLP, proceedings). Mark self-claimed acceptance as unverified.
3. The paper must use the benchmark in its own experiments. Citing it
   is not enough.
4. Leave missing fields as a dash. Do not invent.
5. If nothing is found, say so explicitly and name the channels you
   searched.

Output: | method | full title | arXiv ID (link) | venue and year
(official link) | benchmark used | code link | notes |
```

Extra duty for the preprint axis: also check whether a preprint
already in the list has since been accepted. Only official-page
evidence counts as an upgrade.

## P2 Acquisition and three-gate validation

```
Download the following papers as PDFs into [directory], named
<index> <title>.pdf.

Every file must pass three gates, run in one script:
1. Magic number: data[:5] == b'%PDF-'
2. Size: len(data) > 30_000
3. Content: extract first-page text with PyMuPDF and require the two
   preset title keywords per paper.

Failure handling:
- arXiv direct /pdf/<id> works; the API rate-limits quickly.
- OpenReview often triggers a Cloudflare challenge (403 on both the
  direct link and the API). Find the same-title arXiv version and
  cross-confirm "same title, same venue" against the bibtex in the
  authors' repository.
- IEEE, ACM and Springer paywalls: mark as not acquired, permission
  required. Never fabricate.

Output: success and failure lists with reasons.
```

## P3 Value extraction and baseline reconciliation

```
You are a numeric-extraction sub-agent. Extract experimental numbers
precisely from these PDFs. Use PyMuPDF through a shell call; the Read
tool cannot open binary PDFs.
[absolute PDF paths]

For each paper, output four parts:

A. Experimental setup
- dataset and split; base model quoted verbatim from the source;
  whether the metric definition matches the standard; stopping
  criterion and checkpoint selection; full fine-tuning or LoRA

B. Main numbers
- copy the method rows verbatim, preserving original precision and
  scientific notation, each annotated as "Table X (p.Y)"

C. Baseline reconciliation (mandatory)
- the public baselines this paper reproduces: [list two to four
  public baselines for the domain], with their numbers from the same
  table
- compare item by item against the anchor values below: [paste the S0
  anchor table]
- conclude one of: consistent / half-consistent (name the divergence)
  / inconsistent (isolate)

D. Hyperparameters and code
- batch size, learning rate, epochs or steps, GPU model and count,
  repository link

Iron rules: every number must be findable in the source tables;
unreported means "not reported"; rounding or unit conversion (for
example 0 to 1 decimals against percentages) is forbidden unless
explicitly flagged.
```

## P4 Table construction

```
Build the sheets with the column names and order given in
table-schema.md:
- one A sheet (literature numbers) and one B sheet (own experiments)
  per benchmark
- A sheet rows are methods from the papers; B sheet first row is the
  best value per metric, then the comparison baseline rows, then
  placeholders for your own runs
- non-standard metrics always go into the setup-consistency note.
  Never add a column.
- every new row must fill in: data source (Table X p.Y) and setup
  consistency (consistent / half-consistent with reason /
  isolated with reason)
```

## P5 Baseline selection

```
Select my reproduction baseline from the A sheets. All three
conditions must hold:
1. Caliber match: passes the S0 anchor reconciliation.
2. Reproducible: official code or a framework implementation exists,
   ideally already running locally.
3. Modifiable: my contribution ([describe the contribution]) can be
   attached to it.
   Counter-example: a project about evolving a loss function cannot
   start from a closed-form or training-free method, because there is
   no loss to evolve.

For each candidate give: numbers, hyperparameters, reproducibility
evidence, modifiability verdict.
For each exclusion give the reason: prior work, no code,
incommensurable caliber, or not modifiable.
```

## P7 Method-centric full-text scan

```
Full-text scan every PDF under [archive directory] for the keyword
[core method name]. List every paper where the term appears, with its
context and any tabulated numbers.

Purpose: find every paper that compares numerically against the
method. This reliably surfaces papers that keyword search misses.

Classify the output:
1. Method papers (direct competitors): have a comparable table; give
   the delta against the core method in the same table.
2. Evaluation and attack papers: treat the method as the target; list
   the dimensions where it is shown to be weak.
3. Citation-only papers: exclude.

Iron rule: every better or worse verdict must be supported by a
number that can be grepped in the source PDF. A verdict without a
concrete number is void.
```

Verification reminder: scan reports are the place where venue
fabrication and unsourced numbers appear most often. Convert each
assertion into a greppable string, re-check it, and mark misses as
unconfirmed with a request for evidence or deletion.

## P8 Deep-read notes

```
Produce one Markdown deep-read note per paper:
[PDF path, output note path, named <index> <method> <title>.md]

Read the PDF with PyMuPDF when the Read tool cannot open it.

Note skeleton:
0. Domain pre-check: confirm the paper belongs to [domain]; if not,
   report and stop.
1. Core positioning: paper record (title in original language and in
   English, authors and affiliation, venue, open source or not); the
   core mechanism in one sentence of at most 50 words, naming the
   technical school; inputs, outputs, and the paradigm boundary.
2. Mechanism and architecture: top-level logic (which trade-off is
   being resolved), then components decomposed from Figure 1, then
   the full pipeline, then the low-level operation (what is actually
   done to the weights, the loss or the data).
3. Formula reading: for each key formula give (a) the formula,
   (b) per-variable meaning, (c) a plain-language reading of why this
   operation produces the intended effect.
4. Evaluation and critique: effectiveness metrics, utility metrics,
   efficiency evidence; reviewer attack surface (hidden strong
   assumptions, artefact risks, whether ablations are sufficient,
   unaddressed confounders).
5. Limitations, author-stated versus reviewer view; cross-domain
   inspiration; two questions that can only be answered by returning
   to the source.

Iron rules: be extremely concise and never translate the source;
annotate key claims with section and page; anything absent from the
paper is written as "not mentioned"; give English terms for core
terminology. For attack and evaluation papers, replace the
core-mechanism section with an audit or attack pipeline. Each note is
at least 3000 characters; check the file for completeness after
writing.
```

## P6 Independent verification

```
Use PyMuPDF to check every number already in the table against the
full PDF text by string match:
- input: (PDF path, [(description, list of number strings that must
  all hit)])
- output: hit / miss (every miss requires manual review)

Run two further checks:
1. Structure check: scan all table blocks; blocks with an inconsistent
   column count must be zero.
2. Link spot check: batch-probe newly added official links.

Do not submit until everything hits.
```
