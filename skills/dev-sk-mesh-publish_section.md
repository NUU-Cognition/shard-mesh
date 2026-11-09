---
description: "Create a new Onyx mesh and publish one complete local section to it"
---

> [!important] THIS FILE IS AN INSTRUCTION. WHEN REFERENCED IT IS MEANT TO BE TAKEN AS AN ACTION.

Run `flint shard start mesh` if you haven't already.

# Skill: Publish a Section

Create a new Onyx mesh. Then publish one complete local section to it.

# Input

- The local section: its name, its key, or its header title.
- The name of the new Onyx mesh.
- A stable nonce for create retries.
- (Optional) A remote name. The section key is the default.

# Actions

1. **Check the remotes.** Run `mesh remotes` in the Flint root. Confirm that the section has no remote. Confirm that the section key is not a remote name. When the input gives another remote name, confirm that it is free too.
2. **Check the section.** Run `mesh section check "<section>"`. Stop before the create when it reports an error, `SECTION_NOT_CLOSED` included. Read a repair plan (`mesh section fix "<section>" --dry-run`) before you apply it. Run the check again after a repair. Continue only when it reports no section error.
3. **Check the login.** Run `mesh onyx whoami`. On exit 3, stop: a person must run `mesh onyx login`.
4. **Create the Onyx mesh.** Run `mesh onyx create "<name>" --nonce <key>`. Record the mesh id. Use the same nonce for each retry of this create.
5. **Dry run the first push.** Run `mesh push --remote onyx:<mesh-id> --section "<section>" --dry-run`. Add `--name <remote-name>` when the input gives another remote name. Read each action and each crossing-link warning.
6. **Push.** Run the same command without `--dry-run`. Do not add `--force`: the first push to an empty Onyx mesh does not need it.
7. **Verify.** Run `mesh remotes`. Confirm that one remote now names the section and the new `onyx:` URL. Its name is the section key, unless step 5 used `--name`.
8. **Report.** Give the section, the remote name, the Onyx mesh id, the applied actions, and the crossing-link warnings.

# Output

- One new Onyx mesh.
- One remote row for the section.
- The section documents and their embedded media on the far side.
- A report of the applied actions and the crossing-link warnings.
