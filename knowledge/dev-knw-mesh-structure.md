---
description: "Change the structure of a mesh: rename, move, archive, delete, relink, tag rename, duplicate titles, ids, sections, and groups"
---

# Knowledge: Structure

A structure write changes titles, places, links, or membership. It can touch many files. Use the safe sequence for each one:

1. **Read the scope.** `mesh get`, `mesh backlinks`, `mesh section show`, or `mesh list`.
2. **Dry run.** Add `--dry-run`. Read each `Would` line and the table.
3. **Apply.** Run the command in `next`. It has `--plan <token>`. Add `--intent "<label>"`.
4. **Verify.** `mesh check --code BROKEN_LINK`, or `mesh section check <section>`.

## Rename a Document

```bash
mesh rename "<old>" "<new>" --dry-run
```

- The rename rewrites each inbound link (body links, embeds, frontmatter links) in the same transaction.
- **The typed prefix stays.** A leaf name keeps the prefix of the old title: `(Task) 042 X` and `Y` give `(Task) 042 Y`, and `Parent . (Note) X` and `Y` give `Parent . (Note) Y`. A new title with a `(Type)` or a ` . ` is complete. `--full` reads `<new>` as the complete title in every case.
- `--cascade` also renames each `<old> . <child>` document.
- `--across` also heals the references in each reachable peer mesh ([[dev-knw-mesh-cross_mesh]]).
- **The old title stays reserved.** `mesh address list` lists it. `mesh create` and `mesh rename` refuse it (`ADDRESS_RESERVED`, exit 9). `mesh address release "<old>"` frees it after a sweep of the references. A rename back by the same document is allowed.
- A title or case collision exits 9. A duplicate title exits 8: rename one copy with `--path`. Its links stay ambiguous, so they are not rewritten.
- `mesh media rename <old.png> <new.png>` renames a media file and rewrites each link to it.
- [[dev-sk-mesh-rename_document]] gives the steps.

## Move, Archive, and Delete

| Command | Effect |
|---|---|
| `mesh mv "<title>" <folder>` | Move the file to the folder. The title and each link stay. |
| `mesh archive "<title>"` | Move the file into the archive. A section member goes to `Archive/` in its section folder, and it stays a member. Another document goes to the `Archive` row of its type, else to `[folders] archive`. The title and each link stay. `--tag <tag>` adds a tag. `--refs strip-frontmatter` removes the frontmatter links to it. |
| `mesh delete "<title>"` | Delete the file and sweep the links to it. |

- Do not use `mv` to move a document into a section folder or out of one. Use `mesh section move` or `mesh section unassign`. They change the tag and the place together.
- `--refs` of `delete` decides what the links to the title become: `strip-frontmatter` (the default: a frontmatter value that is one link loses its key, and a list item that is one link goes), `keep`, `unlink` (body links also become plain text), or `refuse` (exit 9 while a link exists).
- **Remotes.** A section member stays in the sync set after an archive, so the far copy stays too. A delete of a member of a section with a remote deletes the far copy at the next push. The dry run of the delete does not warn about it: check `mesh remotes` first.
- A deleted number is never used again on this machine (`.mesh/numbers.json`). `.mesh/` does not sync, so this record is local. To keep a number reserved in the mesh itself, archive the document.
- **Archive a finished artifact. Delete only when the task asks for a delete.**

## Links and Tags

| Command | Effect |
|---|---|
| `mesh relink "<from>" "<to>"` | Point each link to `<from>` at `<to>`. Nothing is renamed, and `<from>` need not exist. Anchors and display text stay. |
| `mesh relink "<stem>" "<new stem>" --prefix` | Rewrite a title prefix, for example a shard link stem `sk-onyx-` to `sk-mesh-`. |
| `mesh tag rename <old> <new>` | Rename a tag in each document. `--prefix` renames a namespace (it stops at `/`). `--inline` also renames `#tag` in the bodies. |
| `mesh titles dedupe --map <relpath>=<new title>` | Give one copy of a duplicate title a new title, by its path. No link is rewritten. |
| `mesh ensure-ids [--write]` | List the documents with no id. `--write` adds the ids. |

`--kind body|embed|frontmatter` limits `relink` to one kind of link.

## Sections

A section name has words of letters and digits. Each word starts with a capital letter or a digit. One space separates two words. A name has 1 to 64 characters. The key is the name in lowercase with `-` between the words (`Mesh SDK Docs` gives `mesh-sdk-docs`). A section folder is never inside another section folder.

| Command | Effect |
|---|---|
| `mesh section create "<Name>" [--in <folder>] [--kind open\|closed] [--description <text>]` | Write the folder `(Section) <Name>/` and its header. The parent is `--in`, else `[sections] folder`. |
| `mesh section move "<document>"... --to <section> [--folder <subfolder>]` | Give each document the section tag, in place of any other section tag, and move its file into the section folder. |
| `mesh section unassign "<document>"... [--to <folder>]` | Remove the section tag and move the file out of the section folder. |
| `mesh section rename <section> "<New Name>"` | Rename the folder and the header, give each member the new tag, and point each link to the header at the new title. |
| `mesh section kind <section> [open\|closed]` | Print the kind, or write it. `closed` is refused while crossing links exist. |
| `mesh section merge <from> --into <to>` | Move each member of `<from>` into `<to>`, point the links to the old header at the new header, and delete the old header. |
| `mesh section delete <section> [--unassign] [--refs <policy>]` | Delete the header and the empty folder. `--unassign` first unassigns the members. |
| `mesh section fix [<section>] [--follow-folder]` | Repair `SECTION_FOLDER_MISMATCH`. By default the tag wins: a file with a section tag moves to the folder of its tag, and a file with no section tag in a section folder gets the tag of that folder. With `--follow-folder` the folder wins. |

- In a Flint with the Invironments shard, `flint shard ie section "<Name>"` creates the section and writes the header body from its template. Use it when the header needs its context prose. Then write the prose.
- A move into a closed section is refused when it adds crossing links. The refusal lists them.
- `section fix` can make a file a member: a file with no section tag in a section folder gets the tag. When the section has a remote, the next push publishes that file. Read each `Would tag` line of the dry run before you apply it.
- A move of a member of a section with a remote changes the sync set: the next push sends a new member, and a member that leaves stays on the far side ([[dev-knw-mesh-sync]]).
- For a large change, run `mesh section check`, then `mesh section fix`, then the moves, then `mesh section kind`.
- [[dev-sk-mesh-move_to_section]] gives the steps.

## Groups

A group key and a definition name use the grammar of a section key and a section name.

| Command | Effect |
|---|---|
| `mesh group create "<Name>" [--description <text>]` | Write the definition `(Group) <Name>.md` with its group tag in `[groups] folder`. |
| `mesh group add <group> "<document>"...` | Add the tag `#ie/groups/<key>`. A second add changes nothing. A new valid key starts a new group, with a warning. |
| `mesh group remove <group> "<document>"...` | Remove the group tag. The definition stays. |
| `mesh group rename <group> "<New Name>"` | Rewrite the tag on each member, and rename the definition with its links. |
| `mesh group delete <group> [--refs <policy>]` | Remove the tag from each member and delete the definition. |
