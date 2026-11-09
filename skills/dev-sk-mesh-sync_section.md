---
description: "Sync one local section with its remote (onyx: or fs:): pull, settle each conflict by an explicit policy, preview the push, then push"
---

> [!important] THIS FILE IS AN INSTRUCTION. WHEN REFERENCED IT IS MEANT TO BE TAKEN AS AN ACTION.

Run `flint shard start mesh` if you haven't already.

# Skill: Sync a Section

Bring one local section and its far side to the same state. Preview the push before it goes out.

# Input

- The remote name from `mesh.toml`.
- An explicit conflict policy for the conflicts that the merge cannot settle: keep the local side, keep the far side, or merge by hand.
- (Optional) Permission to delete far documents. A push deletes the far copy of each local member that was deleted.

# Actions

1. **Read the remote.** Run `mesh remotes`. Confirm the remote name, its URL, and its section.
2. **Check the section.** Run `mesh section check "<section>"`. Repair a section problem only after you read the repair plan (`mesh section fix "<section>" --dry-run`).
3. **Preview the pull.** Run `mesh pull --remote <name> --dry-run`. Read the planned local changes.
4. **Pull.** Run `mesh pull --remote <name> --json`. For an `onyx:` remote, exit 3 means that there is no valid login: stop and ask the person to run `mesh onyx login`.
5. **Settle conflicts.** When the pull reports conflicts (exit 7, `UNRESOLVED_CONFLICTS`), run `mesh conflicts --json`. For each document, apply the input policy:
   - `mesh resolve "<title>" --ours` keeps the local side.
   - `mesh resolve "<title>" --theirs` keeps the far side.
   - For a hand merge, edit the document to remove every marker, then run `mesh resolve "<title>" --merged`.
   Run `mesh conflicts --json` again. Continue only when it lists no document.
6. **Far section tags.** When a far document has a section tag (a `MULTIPLE_SECTIONS` conflict), skip that document. Report it, with the command in `next`: a person must remove the tag on the far side. Then the next pull writes it.
7. **Preview the push.** Run `mesh push --remote <name> --dry-run`. Read each far action. A far delete needs the permission of the input: without it, stop and report the deletes.
8. **Push.** Run `mesh push --remote <name> --json`. On `REMOTE_DRIFT` (exit 7), the far side changed after the pull: go back to step 3. On `UNRESOLVED_MARKERS` (exit 7), go back to step 5. Do not force the push.
9. **Report.** Give the local actions, the far actions, each conflict decision, the documents that a person must fix, and the far version from `mesh remotes` when the remote shows one (an `fs:` remote can show none).

# Output

- The local section holds the merged state.
- The far side holds the accepted local changes.
- No conflict marker remains.
- A report of each conflict decision and each far action.
