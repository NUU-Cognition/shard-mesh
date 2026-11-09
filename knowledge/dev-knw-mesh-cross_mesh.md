---
description: "Work across meshes: peer rows, braced cross-mesh references, address history and reserved titles, heal, mesh rename, and the split of a section into another mesh"
---

# Knowledge: Across Meshes

A document can link to a document of another mesh. A `[references.<key>]` row of `mesh.toml` binds that other mesh (a peer). A split moves a section into another mesh and rewrites the links across the cut.

## Cross-Mesh References

A cross-mesh reference uses the braced form:

```markdown
[[{ Mesh . <Name> . Mesh Document . <title> }]]
```

- `<Name>` is the Name of the peer mesh (its `mesh.json` Name). The anchor, the display text, and the embed mark work as in a local link.
- A braced reference to this mesh itself resolves here.
- The old form `[[Mesh::Doc]]` is legacy. `mesh check` reports it as `LEGACY_MESH_REF`. An edit that writes it is refused (exit 9). `mesh references migrate --dry-run` plans the rewrite of every old form, and `--plan <token>` applies it.
- A title that starts with `{` is reserved: no local link can name it.

## Peer Rows

A row binds the address of a peer (`@<org>/mesh/<mesh key>`; the org is empty for a mesh with no org) to its identity (the `mesh.json` id that the peer must hold) and at most one locator.

| Command | Effect |
|---|---|
| `mesh references list [--resolve]` | List the rows. `--resolve` also runs discovery and prints the status of each row. |
| `mesh references add <key> --path <mesh-root>` | Add a row for a mesh on disk. The id and the Name come from its `mesh.json`. |
| `mesh references add <key> --provider <provider> --deployment <host> --mesh-id <uuid>` | Add a row for a mesh behind a provider. |
| `mesh references set <key> ...` | Change fields of a row. `--no-locator` leaves the places roster as the only discovery step. |
| `mesh references remove <key>` | Remove a row. The references to its address stay, and `mesh check` reports them as `UNRESOLVED_MESH_REF`. |
| `mesh references resolve [<key> \| <address>]` | Run discovery for one row, or for every row. |

- The key is a settings handle (letters, digits, `-`, and `_`). It never appears in a reference.
- Discovery tries the locator of the row, then the places roster of this machine (`$NUU_HOME/places.json`). Each step checks the `mesh.json` id.
- Discovery statuses: `available`, `unavailable` (normal, exit 0), `mismatch` (another id, exit 9), `ambiguous` (two roots hold the id, exit 8), `refused` (this mesh, or an overlapping root, exit 9), and `invalid` (exit 9).
- Each write takes `--dry-run` and `--plan <token>`.

## Reads Across Meshes

`--across` resolves each cross-mesh reference through the peers:

| Command | Effect |
|---|---|
| `mesh check --across` | Adds `MESH_REF_BROKEN`, `MESH_REF_HEALABLE`, and `MESH_REF_UNAVAILABLE`, and `MESH_REF_RESERVED` for a reference that moved into a mesh with no row here. A local check also gives `MESH_REF_RESERVED` for a link to a reserved local title. |
| `mesh links "<title>" --across` | Resolves each cross-mesh link of one document |
| `mesh backlinks "<title>" --across` | Also lists the peer documents that reference the title |
| `mesh compile "<title>" --across` | Follows a cross-mesh link into the peer |

An unavailable peer is never proof of a missing document. A result that is not complete gives a retry command.

## Address History

Each rename, Name change, release, and move writes one event to `.mesh/renames.jsonl` in the same transaction.

| Command | Effect |
|---|---|
| `mesh history [--since <version>]` | List the events. |
| `mesh address list` | List the reserved old titles and where each one leads. |
| `mesh address release "<title>"` | Free one reserved title after a sweep of this mesh and of every reachable peer. It refuses while a reference remains (`ADDRESS_STILL_REFERENCED`, exit 9). `--force` releases it anyway. |
| `mesh heal --peer <key>` | Rewrite each reference to one peer that its history can heal: a rename, a move, or a new Name. |
| `mesh check --fix-refs` | Heal every reachable peer, then check with `--across`. With `--dry-run`, plan only. |
| `mesh rename-mesh "<Name>"` | Change the Name of this mesh. A peer that references the old Name heals with `mesh heal --peer <key>`. |

A plain read never heals. After a released title is used again, an old reference that a peer still holds reaches the new document.

## Split a Section

`mesh split` moves the members of one section and its header into another mesh: an existing Flint, or a new Onyx mesh.

```bash
mesh split --section <section> --into <flint-root> --dry-run --json > <plan-file>
mesh split --section <section> --into <flint-root> --plan <token> --plan-file <plan-file>
```

- **The section.** `--section` takes the section name, its key, or its header title, as the other section commands do. Give the same section in the dry run and in the apply.
- **The source Name.** The source mesh needs a valid Name in `mesh.json`. Else the split refuses with exit 9 (`SOURCE_NAME_INVALID`). A person sets it with `mesh rename-mesh "<Name>"`.
- **The target.** `--into <path or Flint name>` is an existing Flint, and it keeps its identity and settings. Give the absolute path of its root. `--into onyx:<new mesh name>` creates a new Onyx mesh, private to the login.
- **The plan.** The dry run writes nothing. Its JSON is the plan: save it to a file outside the mesh and review it. The apply takes the same arguments, the same root, `--plan <token>`, and `--plan-file <file>`. It makes the plan again under the locks of both meshes. When anything changed, it refuses with `STALE_PLAN` (exit 7).
- **The references.** A link from a document that stays to a document that moves, or the reverse, gets the braced form. The plan also writes the peer rows of both meshes.
- **No closure.** Only the members move. A folder, a child title, or a link adds no member. The report lists the folder stragglers and the documents that only members link to.
- **Media.** A media file that only members use moves. A media file that others use too is copied.
- **Types.** The members keep their type references across the cut. `--copy-types` copies the definitions that the members use into the target.
- **Onyx targets.** `--accept-unreachable-source` accepts references that the Onyx target cannot route back to the source. `--accept-canonical-frontmatter` accepts the canonical frontmatter bytes that Onyx stores.
- **Refusals.** A section error, a duplicate or colliding title, a number that the target uses (`NUMBER_COLLISION`), a media name with other bytes, a sync remote of the section, or an unfinished transaction refuses the split before any write. Run `mesh section check <section>` and `mesh section fix <section>` first.
- **Outcomes.** `committed`, `not_applied`, or `recovery_required` (exit 7). The target is written and checked first. A member is deleted from the source only after its target copy is verified.
- **After an interruption.** `mesh split status [<split-id>]` prints the records. `mesh split recover <split-id>` settles the split from either root. Run `mesh split abort <split-id>` only for a split into an Onyx target that stopped with `SPLIT_TARGET_REFUSED`. For any other stop, run `split recover`.
- **`recovery_required`.** The split kept bytes that belong to no state of the split, with the backups and the fences. A new recovery gives the same outcome. Stop, and report the paths and the hashes from `mesh split status <split-id>` to the person.
- A split is a large change that other people can see. [[dev-wkfl-mesh-split_section]] gives the stages, with the review of a person before the apply.
