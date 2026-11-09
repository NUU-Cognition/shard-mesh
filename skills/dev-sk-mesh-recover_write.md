---
description: "Find out what happened to a failed, refused, or lost mesh write, and settle it with no data loss"
---

> [!important] THIS FILE IS AN INSTRUCTION. WHEN REFERENCED IT IS MEANT TO BE TAKEN AS AN ACTION.

Run `flint shard start mesh` if you haven't already.

# Skill: Recover a Write

Learn the true state of a write before you repeat it.

# Input

- The command that failed, or that gave no result.
- (Optional) Its transaction id.
- (Optional) Its exit code and its `code`.

# Actions

1. **Find the receipt.** With a transaction id, run `mesh txn show <id>`. Without one, run `mesh txn list --limit 10` and find the transaction by its time and its intent. Then run `mesh txn show <id>` and confirm its paths.
2. **Act on the outcome.**
   - `committed`: the write is done. Do not repeat it. Verify the documents with `mesh get`.
   - `not_applied` or `rolled_back`: nothing was written. Make a new dry run of the command, then apply it.
   - A pending transaction (an intent and no receipt): run `mesh txn recover`. Then run `mesh txn show <id>` again, and act on its outcome. Do not repeat the write before that.
   - `recovery_required` (exit 7): the recovery found bytes that belong to no state of the transaction. It kept them and the backups in `.mesh/txn/kept/`. Stop. Report the paths and the hashes from `mesh txn show <id>`, and ask the person.
3. **A lock timeout** (exit 7, `LOCK_TIMEOUT`): run `mesh lock status`.
   - The owner is alive: another write runs. Wait, then try again. A longer `--lock-timeout <ms>` is acceptable.
   - The owner is dead on this host: the next write takes over the lock by itself. Try again.
   - The owner is unreadable, or on another host: stop and report it. Run `mesh lock break --owner <entry> --force` only when the person confirms that the owner does not write.
4. **A stale plan** (exit 7 after `--plan`): the mesh changed after the dry run. Make a new dry run and read it again.
5. **A failed guard** (exit 7 after `--expect-hash`): the file changed. Read it again with `mesh get --json`, and decide again with the new content.
6. **An unfinished split** (`UNFINISHED_SPLIT`): run `mesh split status`, then follow its `next` command (`mesh split recover <split-id>`).
7. **Report.** Give the command, the outcome, what you did, and the state now.

# Output

- The true outcome of the write.
- The write completed once, or a report that a person must decide.
