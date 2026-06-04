Create a new synthesis idea document in `./review_literature/`.

$ARGUMENTS

**Steps:**

1. Use the provided title from `$ARGUMENTS`. If none given, ask for one before proceeding.
2. Convert the title to a filename slug: lowercase, spaces to hyphens (e.g. "cross-domain transfer" → `cross-domain-transfer.md`).
3. Check if `./review_literature/<slug>.md` already exists. If it does, open it for collaborative editing instead of creating a new file.
4. Create `./review_literature/<slug>.md` using the structure from `idea-template.md`. Replace the placeholder date with today's date.
5. After creating the file, read it back and invite the user to start filling in sections — or begin filling in anything they've already described in the conversation.
