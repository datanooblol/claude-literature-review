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
original_idea/          ← The user's own research paper and its evolving literature review
  first_draft.pdf       ← Source PDF of the user's paper
  first_draft.md        ← Markdown transcription of the full paper
  literature_review_v0.md  ← First integrated revision (baseline)
  literature_review_v1.md  ← Subsequent revisions (increment N on each rewrite)
log/papers.md           ← Processing log — single source of truth
log/overview.md         ← Quick digest: one card per paper (problem + conclusion)
log/literature_revision.md ← Version history for original_idea/literature_review_vN.md
.claude/skills/         ← Custom slash commands (skills)
```

## Core Rules
1. **Extract, don't invent.** Every summary field must come directly from the paper. If a section is absent, write `Not mentioned in paper.`
2. **One summary per paper.** Filename mirrors the raw file without its extension: `raw/foo.pdf` → `summary/foo.md`
3. **Log every paper.** All processed papers must appear in `log/papers.md` before moving on.
4. **Keep all ideas.** Folders in `review_literature/` are working drafts — accumulate them, decide which to pursue later.
5. **Archive before revising.** Before overwriting `section.md`, copy the current version to `archive/section-YYYY-MM-DD.md`. The `/draft-section` skill does this automatically.
6. **Version the literature review.** Each rewrite of the user's integrated literature review is saved as a new file: `original_idea/literature_review_vN.md`. Never overwrite an existing version — always increment N. Log every version in `log/literature_revision.md`.

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

### Phase 3 — Literature Review Revision (user's own paper)
When revising the integrated literature review in `original_idea/`:
1. Read `original_idea/first_draft.md` to understand current research objectives and gaps
2. Read `log/literature_revision.md` to see what has been done in previous versions
3. Discuss and agree on the structural plan before writing
4. Write the new version as `original_idea/literature_review_vN.md` (N = next integer after the current highest)
5. Append a new entry to `log/literature_revision.md` describing what changed and why
6. Never delete or overwrite previous `vN` files — all versions are permanent

## Log Format (`log/literature_revision.md`)

Markdown table + per-version summary blocks:

```markdown
# Literature Review Version History

| Version | Date | Summary of Changes |
|---------|------|--------------------|
| v0 | YYYY-MM-DD | Brief one-line description |
| v1 | YYYY-MM-DD | Brief one-line description |

---

## v0 — YYYY-MM-DD

**File:** `original_idea/literature_review_v0.md`
**Sections:** list section titles
**Key changes:** what was added, removed, or restructured vs. the paper's original placeholder
**Rationale:** why this version was written
```

## What to Log
- Paper processed → row in `log/papers.md` + card in `log/overview.md`
- New idea created → `review_literature/<slug>/proposed.md`
- Section drafted or revised → `review_literature/<slug>/section.md` (previous version auto-archived)
- Literature review version written → new entry in `log/literature_revision.md`
- Significant decisions → noted inside the relevant `proposed.md`

Do NOT log: every file read, tool calls, internal reasoning, or minor edits.
