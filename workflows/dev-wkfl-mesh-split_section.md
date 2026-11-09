---
description: "Split one section into an existing Flint or a new Onyx mesh: plan, review with the person, apply the saved plan, and verify both meshes"
---

> [!important] THIS FILE IS AN INSTRUCTION. WHEN REFERENCED IT IS MEANT TO BE TAKEN AS AN ACTION.

Run `flint shard start mesh` if you haven't already.

# Workflow: Split a Section

Move one section, with its header and its members, into another mesh. The links across the cut become cross-mesh references.

# Input

- The section to move: its name, its key, or its header title.
- The target: the absolute path of the root of an existing Flint, or `onyx:<new mesh name>`.
- (Optional) Whether to copy the type definitions that the members use (`--copy-types`).

# Actions

## Stage 1: Prepare

1. Run `mesh section show "<section>" --json`. Record the section key, the members, the crossing links, and the inbound count. Use the key in each command below.
2. Run `mesh root --json`. Record the source `root`, its id (`config.id`), and its Name (`config.name`). The source needs a valid Name, else the split refuses (`SOURCE_NAME_INVALID`). Report a missing Name: a person sets it with `mesh rename-mesh`.
3. Run `mesh section check "<section-key>"`. A section error stops the split. Repair it with `mesh section fix "<section-key>" --dry-run`, then the plan, only after the person agrees.
4. Run `mesh remotes`. A sync remote of the section stops the split. Report it.
5. Run `mesh txn list --limit 5` and `mesh split status`. An unfinished transaction or an unsettled split stops the split. Settle it first ([[dev-sk-mesh-recover_write]]).
6. **For an existing Flint target:** confirm that `<target>/mesh.json` exists. Run `mesh root --root "<target>" --json`. Confirm that its `root` is the target path, and that its `config.id` is not the source id. `--root` only starts an upward search: a wrong path can find the source mesh. When the check fails, stop. Then run `mesh txn list --root "<target>" --limit 5`.
7. **For an Onyx target:** run `mesh onyx whoami`. On exit 3, stop: a person must run `mesh onyx login`.
8. Once nothing stops the split, progress to the next stage.

## Stage 2: Plan

1. Make the plan, and save it in a temporary folder outside both meshes:
   ```bash
   mesh split --section <section-key> --into "<target>" --dry-run --json > "<plan-file>"
   ```
   Add `--copy-types` when the input asks for it.
2. Read the plan file: the members, the documents that only members link to (they do not move), the folder stragglers, the rewritten references, the media that move or are copied, the numbers, and the warnings.
3. **For an Onyx target:** a refusal with `SOURCE_UNREACHABLE_FROM_TARGET` or `FRONTMATTER_BYTES_CHANGE` waits for the person. Record the titles that it lists, and progress to the next stage. Do not add `--accept-unreachable-source` or `--accept-canonical-frontmatter` yet.
4. Each other refusal (exit 9 or 8) stops the workflow. Report each refusal and its next command. Make the plan again when the cause is settled.
5. Once a plan exists, or only the two Onyx refusals remain, progress to the next stage.

## Stage 3: Review

1. Show the person a summary of the plan: the section, the target, the member count, the documents that stay behind, the references that become cross-mesh, the media, and each warning.
2. **For an Onyx target:** ask the person about each `--accept-*` option that the plan needs. Make the plan again with the accepted options. When the person declines an option, stop.
3. Ask the person to approve the apply. A split deletes the members from the source after it verifies the target.
4. Once the person approves, progress to the next stage.

## Stage 4: Apply

1. Read the token: the `plan` key of the plan file.
2. Run the apply with the same arguments, from the same root:
   ```bash
   mesh split --section <section-key> --into "<target>" --plan <token> --plan-file "<plan-file>" --intent "<label>" --json
   ```
   Add the same `--copy-types` and `--accept-*` options as in the plan.
3. `STALE_PLAN` (exit 7) means that a mesh changed. Go back to Stage 2.
4. An interruption (no login, no network, a stopped process): run `mesh split status <split-id>`, then `mesh split recover <split-id>`. Run `mesh split abort <split-id>` only for an Onyx target that stopped with `SPLIT_TARGET_REFUSED`, and only after the person agrees.
5. `recovery_required` (exit 7): stop. A new recovery gives the same outcome. Report the paths and the hashes from `mesh split status <split-id>` to the person.
6. Once the outcome is `committed`, progress to the next stage.

## Stage 5: Verify

1. Run `mesh split status <split-id>`. Confirm that the split is settled.
2. In the source: run `mesh check --summary`, `mesh check --across --summary`, and `mesh references list`.
3. In a Flint target: run `mesh section check "<section-key>" --root "<target>"` and `mesh check --across --summary --root "<target>"`.
4. Delete the plan file.
5. Report to the person: the split id, the outcome, the moved members, the documents that stayed, the new peer rows, and the check results.

# Output

- The section and its members live in the target mesh.
- The links across the cut are cross-mesh references, with peer rows in both meshes.
- A settled split with a report.
