Draft or revise a fleshed-out literature review section for an existing idea.

$ARGUMENTS

**Steps:**

1. Use the provided title from `$ARGUMENTS`. If none given, infer the slug from recent conversation context or ask.
2. Convert title to a slug: lowercase, spaces to hyphens.
3. Check if `./review_literature/<slug>/proposed.md` exists. If it does, read it to understand the idea's scope, relevant papers, and open questions. If it doesn't exist, warn the user and suggest running `/propose-idea` first.
4. Read relevant summaries from `./summary/` as needed to ground the section in paper evidence.
5. If `./review_literature/<slug>/section.md` already exists:
   a. Read it to understand the current version and what has changed.
   b. Archive it: copy the current content to `./review_literature/<slug>/archive/section-<today's date YYYY-MM-DD>.md` before overwriting. Create the `archive/` folder if it doesn't exist.
6. Draft or revise the section using:
   - The structure and groupings from `proposed.md`
   - Evidence and findings from the relevant `./summary/` files
   - Any directions or feedback given in the current conversation
7. Write the result to `./review_literature/<slug>/section.md`. The file should open with a header, a **Based on** link to the `proposed.md`, and a **Drafted/Revised** date.
8. Confirm: report what was written, whether a previous version was archived, and invite the user to review or give feedback.
