---
description: "Edit documents with the mesh CLI: frontmatter, tags, body text, heading sections, guarded replace, create, bulk edits, atomic batches, sources, and pointers"
---

# Knowledge: Edit Documents

Each edit command in this file changes one document in one guarded transaction, or many documents with `--select`. Each one takes `--dry-run`, `--expect-hash <hash>` (a single document), and `--intent <label>`. A dry run of a single edit prints the diff. The creates (`create` and `pointer make`) take `--dry-run` and `--intent`, but no `--expect-hash` and no `--plan`.

## Frontmatter

| Command | Effect |
|---|---|
| `mesh props set "<title>" <key>=<value>...` | Set top-level keys. A value is JSON when it parses (`3`, `true`, `null`, `["a"]`), else text. `--string` keeps every value as text. |
| `mesh props unset "<title>" <key>...` | Remove keys. A key that is not there is ignored. |
| `mesh props add "<title>" <key> <value>...` | Append values to a list key, with no duplicates. A missing key becomes a list. `--json-values` reads each value as JSON. |
| `mesh props remove "<title>" <key> <value>...` | Remove values from a list key. The key goes with its last value. |

- `fm` is an alias of `props`.
- The `id` key cannot change or go (exit 9). `mesh ensure-ids --write` adds a missing id.
- Comments, quotes, and the order of the other keys stay.
- A value that starts with `-` comes after `--`: `mesh props add "(Task) 665 X" offsets -- -1 -2`. Every option comes before `--`. After `--`, each token is a value, also `--dry-run`.
- A frontmatter link is a text value: `mesh props add "(Task) 712 X" orbh-sessions "[[<session-id>]]"`.
- `status=done` gives text. `priority=3` gives a number. Add `--string` when the key must hold text.

## Tags

- `mesh tag add "<title>" <tag>...` adds tags to the `tags` list. A tag is given with or without `#`. A new tag takes the spelling of the first tag of the document.
- `mesh tag remove "<title>" <tag>...` removes tags. The `tags` key goes with its last tag.
- To rename a tag in every document, use `mesh tag rename` ([[dev-knw-mesh-structure]]).

## Body

| Command | Effect |
|---|---|
| `mesh append "<title>" --text "<text>"` | Add the text after the last line of the body. `--heading "## Log"` adds it at the end of that heading section. |
| `mesh heading set "<title>" "## Status" --text "<text>"` | Replace the text of one heading section. Its subsections go too, with a warning. Empty content removes the text. |
| `mesh replace "<title>" --find "<text>" --with "<text>"` | Replace each match, only when the body has exactly `--count <n>` matches (default 1). Else exit 7 and nothing is written. |
| `mesh update "<title>" --file <path>` | Replace the whole body. The frontmatter block stays byte for byte. |
| `mesh update "<title>" --raw --file <path>` | Replace the whole file. The document keeps its id. This is the only way to change a file whose frontmatter does not parse. |

- The content comes from `--text` or `--file`, else from piped stdin.
- `replace` does not count a match in code unless you add `--include-code`. `--regex` reads `--find` as a JavaScript regular expression (flags `g` and `m`). `--with` can then use `$1` and `$<name>`.
- Example: `mesh replace "(Task) 712 X" --find "- [ ] Ship it" --with "- [x] Ship it"`.

## Create

```bash
mesh create "<title>" [--folder <relpath>] [--section <section>] [--tag <tag>]... [--set <key>=<value>]... [--add <key>=<value>]... [--content-file <path>] [--if-missing] [--dry-run]
```

- **Numbered artifacts.** With `--type <Type>`, the title is the name: the command writes `(<Type>) NNN <name>`. It reserves the next number under the write lock and adds the tag of the type. The folder is the `Location` row of the type. `mesh type next <Type>` only previews the number. Do not write a numbered title by hand.
- **The content file.** `--content-file <path>` writes that file as the document after its tokens expand: `{{number}}`, `{{id}}`, `{{title}}`, `{{date}}`, `{{author}}`, `{{session}}`. `--author <name>` and `--session <id>` give the values. Then `--set` replaces keys, and `--add` and `--tag` append values. The command adds no id to a content file: put `id: {{id}}` in its frontmatter. Without a content file, the document has a new id and its tags.
- **The place.** With `--section`, the file goes in the section folder, or in a `--folder` below it. A `--folder` in a section folder needs `--section`. A title that starts with `(Section) ` is refused: use `mesh section create`.
- **Repeats.** `--if-missing` exits 0 and writes nothing when the title exists.
- A create has no `--plan`: the new id is random. The JSON result gives the new id in `documentId`.
- When a shard gives its own skill to create its type (for example a task of the Projects shard), use that skill. It gives the template.

