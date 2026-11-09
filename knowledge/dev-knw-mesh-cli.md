---
description: "The mesh CLI contract for agents: the root, the scopes, addressing, output streams, the JSON envelope, option values, the write model, and recovery"
---

# Knowledge: The mesh CLI

The `mesh` command is one registry of commands. The CLI, the help text, `CLI-REFERENCE.md` in the mesh repository, and the request names of `mesh batch` all come from it. This file gives the rules that every command follows. `mesh <command> --help` gives the options of one command.

## The Root and the Settings

`mesh` finds the mesh root upward from the current folder: the first folder with `mesh.json`. `--root <path>` starts the search at another folder. In a Flint, the root is the Flint root.

| Command | What it prints |
|---|---|
| `mesh root` | The root, and a summary of `mesh.json` and `mesh.toml` |
| `mesh info` | The identity, the folders, the counts, the remotes, the peers, and the diagnostic counts |

`mesh.json` holds the identity of the mesh: the id and the Name. `mesh.toml` holds the settings:

| Table | Holds | Who changes it |
|---|---|---|
| `[folders]` | `mesh` (the writable folder), `media`, `resolve` (the read-only scope), `archive` | A person |
| `[types]`, `[sections]`, `[groups]`, `[archive]` | The folders of types, sections, and groups; the closure-exempt keys; the archive tags | A person |
| `[remotes.<name>]` | One sync remote for one section | `mesh pull` and `mesh push` with a URL write a new row |
| `[references.<key>]` | One peer mesh | `mesh references add`, `set`, and `remove` |

Do not edit `[remotes]` or `[references]` by hand. The commands check each row before they write it. The one exception is an old Onyx row with `tag` and `folder` keys: every command refuses that `mesh.toml` (exit 9), so a person or an agent edits that row by hand ([[dev-knw-mesh-sync]] § Old Tag-Scoped Remotes).

## Scopes

- **The writable scope** is the mesh folder (`Mesh/`) and the media folder (`Media/`). Every write stays in it.
- **The read-only scope** is `[folders] resolve`. In a Flint it is `Shards/`. Its titles resolve links, and `get`, `find`, and `heading get` read them. No command writes them.

## Addressing

- **A document.** Give the exact title, or `--path <relpath>` (relative to the mesh root, for example `Mesh/Types/Tasks/(Task) 042 Fix.md`), or `--id <uuid>` (the frontmatter id).
- **A missing title** exits 5. The envelope has `suggestions`. When exactly one title matches in another case, `next` has the exact command.
- **A duplicate title** exits 8. The envelope has `paths` and one `next` command for each copy.
- **A section** is its name (`Mesh SDK Docs`), its key (`mesh-sdk-docs`), or its header title (`(Section) Mesh SDK Docs`).
- **A group** is its key (`reading-list`), its name (`Reading List`), or its definition title (`(Group) Reading List`).
- **Title forms.** A numbered title is `(<Type>) NNN <Name>`. A child title is `<Parent> . <Child>`. A title that starts with `{` is reserved for cross-mesh references.

## Output

| Stream | Holds |
|---|---|
| stdout | The data and the result of the command |
| stderr | A refusal (`✘ <reason>`, then `Next:` and `Or:` lines), warnings (`⚠ <fact>.`), `Next:` hints, an empty note (`No remotes.`), and the dry-run footer (`Dry run: nothing was written.`) |

- An empty result prints nothing on stdout, a note on stderr, and exits 0.
- A write that changes nothing prints one sentence and exits 0, for example `Nothing to change in "Alpha" (Mesh/Alpha.md).` or `No links to rewrite.`
- A dry run prints lines that start with `Would`. It never prints `✓`.
- A list prints one title or one path on each line, with no quotes. It is safe to pipe.
- Color and wrap apply only on a terminal. `NO_COLOR` turns color off.

## The JSON Envelope

With `--json`, stdout holds exactly one line, and stderr is empty:

```json
{"ok":true,"kind":"rename","next":["mesh rename Alpha Gamma --plan 391b… --json"],"warnings":[],"oldTitle":"Alpha","newTitle":"Gamma","plan":"391b…"}
```

- The keys come in this order: `ok`, `kind`, `code` (refusal only), `reason` (refusal only), `next`, `warnings`, then the data.
- `ok` is true only for exit 0.
- `kind` is the request name of the command (`section.list`), or `usage` for a usage error before the command is known.
- `next` holds commands only, the primary command first. `next` and `warnings` are always present.
- A refusal has `code` (UPPER_SNAKE) and `reason` (full sentences).
- A result with a non-zero exit (`check` with an error, `pull` with conflicts) has `ok: false`, its own `code` and `reason`, and the data.
- Some data keys have other names, so that they do not collide with the base keys: `documentId` (in `get`, `props get`, `create`, and `delete`), `plan` (the token of a dry run), `numberText` (in `type next`), `targetKind` (in `tree`), and `succeeded` (in `batch`).
- `mesh batch` prints JSON Lines (one line for each request) unless you add `--json`.

## Option Values

- A value that starts with `-` needs the `=` form: `--match=-x`.
- A free-text option takes the next token as its value, also when it starts with `-`. The free-text options are `--text`, `--find`, `--with`, `--heading`, `--description`, `--title`, `--note`, and `--body`. Example: `--text "- 2026-10-24: Done."`.
- In a bulk edit of frontmatter or tags, the edit arguments come after `--`. Every option comes before `--`: after it, each token is an edit argument, also `--dry-run`, `--plan`, and `--intent`. Example: `mesh props set --select --tag proj/task --expect-count 3 --intent "Task 712" -- status=done`.
- The content of a write comes from `--text` or `--file`, else from piped stdin. `--text ""` is empty content.

