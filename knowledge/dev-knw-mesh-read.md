---
description: "Read and query a mesh: get, path, list and its selection flags, find, links, backlinks, tags, tree, compile, the health check, and the section, group, and type reads"
---

# Knowledge: Read and Query

All commands in this file read only, except `mesh check --fix-refs`. Use them before every write, to see the scope of the change.

## Read One Document

| Command | Prints |
|---|---|
| `mesh get "<title>"` | The frontmatter block and the body |
| `mesh get "<title>" --body` | The body only |
| `mesh get "<title>" --frontmatter` | The YAML of the block, as written |
| `mesh get "<title>" --raw` | The exact file |
| `mesh get "<title>" --fields status,priority` | Those frontmatter keys (a missing key is null) |
| `mesh get "<title>" --json` | Also the path, the `documentId`, and the full-file `hash` (the value for `--expect-hash`) |
| `mesh path "<title>"...` | The path of each title, one on each line |
| `mesh props get "<title>" [<key>]` | The frontmatter block, or the value of one key |
| `mesh heading get "<title>" "## Notes"` | The text of one heading section, with its subsections |

A heading is the exact heading text. Add `#` marks to choose one level (`"## Notes"`). When two headings have the same text, the command exits 8 and gives one command with `--line <n>` for each heading.

## Select Documents

`mesh list` prints the documents that match every selection flag, in title order.

| Flag | Matches |
|---|---|
| `--tag <tag>` | The frontmatter tag, with or without `#` |
| `--type <type>` | The `(Type)` of the title, for example `Task` |
| `--section <section>` | A member of the section |
| `--unsectioned` | A document with no section tag |
| `--group <group>` | A member of the group |
| `--any-group <group>` | A member of one of the groups |
| `--folder <relpath>` | A file inside the folder |
| `--where <key>=<value>` | The frontmatter value is equal (a list matches one item). `<key>!=<value>` negates. |
| `--has <key>`, `--missing <key>` | The key has a value that is not null, or it has none |
| `--links-to <title>` | The document links to the title |
| `--linked-from <title>` | A link of that document resolves to this one |
| `--match <text>` | A part of the title, with no case |

A repeated `--type`, `--folder`, `--section`, or `--any-group` means one of the values. Another repeated flag means every value. An unknown section or group exits 5.

| Output option | Effect |
|---|---|
| `--format titles` | One title on each line (the default) |
| `--format paths` | One path on each line |
| `--fields a,b` | A table with these frontmatter keys (TSV) |
| `--sort <key>[:desc]` | Sort by `title`, `path`, `id`, or a frontmatter key |
| `--limit <n>`, `--offset <n>` | Take a page of the result |
| `--no-archive` | Leave out the files under `[folders] archive` |

The same selection flags choose the documents of a bulk edit ([[dev-knw-mesh-edit]]).

Examples:

```bash
mesh list --type Task --where status=review --fields status,priority
mesh list --section "Mesh SDK Docs" --format paths
mesh list --tag proj/task --where status=done --missing completed
mesh list --links-to "(Spec) Mesh CLI Output Contract"
mesh list --unsectioned --folder Mesh/Main --no-archive
```

## Search

- `mesh find "<query>"` searches the titles of the mesh and of the read-only scope, with no case: the exact matches first, then the prefix, word prefix, and part matches. `--limit <n>` caps each list (default 20).
- `mesh find "<query>" --content` also prints each body line that holds the query outside code.

## Links

- `mesh links "<title>"` prints each link of one document: the line, the kind (body, embed, frontmatter), the status, the target, and the files that it reaches. The statuses are `resolved`, `read-only`, `ambiguous`, `broken`, `session-reference` (a frontmatter link to an Orbh session id), and `cross-mesh`. `--kind` limits the kind.
- `mesh backlinks "<title>"` prints the documents that link to the title. `--context` prints the line of each link. The title must name exactly one file. For a title with no file, or a duplicate title, use `mesh list --links-to "<title>"`.
- A link resolves by one rule: a title of the mesh or of the read-only scope first, then a media file.

## Tags, Tree, and Compile

- `mesh tags [--prefix <prefix>] [--counts]` lists the frontmatter tags without `#`.
- `mesh tree ["<title>"]` prints the `Parent . Child` hierarchy.
- `mesh compile "<title>" [--depth <n>]` prints the body, then the body of each linked document under `# Linked: <title>`. It is the mesh form of `flint helper stitch`, and it prints to stdout.

