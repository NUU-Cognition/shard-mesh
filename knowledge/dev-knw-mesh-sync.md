---
description: "Sync a section with a remote (onyx: or fs:): remote rows, the sync set, pull, push, sync, conflicts, guards, unpublish, clone, and old tag-scoped remotes"
---

# Knowledge: Section Sync

A remote connects one local section to one far side. A section can have at most one remote. The same commands work for each kind of far side:

| URL | Far side |
|---|---|
| `onyx:<mesh-id>` | An Onyx mesh on the Onyx server. It needs the NUU account login ([[dev-knw-mesh-onyx]]). |
| `fs:<path>` | Another mesh folder on disk |

## Remote Row

```toml
[remotes.field-notes]
url = "onyx:11111111-1111-4111-8111-111111111111"
meshId = "11111111-1111-4111-8111-111111111111"
section = "field-notes"
```

`mesh remotes` lists each remote: the name, the URL, the section key, the kind and member count of the local section, and the remote version of the last sync.

A first `mesh pull` or `mesh push` with a URL (`--remote onyx:<mesh-id>`) and `--section <section>` writes a new row. The row name is the section key. `--name <remote-name>` gives another name. The command checks the name, the URL, the mesh id, and the section before it writes `mesh.toml`. A refusal writes no row.

## Commands

| Command | Effect |
|---|---|
| `mesh pull --remote <name\|url>` | Bring the far changes into the local section by a three-way merge with the last sync. |
| `mesh push --remote <name\|url>` | Send the local changes of the section since the last sync. |
| `mesh sync [--remote <name\|url>]` | Pull, then push when the pull leaves nothing unresolved. With no `--remote`, sync each remote of `mesh.toml` in order. |
| `mesh conflicts --json` | List the local documents with conflict markers. |
| `mesh resolve "<title>" --ours\|--theirs\|--merged` | Clear the markers in one document. |
| `mesh remotes` | List the remotes. |

- `--dry-run` reports what a pull, push, or sync would do, and writes nothing. When the pull would change local files, a sync dry run cannot plan the push.
- `--on-conflict` sets the policy of a pull: `stop` (the default: write conflict markers), `keep-local`, `take-remote`, or `copy`.
- **Preview each push.** `mesh push --remote <name> --dry-run` shows each far action, the far deletes included: a local member that you deleted is deleted on the far side. `mesh sync` pushes with no preview of the push.
- `--force` on a push or a sync writes the local state of the section over the far side. It can delete far edits. Use it only with permission. A first push to an empty far side does not need it.
- `--prune` is refused on every section remote.
- `mesh conflicts` without `--json` opens a screen for a person. With no terminal and no `--json`, it refuses with exit 2. An agent always adds `--json`.

## Sync Set

The sync set holds each section member, the section header included, and the media that the members embed.

The far side never stores the section tag. A push removes the tag before it calculates the hashes and sends a document. A pull adds the tag to each local document that it writes. All other frontmatter travels with no change.

A local document keeps its path. A new far document is written at the root of the section folder.

## Membership Changes

| Event | Result |
|---|---|
| `mesh section move "<document>" --to <section>` adds a document | It becomes a member. The next push sends it. |
| The same command moves a member to another section | It leaves this sync set. The far copy stays. |
| `mesh section unassign "<document>"` | The local document stays. The far copy stays. |
| A local member file is deleted | The next push deletes the far copy. |
| A far document is deleted | The next pull detaches the local document and keeps its file. |
| A detached document joins again after both sides changed | Sync reports a `no-base` conflict. |
| `mesh section unassign "<document>" --delete-remote` | The far copy is deleted. The local document stays, with no section. |

The last row is the only explicit unpublish action. [[dev-sk-mesh-unpublish_document]] gives the steps.

## Guards

