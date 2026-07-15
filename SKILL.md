---
name: tool-call-discipline
description: Load before any non-trivial multi-tool task (3+ calls, file exploration, refactors, batch edits). Enforces right-tool-first-try, parallel dispatch of independent calls, wide read windows, and parameter self-check. Prevents wasted turns from cat/head/tail/find/grep-in-bash, from 30-line slice re-reads, and from fabricated parameters.
---

# Tool Call Discipline

Cheap rules that keep tool calls from wasting turns.

## The Five Hard Rules

1. **Right tool, first try** — never shell out for something a dedicated tool does better.
2. **Parallel independent calls** — single message, multiple tool uses. Sequential is opt-in.
3. **Wide read windows** — ≥ 100 lines when you don't know what you're looking for.
4. **Read before Edit/Write** — the tools enforce it; don't waste a turn on the error.
5. **Parameter sanity 3-check** — absolute path, verbatim field name, no fabricated values.

## Right-Tool Cheat Sheet

| Goal | ❌ Bad (bash) | ✅ Good |
|---|---|---|
| Read a file | `cat foo.md` / `head -100 foo` | `Read({filePath})` |
| Read tail | `tail -50 log` | `Read({filePath, offset: N})` |
| Find files by name/pattern | `find . -name "*.ts"` | `Glob({pattern:"**/*.ts"})` |
| Search file contents | `grep -r "foo" .` | `Grep({pattern:"foo"})` |
| Count matches | — | `bash("rg -c pattern")` (OK) |
| In-place edit | `sed -i s/a/b/g` | `Edit({filePath, oldString, newString, replaceAll:true})` |
| Print variables | `echo "$VAR"` | just say it in your reply |
| Write a file | `cat <<EOF > file` / `echo … >` | `Write({filePath, content})` |
| Change dir + run | `cd X && npm test` | `bash({command:"npm test", workdir:"X"})` |

When in doubt: dedicated tool is faster, deterministic, and doesn't spawn a shell.

## Parallel Dispatch

Independent calls = **one message, N tool uses**.

Good pattern for "check config + list dir + get user":

```
[read config.json] [ls ~/foo] [get_user()]   ← same message
```

Bad pattern (three round-trips of latency):

```
turn 1: read config.json     ← wait
turn 2: ls ~/foo             ← wait
turn 3: get_user()
```

Rule of thumb: if the second call doesn't use the output of the first, batch them.

## Read Windows

- Default `limit` is 2000 — usually fine. Don't manually shrink it.
- Unknown file: start with a **≥ 100-line window** from offset 1.
- Big file (> 2000 lines): use `Grep` first to locate, then `Read` with a target `offset`.
- Never do `Read(offset:1,limit:30) → Read(offset:31,limit:30) → …`. Widen instead.

## Parameter Sanity 3-Check

Before firing any tool call, verify:

1. **Absolute paths** — `/Users/...` or `~/...` expanded. Not `./foo` when the tool wants absolute.
2. **Field names verbatim** — copied from official docs, not guessed from memory. Especially:
   - `embeddingDimensions` (plural — singular fails silently → 768)
   - Yuque `repo_id` accepts `login/slug` OR numeric id, not one arbitrary format
   - Figma `nodeId` uses `123:456` or `123-456`
3. **No fabricated values** — if you don't know user login, repo id, port, or model name → probe first (`yuque_get_user`, `ls`, `curl :port/health`).

## Failure Handling

- **Read the actual error.** Don't guess. `404` ≠ `permission denied` ≠ `no such table`.
- **Fix that specific thing.** Retry ≤ 1 time with the fix.
- **Don't tool-swap speculatively.** If `Grep` fails, `Grep` again with a fixed pattern — don't jump to `bash rg`.

## Anti-Patterns

- Reading a 500-line file in 15 × 30-line chunks.
- Firing `bash "cat file.md"` when `Read` exists.
- Calling `find` from bash when `Glob` exists.
- Sequential `Read` × 5 for unrelated files (should be one message, five reads).
- Passing `filePath: "AGENTS.md"` (relative) — will fail; use absolute path.
- Retrying the same failing call with no change ("maybe it works this time").