## The Health Check

`mesh check` loads the mesh and prints the diagnostics. It exits 1 when an error diagnostic exists.

| Option | Effect |
|---|---|
| `--summary` | The count of each code and its ten most frequent subjects |
| `--code <code>` | Only these codes (repeat it, or give a list with commas) |
| `--section <section>` | Limit the section codes to one section |
| `--across` | Also resolve the cross-mesh references through the peers |

A code with one exact repair gives a `next` command for each diagnostic: for example `MISSING_ID` (`mesh ensure-ids --dry-run`), `SECTION_FOLDER_MISMATCH` (`mesh section fix <key> --dry-run`), `MULTIPLE_SECTIONS` (a `mesh section move` into each of its sections, and `mesh section unassign`), `LEGACY_MESH_REF`, and `MESH_REF_HEALABLE`. `next` of the envelope then holds each distinct command once. Other codes give no `next`: use the table below. The usual repairs:

| Code | Repair |
|---|---|
| `BROKEN_LINK`, `BROKEN_EMBED` | Correct the link: `mesh relink "<from>" "<to>"`, or create the target. |
| `DUPLICATE_TITLE`, `AMBIGUOUS_LINK` | Give each copy its own title: `mesh titles dedupe --map <relpath>=<new title>`. |
| `MISSING_ID` | `mesh ensure-ids --write` |
| `MALFORMED_FRONTMATTER` | Read `mesh get --path <relpath> --raw`. Write the corrected file with `mesh update --path <relpath> --raw`. |
| `DUPLICATE_TYPE_NUMBER` | Rename one copy to a free number (`mesh type next <Type>` gives it). |
| `MULTIPLE_SECTIONS` | Choose the one section: `mesh section move "<title>" --to <section> --dry-run`, then apply the plan. Or remove every section: `mesh section unassign "<title>" --dry-run`. A section tag changes only with the place of the file: `tag remove` and `props set` refuse it (exit 9). |
| `SECTION_FOLDER_MISMATCH` | `mesh section fix <section> --dry-run`, then apply the plan. |
| `SECTION_NOT_CLOSED` | `mesh section crossings <section>`. Remove the crossing links, or make the section open. |
| `LEGACY_MESH_REF` | `mesh references migrate --dry-run`, then apply the plan. |
| `UNFINISHED_SPLIT` | `mesh split status`, then `mesh split recover <split-id>`. |
| `ORPHAN`, `UNUSED_MEDIA` | Information only. They never fail the check. |

`mesh check` also lists the unfinished, kept, and pending transactions. They do not change the exit code. [[dev-sk-mesh-health_check]] gives the steps of a full check.

## Sections, Groups, and Types

| Command | Prints |
|---|---|
| `mesh section list` | Each section: key, name, kind, members, crossing links, diagnostics, folder. The last row counts the unsectioned documents. |
| `mesh section show <section>` | The header, the kind, the description, each member, the crossing links, and the inbound count |
| `mesh section crossings <section>` | The crossing links, grouped by target |
| `mesh section check [<section>]` | Only the section diagnostics. Exit 1 when one exists. |
| `mesh group list` | Each group: key, definition, members, sections that the members span |
| `mesh group show <group>` | The definition, then each member with its section, type, and description |
| `mesh type list` | Each type: tag, folders, count, and next number |
| `mesh type next <Type>` | The next number. It reserves nothing. |

## Other Reads

| Command | Prints |
|---|---|
| `mesh remotes` | Each sync remote: name, URL, section, kind, members, last sync version |
| `mesh references list [--resolve]` | Each peer mesh ([[dev-knw-mesh-cross_mesh]]) |
| `mesh history` | The address history: renames, Name changes, releases, moves |
| `mesh address list` | The reserved old titles |
| `mesh sources list [--kind <kind>]` | Each document with a `#source/<kind>` tag: title, kind, type (pointer or note), origin (`points-to` or `from`). A tag with a kind that is not lowercase kebab gives a warning |
| `mesh pointers list`, `mesh pointer show "<title>"` | The pointer documents |
| `mesh txn list`, `mesh txn show <id>` | The transaction receipts |
