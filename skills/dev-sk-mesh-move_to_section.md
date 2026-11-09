---
description: "Move documents into a section, with the tag and the file in one transaction, and keep the section valid"
---

> [!important] THIS FILE IS AN INSTRUCTION. WHEN REFERENCED IT IS MEANT TO BE TAKEN AS AN ACTION.

Run `flint shard start mesh` if you haven't already.

# Skill: Move Documents into a Section

Give documents the section tag and move their files into the section folder.

# Input

- The exact titles of the documents.
- The section: its name, its key, or its header title.
- (Optional) A subfolder below the section folder.
- An intent label.

# Actions

1. **Find the section.** Run `mesh section show "<section>"`. On exit 5 the section does not exist. Create it only when the task asks for it: in a Flint with the Invironments shard, follow its skill `sk-ie-create_section`; else run `mesh section create "<Name>"`.
2. **Check the section.** Run `mesh section check "<section>"`. When it reports an error, stop and report it. Do not move documents into a section with an error.
3. **Check the documents.** Run `mesh get "<title>" --frontmatter` for each document. Record its current section tag, if any.
4. **Check the remote.** Run `mesh remotes`. When the section has a remote, the next push sends the new members. When a document leaves a section with a remote, its far copy stays. Report both facts.
5. **Dry run.** Run `mesh section move "<title>"... --to "<section>" --dry-run`. Add `--folder "<subfolder>"` when the input gives one. A closed section refuses here (exit 9) a move that adds crossing links, and the refusal lists them. Report them, and stop. Do not change the kind of the section unless the task asks for it.
6. **Read the plan.** Confirm each source path, each destination path, and each section change.
7. **Apply.** Run the command in `next` (it has `--plan <token>`). Add `--intent "<label>"`. Add `--json` and read `write.transactionId`.
8. **Verify.** Run `mesh section check "<section>"`, and `mesh list --section "<section>"`.
9. **Report.** Give the moved documents, their new paths, the section, and the transaction id.

# Output

- Each document has the section tag and is in the section folder.
- The section has no section error.
