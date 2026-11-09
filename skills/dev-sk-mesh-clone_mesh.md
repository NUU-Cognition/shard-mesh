---
description: "Clone one Onyx mesh into a new local section"
---

> [!important] THIS FILE IS AN INSTRUCTION. WHEN REFERENCED IT IS MEANT TO BE TAKEN AS AN ACTION.

Run `flint shard start mesh` if you haven't already.

# Skill: Clone an Onyx Mesh

Create a new local section from one Onyx mesh.

# Input

- The Onyx mesh id.
- The name of the new local section.
- (Optional) A remote name. The section key is the default.

# Actions

1. **Check the access.** Run `mesh onyx list`. Confirm that the account can read the Onyx mesh. On exit 3, stop: a person must run `mesh onyx login`.
2. **Check the section.** Run `mesh section list`. Confirm that no row has the section key.
3. **Check the remotes.** Run `mesh remotes`. Confirm that the section key is not a remote name. When the input gives another remote name, confirm that it is free too. Confirm that no remote names the new section.
4. **Dry run.** Run `mesh pull --remote onyx:<mesh-id> --section "<Name>" --dry-run`. Add `--name <remote-name>` when the input gives another remote name. Read the complete plan.
5. **Clone.** Run the same command without `--dry-run`. Do not add `--force`.
6. **Far section tags.** When a far document has a section tag, the pull does not write it. Report it: a person must remove the tag in Onyx. Then repeat steps 4 and 5. The command never removes that far tag.
7. **Header conflict.** When the far side has a section header with another key, the clone refuses with exit 9. Stop. Choose a section name that matches the far header, or ask the person to correct the far header.
8. **Check the result.** Run `mesh section check "<Name>"`.
9. **Verify.** Run `mesh remotes`. Confirm that the new remote names the new section and the expected `onyx:` URL. Its name is the section key, unless step 4 used `--name`.
10. **Report.** Give the section, the remote name, the mesh id, the document count, the media count, and the conflicts.

# Output

- One new local section, with its header (a new header when the far side had none).
- One remote row for the Onyx mesh.
- Local documents with the section tag.