- **Unpublish guard.** `--delete-remote` compares the far content hash and the far frontmatter hash with the base entry. When the far document changed after the last sync, or has no base entry, the command refuses with exit 7, writes nothing, and gives `mesh pull` as next. `--force` deletes the far document anyway. When no far copy exists, the command refuses with exit 9 and gives `mesh section unassign "<document>"` as next.
- **Section guards.** Pull and push refuse with exit 9 when a member has two section tags, when a member is outside the section folder, or when two members have one title. Run `mesh section check <section>`, then `mesh section fix <section>` when the repair plan is correct.
- **Closed sections.** A push of a closed section refuses when a member links to a document outside the section. A push of an open section lists each crossing link that will not resolve on the far side.
- **Drift.** A push refuses when the far side changed after the last sync (`REMOTE_DRIFT`, exit 7). Run `mesh pull` first.
- **Markers.** A push refuses when a local file has conflict markers (`UNRESOLVED_MARKERS`, exit 7). Run `mesh conflicts --json`.
- **Sync.** `mesh sync` skips the push when the pull leaves a conflict (`PUSH_SKIPPED`, exit 7). With many remotes, each remote runs, and the command exits with the code of the first failure (`REMOTE_FAILED`).

## Far Section Tags

A pull does not write a far document that has a section tag. It writes the other documents, and it reports the refused document as a `MULTIPLE_SECTIONS` conflict. Remove the tag from the document on the far side: in Onyx for an `onyx:` remote, or with the command in `next` for an `fs:` remote (`mesh section unassign "<document>" --root <far root>`). Then pull again. The pull never removes that far tag by itself.

## Conflicts and the Base

A pull with the policy `stop` writes git-style markers into each local file with a conflict, and exits 7 (`UNRESOLVED_CONFLICTS`). Each marker line starts with a backslash (`\<<<<<<< local`, `\=======`, `\>>>>>>> <remote>`), so that Markdown shows it as text. Do not search for the markers by hand: use `mesh conflicts --json`. Then:

1. `mesh conflicts --json` lists the files.
2. For each file, `mesh resolve "<title>" --ours` keeps the local side, and `--theirs` keeps the far side.
3. For a hand merge, edit the file to remove every marker, then run `mesh resolve "<title>" --merged`.
4. Push only when no marker remains.

The `.mesh/` folder holds the sync base and its stored objects. Do not edit it.

## Clone

Clone a far mesh into a section that does not exist locally:

```bash
mesh pull --remote onyx:<mesh-id> --section "<Name>" --dry-run
mesh pull --remote onyx:<mesh-id> --section "<Name>"
```

The pull creates the section folder under `[sections] folder` and writes the remote row only after every check passes. It adds the section tag to each local document. When the far side has no section header, the clone creates one locally, and the next push sends it. When the far side has a header with another key, the clone refuses with exit 9. [[dev-sk-mesh-clone_mesh]] gives the steps.

## Old Tag-Scoped Remotes

An old Onyx remote row had `tag` and `folder` keys in place of `section`, and its documents had a tag `onyx/<old-key>`. While such a row is in `mesh.toml`, every `mesh` command refuses with exit 9 (`uses retired field "tag"`). Do not run a `mesh` command before step 2. To move the row to a section remote, and keep the existing Onyx mesh:

1. Read the old row directly in `mesh.toml`. Record the name, the URL, the mesh id, `tag`, and `folder`.
2. Edit the row by hand. This is the one permitted hand edit of `[remotes]`. Remove `tag` and `folder`, and add `section = "<section-key>"`. Keep the name, the URL, and the mesh id.
3. Run `mesh list --tag onyx/<old-key>`. Record each document.
4. Run `mesh section create "<Section Name>" --in "<parent-folder>"`. The parent folder is relative to the mesh root, for example `--in "Mesh/Onyx"`. Stop if the command refuses the section.
5. Write one JSON Lines file: one `tag.remove` request for each document, then one `section.move` request for all documents. The last request moves the documents and adds the new section tag. Run `mesh batch --atomic --dry-run --file "<file>"`, read each line, then run `mesh batch --atomic --file "<file>"`.
6. Remove the old folder when it is empty.
7. Run `mesh pull --remote <name>`. Stop if it reports a conflict, and push only when each conflict is settled. Then run `mesh push --remote <name> --dry-run`, read the plan, and run `mesh push --remote <name>`.
8. Run `mesh remotes` and `mesh section check "<section-key>"`. Confirm that the existing remote names the section and that no section error remains.

Do not use the publish or clone skills for this change: they make a new Onyx mesh.
