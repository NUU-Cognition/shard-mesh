---
description: "Apply several different mesh writes as one all-or-nothing transaction with mesh batch --atomic"
---

> [!important] THIS FILE IS AN INSTRUCTION. WHEN REFERENCED IT IS MEANT TO BE TAKEN AS AN ACTION.

Run `flint shard start mesh` if you haven't already.

# Skill: Atomic Batch

Run different `mesh` writes as one transaction. When one request fails, nothing is written.

# Input

- The list of writes: each one is a `mesh` command with its arguments.
- An intent label.

# Actions

1. **Write the requests.** Write one JSON Lines file in a temporary folder outside the mesh. One line for each request:
   ```json
   {"id":"<id>","cmd":"<command name>","args":{"_":["<positional>"],"<long-option>":"<value>"}}
   ```
   `cmd` is the command name with `.` between the words (`props.set`, `tag.remove`, `section.move`). `"_"` holds the positional arguments in order. A boolean option is `true`. Give each request a clear `id`.
2. **Put the requests in a valid order.**
   - A request that needs the full index (`list`, `check`, `rename`, `delete`, a bulk edit) comes before the first write.
   - A write that moves files (`mv`, `archive`, `section.move`, `section.unassign`, `section.rename`, `section.merge`, `section.delete`, `section.fix`) is the last write.
   - A media operation is the last request.
   - Do not put `pull`, `push`, `sync`, `resolve`, `index rebuild`, `index clear`, `split`, `split recover`, or `txn recover` in the batch.
3. **Dry run.** Run `mesh batch --atomic --dry-run --file "<file>" --json`. Read each line of `results`. Each request must plan with `exitCode` 0.
4. **Correct a refusal.** Exit 2 (`BATCH_INVALID`) means a malformed line, an unknown command, or a usage error: correct the file. A refusal of one request names it with its `id`. Correct it and repeat step 3.
5. **Apply.** Run `mesh batch --atomic --file "<file>" --intent "<label>" --json`. Record the `transactionId`.
6. **Handle a failure.** When a request fails, nothing was written: the earlier requests show `NOT_APPLIED` and the later ones `SKIPPED`. Correct the cause and repeat from step 3. When the commit fails, read the receipt with `mesh txn show <transactionId>`.
7. **Verify.** Read the changed documents with `mesh get` or `mesh list`.
8. **Clean up.** Delete the temporary file.
9. **Report.** Give each request id, its result, and the transaction id.

# Output

- Every write of the batch is applied in one transaction, or none is.
- A report with the transaction id.
