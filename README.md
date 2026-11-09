# Mesh

Use the `mesh` CLI to read, edit, restructure, and sync the documents of a mesh, Onyx included.

The shard tells an agent which `mesh` command to use for each job, and how to use it safely: the dry run, the plan token, the guards, the write lock, and the receipt. It replaces the Onyx shard: the Onyx section sync is one part of it.

This repository is the source of the Mesh shard (shorthand `mesh`), a core shard of Flint. Edit the files here. Then run `flint shard build mesh` to build the shard in `Shards/Mesh/`.

## Structure

```
Shards/(Source Remote) Mesh/
  shard.yaml                                # Manifest
  dev-init-mesh.md                          # Model, tool choice, rules, exit codes
  dev-setup-mesh.md                         # Per-machine check of the command and the root
  knowledge/
    dev-knw-mesh-cli.md                     # The CLI contract (required reading)
    dev-knw-mesh-read.md                    # Read, query, and the health check
    dev-knw-mesh-edit.md                    # Frontmatter, tags, body, create, bulk edits, batches
    dev-knw-mesh-structure.md               # Rename, move, archive, delete, links, sections, groups
    dev-knw-mesh-sync.md                    # Section remotes, pull, push, sync, conflicts, clone
    dev-knw-mesh-onyx.md                    # The NUU account login and the Onyx-only commands
    dev-knw-mesh-cross_mesh.md              # Peers, cross-mesh references, history, split
  skills/
    dev-sk-mesh-bulk_edit.md                # Change many documents in one transaction
    dev-sk-mesh-rename_document.md          # Rename a document and rewrite its links
    dev-sk-mesh-move_to_section.md          # Move documents into a section
    dev-sk-mesh-atomic_batch.md             # Apply different writes as one transaction
    dev-sk-mesh-health_check.md             # Check the mesh and report or repair
    dev-sk-mesh-recover_write.md            # Settle a failed or lost write
    dev-sk-mesh-publish_section.md          # Publish one section to a new Onyx mesh
    dev-sk-mesh-sync_section.md             # Sync one section with its remote
    dev-sk-mesh-unpublish_document.md       # Delete one far document and detach it
    dev-sk-mesh-clone_mesh.md               # Clone one Onyx mesh into a new section
  workflows/
    dev-wkfl-mesh-split_section.md          # Split a section into another mesh, with a review
```

## Sources

- `Repos/mesh/packages/mesh-cli/CLI-REFERENCE.md` and `mesh --help --json`
- `(Spec) Mesh CLI Output Contract`
- `(Spec) Mesh Section and Group Model` and its parts Sections, Groups, Mesh Level, and Sync and Split
- The Onyx shard 0.2.0 (folded into this shard)
