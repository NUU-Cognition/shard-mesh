---
description: "Rename one document and rewrite every link to it, with a dry run and a plan token"
---

> [!important] THIS FILE IS AN INSTRUCTION. WHEN REFERENCED IT IS MEANT TO BE TAKEN AS AN ACTION.

Run `flint shard start mesh` if you haven't already.

# Skill: Rename a Document

Give one document a new title. Every link to it follows.

# Input

- The current title of the document.
- The new title, or the new name for a numbered title (the prefix `(<Type>) NNN` stays).
- (Optional) Permission to rename the child documents (`<old> . <child>`) too.
- An intent label.

# Actions

1. **Confirm the title.** Run `mesh get "<old>" --json`. On exit 5, run `mesh find "<words>"` and use the exact title. On exit 8 (a duplicate title), stop and report: rename one copy only when the task names its path, with `--path`.
2. **Check the new title.** Run `mesh path "<new>" --json`. Exit 0 means that the title exists. Exit 5 means that no document has the exact title, but read `suggestions`: a suggestion that differs from the new title only in case is a case collision, and the rename will refuse it. Then run `mesh address list` and confirm that the new title is not a reserved old title. When the title is not free, stop and report.
3. **See the links.** Run `mesh backlinks "<old>"`. Record the count.
4. **Dry run.** Run `mesh rename "<old>" "<new>" --dry-run --json`. Add `--cascade` only when the input permits the child documents. Add `--full` when `<new>` is a complete title that must not keep the old prefix.
5. **Handle a refusal of the dry run.** Exit 9 with `ADDRESS_RESERVED` means that the new title is an old title of another document: choose another title, or ask. Another exit 9 is a title or case collision: stop and report.
6. **Read the plan.** Confirm `newTitle`, the destination path in `renames`, and the documents in `updatedTitles` whose links change. The text output gives only their count.
7. **Apply.** Take the command from `next` (it has `--plan <token>`). Add `--intent "<label>"`. Run it, and read `write.transactionId`. Exit 7 means that the mesh changed: go back to step 1.
8. **Verify.** Run `mesh backlinks "<new>"` and confirm the count of step 3. Run `mesh check --code BROKEN_LINK --summary`.
9. **Report.** Give the old title, the new title, the count of rewritten documents, and the transaction id. State that the old title is now reserved (`mesh address list`).

# Output

- The document has the new title.
- Every link to it points at the new title.
- The old title is a reserved address.