## The Write Model

Every write makes a plan first and then applies it. It takes the write lock of the root (`--lock-timeout <ms>`, default 10000), writes in one transaction, and puts a full-file guard on each file that it touches.

| Option | Effect |
|---|---|
| `--dry-run` | Make the plan, print it, and write nothing. A command that takes `--plan` puts the apply command in `next`, and the token in `plan`. |
| `--plan <token>` | Apply only when the new plan has this token. Else exit 7: the mesh changed after the dry run. |
| `--expect-hash <hash>` | Apply only when the file has this full-file hash (from `mesh get --json`). Else exit 7. |
| `--expect-count <n>` | For a bulk edit: apply only when the selection has exactly `<n>` documents. Else exit 7. |
| `--intent <label>` | Record the label on the transaction. It does not change the plan token. |

- **Commands with `--plan`:** the structure commands (rename, move, archive, delete, relink, tag rename, titles dedupe, ensure-ids, media rename), the section and group writes, the bulk edits, the reference writes, `heal`, `address release`, `rename-mesh`, and `ops apply`. `split` takes `--plan` with `--plan-file` ([[dev-knw-mesh-cross_mesh]]).
- **Commands with no `--plan`:** the creates (`create`, `section create`, `group create`, `pointer make`). A create makes a new random id, so no later plan has the token of its dry run. Their `--dry-run` still shows the title and the path.
- **The receipt.** Every write has a transaction id. Add `--json` and read `write.transactionId` (`mesh batch --json` gives `transactionId` at the top level). The text output of a committed write shows `Transaction : <id>`. `pull`, `push`, `sync`, and `resolve` give no transaction id. `mesh txn show <id>` reads the receipt. When you lose the result of a write (a crash or a timeout), read the receipt before you repeat the write. `mesh txn list` shows the time, the outcome, the path count, and the intent of each transaction. `mesh txn show <id>` gives its paths.
- **Malformed frontmatter.** A command never edits a file in part. When it would change a file whose frontmatter does not parse, it exits 9 with `MALFORMED_FRONTMATTER` and names each file. Some commands take `--skip-malformed`: they leave such a file unchanged and list it. To repair the file, read it with `mesh get --path <relpath> --raw`, correct the text, and write it with `mesh update --path <relpath> --raw --file <corrected-file>`.
- **The safety boundary.** The guarantees hold for writers that take the lock: every `mesh` command and every `mesh-local` caller. An editor (Obsidian, a file edit tool) does not take the lock. When it saves a file during a `mesh` write, the guard reports a conflict, and the recovery keeps the bytes that it did not expect. A reader that takes no lock can see part of a transaction while it commits.

## Recovery

| Command | Use |
|---|---|
| `mesh lock status` | The owner of the write lock, and whether its process is alive. A write takes over a dead lock by itself when the same host and boot prove the owner is dead. |
| `mesh lock break --owner <id> --force` | Remove one owner of the lock with no proof. Only when a person confirms that the owner does not write. |
| `mesh txn list` | The receipts (newest first), the unfinished records, the kept records, and the pending transactions. |
| `mesh txn show <id>` | One receipt and its record. |
| `mesh txn recover` | Settle the unfinished transactions of the root now. Every write does this first. |
| `mesh index status` | The counts, the cache state, the transactions, the lock, and the load times. |
| `mesh index rebuild` | Delete the index cache and load the full index again. The cache is derived data. |

An outcome `recovery_required` (exit 7) means that the recovery found bytes that belong to no state of the transaction. It keeps the bytes and the backups in `.mesh/txn/kept/`. Stop and tell the person. [[dev-sk-mesh-recover_write]] gives the steps.

## Command Groups

`mesh --help` lists the commands in these groups:

| Group | Commands | Knowledge |
|---|---|---|
| Mesh | `init`, `root`, `info`, `rename-mesh` | This file, [[dev-knw-mesh-cross_mesh]] |
| Read and query | `check`, `get`, `path`, `list`, `find`, `links`, `backlinks`, `tags`, `tree`, `compile` | [[dev-knw-mesh-read]] |
| Edit | `props`, `tag add`, `tag remove`, `update`, `append`, `heading`, `replace` | [[dev-knw-mesh-edit]] |
| Structure | `rename`, `media rename`, `ensure-ids`, `create`, `type`, `delete`, `archive`, `mv`, `relink`, `tag rename`, `titles dedupe`, `section`, `group`, `split` | [[dev-knw-mesh-structure]], [[dev-knw-mesh-cross_mesh]] |
| Sources and pointers | `sources`, `pointers`, `pointer` | [[dev-knw-mesh-edit]] |
| Cross-mesh references | `references`, `history`, `address`, `heal` | [[dev-knw-mesh-cross_mesh]] |
| Batch and integration | `snapshot`, `batch`, `ops apply`, `index`, `lock`, `txn` | [[dev-knw-mesh-edit]], this file |
| Sync with a remote | `pull`, `push`, `sync`, `resolve`, `conflicts`, `remotes` | [[dev-knw-mesh-sync]] |
| Onyx | `onyx login`, `logout`, `whoami`, `list`, `create`, `share`, `publish`, `address release` | [[dev-knw-mesh-onyx]] |

Aliases: `ls` is `list`, `fm` is `props`, `new` is `create`, and `rm` is `delete`.

The read commands have tiers. A Tier 1 command (`get`, `path`, `find`, `links`, `props get`, `heading get`) reads the folder scan and one file. A Tier 2 command (`list`, `backlinks`, `check`, the section and group reads) loads the full index. The full index uses a cache in `.mesh/`, and each load checks each cache entry against its file.
