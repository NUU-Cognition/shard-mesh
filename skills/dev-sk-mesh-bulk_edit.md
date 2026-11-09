---
description: "Change the frontmatter, tags, or body of many documents in one guarded transaction: select, dry run, apply the plan, verify"
---

> [!important] THIS FILE IS AN INSTRUCTION. WHEN REFERENCED IT IS MEANT TO BE TAKEN AS AN ACTION.

Run `flint shard start mesh` if you haven't already.

# Skill: Bulk Edit

Change many documents together. All of them change, or none of them changes.

# Input

- The selection: the selection flags of `mesh list` (for example `--type Task --where status=review`).
- The edit: one of `props set`, `props unset`, `props add`, `props remove`, `tag add`, `tag remove`, `append`, or `replace`, with its arguments.
- (Optional) The expected count of documents.
- An intent label (the task or the session).

# Actions

1. **Check the selection.** Run `mesh list <selection> --fields <keys>`. `<keys>` are the keys that the edit changes. Confirm that each document is in scope. Record the count. When the input gives an expected count and the count is different, stop and report both counts.
2. **Correct a wrong selection.** If a document is out of scope, or a document is missing, change the selection flags. Repeat step 1. Do not continue with a wrong selection.
3. **Dry run.** Run the edit with `--select <selection> --dry-run --json`. Put every option before `--`. For `props` and `tag`, put only the edit arguments after `--`:
   ```bash
   mesh props set --select <selection> --dry-run --json -- <key>=<value>
   ```
   For `append` and `replace`, put the edit options (`--text`, `--find`, `--with`) with the other options, and nothing after `--`. A token after `--` is always an edit argument: `--dry-run` after `--` becomes a value, and the command writes it.
4. **Handle a refusal of the dry run.** Exit 2 is a usage error: run `mesh <command> --help` and correct the command. Exit 5 is an unknown section, group, or title in the selection (correct the selection and go back to step 1), or a heading that a member does not have (`HEADING_NOT_FOUND`: correct `--heading`, or the selection). Exit 7 with `MATCH_COUNT` means that a member of a bulk `replace` has another number of matches: correct `--find` or `--count`, or the selection.
5. **Read the plan.** Read each document and each diff. Confirm that the count is the count of step 1. If the plan is wrong, go back to step 1.
6. **Apply.** Take the command from `next`. It has `--plan <token>` before `--`. Add `--intent "<label>"` before `--`, not at the end. Run it. Read `write.transactionId` from the JSON result.
7. **Handle a refusal of the apply.** Exit 7 means that the mesh changed after the dry run (`STALE_PLAN`), or that a guard failed. Go back to step 1. Do not force it. Exit 9 with `MALFORMED_FRONTMATTER` names a file whose frontmatter does not parse: report it, and do not add `--skip-malformed` unless the task permits it.
8. **Verify.** Run `mesh list <selection> --fields <keys>` again, or a selection that finds the new values. Confirm the change.
9. **Report.** Give the selection, the count, the change, and the transaction id.

# Output

- Every selected document has the change, in one transaction.
- A report with the count and the transaction id.
