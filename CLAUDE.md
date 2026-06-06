# Literature Review Project

## Goal
Systematic literature review. For each paper: extract structured summaries, track progress, then collaboratively synthesize findings into ideas and proposals.

## Folder Structure
```
raw/                    ← Drop PDFs here for processing
summary/                ← Auto-generated summaries (one .md per paper)
review_literature/      ← Synthesis workspace; one subfolder per idea
  <slug>/
    proposed.md         ← Idea scaffold: core idea, relevant papers, open questions
    section.md          ← Fleshed-out literature review prose
    archive/            ← Dated snapshots of section.md before each revision
      section-YYYY-MM-DD.md
original_idea/          ← The user's own research paper and its evolving sections
  first_draft.pdf       ← Source PDF of the user's paper
  first_draft.md        ← Markdown transcription of the full paper
  literature_review_v0.md     ← Versioned rewrites of the literature review section
  literature_review_v1.md     ← (increment N on each rewrite; v0 = baseline from draft)
  research_objectives_v0.md   ← Versioned rewrites of the research objectives section
  research_objectives_v1.md   ← (same convention; any paper section can be versioned)
log/papers.md           ← Processing log — single source of truth
log/overview.md         ← Quick digest: one card per paper (problem + conclusion)
log/revisions.md        ← Unified version history for all original_idea/vN files
log/next_steps.md       ← Living priority queue: what's done, what's pending, what needs user input
.claude/skills/         ← Custom slash commands (skills)
```

## Core Rules
1. **Extract, don't invent.** Every summary field must come directly from the paper. If a section is absent, write `Not mentioned in paper.`
2. **One summary per paper.** Filename mirrors the raw file without its extension: `raw/foo.pdf` → `summary/foo.md`
3. **Log every paper.** All processed papers must appear in `log/papers.md` before moving on.
4. **Keep all ideas.** Folders in `review_literature/` are working drafts — accumulate them, decide which to pursue later.
5. **Archive before revising.** Before overwriting `section.md`, copy the current version to `archive/section-YYYY-MM-DD.md`. The `/draft-section` skill does this automatically.
6. **Version any revised paper section.** Each rewrite of any section extracted from `first_draft.md` is saved as `original_idea/<section-name>_vN.md`. `v0` is always the text extracted verbatim from `first_draft.md` (no edits). Never overwrite an existing version — always increment N. Log every version in `log/revisions.md`.

## Summary Template

Each `summary/<paper>.md` must follow this structure exactly:

```markdown
# [Full Paper Title]

**Source:** `raw/<filename.pdf>`
**Authors:**
**Publisher / Venue:**
**Year:**
**Summarized:** YYYY-MM-DD

## Research Problem / Objective

## Methodology

## Dataset

## Evaluation Metrics

## Findings

## Conclusion / Discussion

## Limitations

## Future Work
```

## Log Format (`log/papers.md`)

Markdown table with these columns:

| File | Title | Status | Date |
|------|-------|--------|------|
| paper.pdf | Full title from paper | done | 2026-06-04 |

**Status values:** `done` · `skipped` (unreadable or confirmed duplicate)

## Workflow

### Phase 1 — Summarize Papers
Run `/summarize-papers`:
1. Scan `raw/` for all files
2. Cross-check each filename against `log/papers.md`
3. For each new file: read → generate structured summary → write `summary/<name>.md` → update log
4. Report: how many new papers processed, how many already done, any failures

### Phase 2 — Synthesis & Ideation
After reading the summaries, work collaboratively:
1. Propose a connection, gap, or direction in conversation
2. Run `/propose-idea <title>` to scaffold `review_literature/<slug>/proposed.md`
3. Iterate — flesh out arguments, link supporting evidence from summaries
4. Run `/draft-section <title>` to write a full literature review section to `review_literature/<slug>/section.md`
5. To revise: run `/draft-section <title>` again — the current `section.md` is archived automatically before overwriting
6. Accumulate all ideas; revisit and select the strongest ones later

### Phase 3 — Paper Section Revision (user's own paper)
When revising any section of the user's paper in `original_idea/`:
1. Read `original_idea/first_draft.md` to locate the current text for the section
2. Read `log/revisions.md` to see what has already been done across all sections
3. Discuss and agree on the plan before writing
4. Write `v0` as a verbatim extract of the current text from `first_draft.md` (no edits)
5. Write `v1` (or next N) as the revised version
6. Append entries for both to `log/revisions.md`
7. Never delete or overwrite previous `vN` files — all versions are permanent

**Naming convention:** `original_idea/<section-name>_vN.md`
- `literature_review_vN.md` — literature review section
- `research_objectives_vN.md` — research statement, RQs, gaps, problem formulation
- Any other section follows the same pattern

## Log Format (`log/revisions.md`)

One file tracks all versioned artifacts. Each artifact has its own table and version blocks:

```markdown
# Paper Section Revision History

## literature_review

| Version | Date | Summary of Changes |
|---------|------|--------------------|
| v0 | YYYY-MM-DD | Brief one-line description |
| v1 | YYYY-MM-DD | Brief one-line description |

### v0 — YYYY-MM-DD
**File:** `original_idea/literature_review_v0.md`
**Sections:** list section titles
**Key changes:** what was added vs. the paper's original placeholder
**Rationale:** why this version was written

---

## research_objectives

| Version | Date | Summary of Changes |
|---------|------|--------------------|
| v0 | YYYY-MM-DD | Brief one-line description |
```

## What to Log
- Paper processed → row in `log/papers.md` + card in `log/overview.md`
- New idea created → `review_literature/<slug>/proposed.md`
- Section drafted or revised → `review_literature/<slug>/section.md` (previous version auto-archived)
- Paper section version written → new entry in `log/revisions.md` under the correct artifact heading
- Significant decisions → noted inside the relevant `proposed.md`

Do NOT log: every file read, tool calls, internal reasoning, or minor edits.
