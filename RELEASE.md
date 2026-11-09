# 0.1.0

- First version. The shard teaches agents to use the `mesh` CLI for all document work: read and query, edits, structure, sections and groups, sync, Onyx, cross-mesh references, and split.
- The init gives the model, the choice of tool, the rules for agents, and the exit codes. The required reading `knw-mesh-cli` gives the CLI contract: the envelope, the addressing, the option values, the write model, and the recovery commands.
- New skills: bulk edit, rename a document, move documents into a section, atomic batch, health check, and recover a write.
- New workflow: split a section, with a review of the person before the apply.
- The Onyx shard 0.2.0 is folded into this shard. Its files have new names:

| Onyx 0.2.0 | Mesh 0.1.0 |
|---|---|
| `init-onyx` | `init-mesh` |
| `setup-onyx` | `setup-mesh` (the Onyx login is now optional) |
| `knw-onyx-cli` | `knw-mesh-onyx` (the Onyx group) and `knw-mesh-cli` (the CLI contract) |
| `knw-onyx-section_sync` | `knw-mesh-sync` |
| `sk-onyx-publish_section` | `sk-mesh-publish_section` |
| `sk-onyx-sync_section` | `sk-mesh-sync_section` (now for each remote kind) |
| `sk-onyx-unpublish_document` | `sk-mesh-unpublish_document` |
| `sk-onyx-clone_mesh` | `sk-mesh-clone_mesh` |
| `mig-onyx-0.1.0-to-0.2.0` | `knw-mesh-sync` § Old Tag-Scoped Remotes |
