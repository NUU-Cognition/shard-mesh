---
description: "Check the health of a mesh, sort the diagnostics, apply only the permitted safe repairs with guarded commands, and report the rest"
---

> [!important] THIS FILE IS AN INSTRUCTION. WHEN REFERENCED IT IS MEANT TO BE TAKEN AS AN ACTION.

Run `flint shard start mesh` if you haven't already.

# Skill: Health Check

Find the problems of a mesh. Repair the safe ones only when the input permits it. Report all of them.

# Input

- (Optional) The scope: one section, or a list of codes. The default is the whole mesh.
- The repair scope: `report` (the default: write nothing) or `safe` (apply the safe repairs below).
- An intent label, for the repairs.

# Actions

1. **Count.** Run `mesh check --summary --json`. Add `--section "<section>"` or `--code <codes>` for a limited scope. Record the count of each code.
2. **Sort the codes.**
   - **Safe repairs:** `MISSING_ID` (`mesh ensure-ids --write`). Also `SECTION_FOLDER_MISMATCH` (`mesh section fix <section>`, the tag wins), but only for a section with no remote (`mesh remotes`), and only when the dry run has no `Would tag` line. A fix can tag a file in a section folder, and a later push then publishes it: in each other case, the fix needs a decision.
   - **Repairs that need a decision:** `BROKEN_LINK`, `BROKEN_EMBED`, `AMBIGUOUS_LINK`, `DUPLICATE_TITLE`, `DUPLICATE_TYPE_NUMBER`, `MALFORMED_FRONTMATTER`, `MULTIPLE_SECTIONS`, `SECTION_NOT_CLOSED`, `LEGACY_MESH_REF`, and each other error code.
   - **Information only:** `ORPHAN` and `UNUSED_MEDIA`.
   - **Stop codes:** `UNFINISHED_SPLIT`, `HISTORY_INVALID`, and a kept transaction (`recovery_required`). Report them first. Do not write while they exist.
3. **Read the details.** For each code that is not information only, run `mesh check --code <code> --json`. Record the subjects. Some codes give a `next` command for each diagnostic. For a code with no `next`, take the repair from the table of [[dev-knw-mesh-read]] § The Health Check.
4. **Apply the safe repairs** (only when the repair scope is `safe`). For each safe repair, run it with `--dry-run`. Read the plan. Then run the command in `next` (it has `--plan <token>`), with `--intent "<label>"` and `--json`. Record `write.transactionId`. `mesh section fix` leaves some documents unchanged (two section tags, a tag of no section, a header out of place): report them.
5. **Do not repair the rest.** Do not delete, rename, or unlink to make a diagnostic go away. For each repair that needs a decision, write the subject, the cause, and the proposed command.
6. **Check again.** Run `mesh check --summary` and compare the counts with step 1.
7. **Report.** Give the counts before and after, each repair that you applied with its transaction id, and each open problem with its proposed command.

# Output

- A health report: counts, applied repairs, and open problems with proposed commands.
- With the `safe` scope: the safe repairs applied in guarded transactions.