## Bulk Edits

`props set`, `props unset`, `props add`, `props remove`, `tag add`, `tag remove`, `append`, and `replace` can change many documents in one transaction:

```bash
mesh props set --select --tag proj/task --where status=review --dry-run -- status=reviewed
mesh props set --select --tag proj/task --where status=review --plan <token> -- status=reviewed
```

- `--select` and the selection flags of `mesh list` ([[dev-knw-mesh-read]]) choose the documents.
- A dry run needs no guard. To apply, the edit needs `--expect-count <n>` or `--plan <token>`. Else it is refused.
- For `props` and `tag`, the edit arguments come after `--`, and every option (`--dry-run`, `--plan`, `--intent`, `--expect-count`) comes before `--`. An option after `--` becomes a value or a tag, and the command writes it. For `append` and `replace`, the edit options (`--text`, `--find`, `--with`, …) come with the other options, and nothing comes after `--`.
- Each member keeps its own full-file guard. The commit is one transaction: all members change, or none.
- [[dev-sk-mesh-bulk_edit]] gives the steps.

## Atomic Batches

`mesh batch` runs JSON Lines requests against one loaded mesh. Use it when different commands must change the mesh together.

```json
{"id":"remove-old-tag","cmd":"tag.remove","args":{"_":["Alpha","onyx/field-notes"]}}
{"id":"move","cmd":"section.move","args":{"_":["Alpha"],"to":"field-notes"}}
```

- `cmd` is a command name of `mesh --help`, with `.` or a space between the words (`props.set`, `fm set`, `section.move`).
- `args` holds the long options by name (`"dry-run": true`, `"expect-count": 17`), and `"_"` holds the positional arguments in order.
- `--file <path>` reads the requests from a file, else from stdin.
- **Without `--atomic`**, each request runs as its own command, in order. A failed request does not stop the batch unless you add `--stop-on-error`.
- **With `--atomic`**, the batch takes one lock and writes one transaction. Each request plans against the state of the requests before it. When one request fails, nothing is written. `--dry-run` with `--atomic` plans every request and writes nothing.
- **Order rules of an atomic batch.** A request that needs the full index (`list`, `check`, `rename`, `delete`, a bulk edit) comes before the first write. A write that moves files (`mv`, `archive`, `section move`, `section unassign`, `section rename`, `section merge`, `section delete`, `section fix`) is the last write. A media operation is the last request. `pull`, `push`, `sync`, `resolve`, `index rebuild`, `index clear`, `split`, `split recover`, and `txn recover` cannot be in it.
- **Output.** One JSON line for each request, with `id` and `exitCode`. With `--json`, one envelope with every line in `results`. The exit code is the code of the first request that failed.
- [[dev-sk-mesh-atomic_batch]] gives the steps.

## Raw Operations and Snapshots

- `mesh ops apply <file>` applies a raw `MeshOperation[]` array (create, update, delete, rename, and the media operations). It does not check cross-mesh references. It is for tools. Use the commands for agent work.
- `mesh snapshot export <file>` and `mesh snapshot import <file>` move a whole mesh through the wire JSON format. A snapshot holds no folders. `import --force` deletes every existing document and media file. Do not run it without permission.

## Sources and Pointers

- `mesh pointer make <address> [--title <title>] [--body <text>] [--tag <tag>]...` writes a pointer document in `Mesh/Pointers/`: a live embed of one entity (`@v1/<org>/<type>/<key>#<app>`).
- A source is a role, not a type: any document with a tag `#source/<kind>`. The kind is lowercase kebab: `web`, `docs-source`, `paper`. A source is a pointer or a plain note.
  - A pointer source: `mesh pointer make <address> --tag source/<kind>`. Its origin is `points-to`, for example the address of a citation of NUU Citations (`@v1/<owner>/citation/<uuid>#citations`).
  - A note source: `mesh create "<title>" --tag source/<kind> --set from=<url>`. Its origin is `from`: a web URL or a v1 address. The body holds the text.
  - To make a document a source, add the tag: `mesh tag add "<title>" source/<kind>`. To stop it, remove the tag (`mesh tag remove`), or delete the document (`mesh delete`).
  - `mesh pointer make` refuses (exit 2) a source tag whose kind is not lowercase kebab.
