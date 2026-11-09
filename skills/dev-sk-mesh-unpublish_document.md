---
description: "Delete one document from the far side of its section remote, and detach its local copy from the section"
---

> [!important] THIS FILE IS AN INSTRUCTION. WHEN REFERENCED IT IS MEANT TO BE TAKEN AS AN ACTION.

Run `flint shard start mesh` if you haven't already.

# Skill: Unpublish One Document

Delete one far document. Keep the local file, and remove its section membership.

# Input

- The local document title.
- Permission to delete the far copy.
- (Optional) Permission to use `--force` when a far edit can be lost.

# Actions

1. **Find the remote.** Run `mesh remotes`. Confirm the remote of the section of the document.
2. **Preview.** Run `mesh section unassign "<document>" --delete-remote --dry-run`. Read the plan: the far delete and the new local path.
3. **Unpublish.** Run the command in `next` (it has `--plan <token>`).
4. **A far edit** (exit 7): the far document changed after the last sync, or it has no base entry. Run `mesh pull --remote <name>`. Read the far edit. Repeat from step 2 only after the person or the task confirms the delete.
5. **Force** only when the input permits the loss of the far edit. Preview it first: `mesh section unassign "<document>" --delete-remote --force --dry-run`. Read the plan. Then run the command in `next` (it has `--force` and `--plan <token>`).
6. **No far copy** (exit 9): run `mesh section unassign "<document>"` to detach the local document only.
7. **Verify.** Run `mesh path "<document>"`. Confirm that the local file is outside the section folder. Run `mesh get "<document>" --frontmatter` and confirm that it has no section tag.
8. **Report.** Give the title, the new local path, the remote, the result of the far delete, and whether `--force` was used.

# Output

- The far document is gone, when it existed.
- The local file stays, outside the section folder, with no section membership.
- A report that states whether the command protected or replaced a far edit.
