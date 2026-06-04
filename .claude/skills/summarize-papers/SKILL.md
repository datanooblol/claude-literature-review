Scan `./raw/` for all files (PDF, DOCX, TXT, MD).

Read `./log/papers.md` and collect all filenames that already have status `done` or `skipped`. Skip those files entirely.

For each unprocessed file, work **one at a time**:

1. Read the paper content carefully and thoroughly.
2. Write a structured summary to `./summary/<filename-without-extension>.md` using the template in `summary-template.md`. Fill in every field:
   - **Authors** — list all authors as they appear on the paper
   - **Publisher / Venue** — journal name, conference name, or arXiv if preprint
   - **Year** — publication year
   - All section content must be extracted **directly from the paper**. If a section is not present, write: `Not mentioned in paper.`
3. After writing the summary, append a new row to `./log/papers.md`:
   `| <original-filename> | <full paper title> | done | <today's date YYYY-MM-DD> |`
4. Append a new row to the table in `./log/overview.md`:

```
| [[<filename-without-extension>]] | <1 sentence: core problem> | <1 sentence: approach/methodology> | <1 sentence: main conclusion> |
```

Use the exact filename without extension as the wiki-link target so Obsidian graph view resolves the link correctly.

Repeat for each new paper. When all done, report:
- How many papers were newly summarized
- How many were already done (skipped)
- Any files that could not be read (mark those as `skipped` in the log with a note)
