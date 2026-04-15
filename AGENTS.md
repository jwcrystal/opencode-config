# AGENTS.md — Personal OpenCode Configuration

> Two-layer design:
> - **Part 1 — Engineering behavior rules** (Karpathy-style): govern *how* the agent codes
> - **Part 2 — context-mode routing rules**: govern *which tools* the agent uses to protect context window

---

## Part 1 — Engineering Behavior Rules

### 1. Think before coding

- Do not guess unclear requirements.
- If the task is ambiguous, list the ambiguity and ask a targeted clarifying question before changing code.
- State important assumptions explicitly before implementation when they could affect architecture, behavior, or data integrity.
- If there is a simpler solution than the requested direction, propose it before implementing.
- When confused about the codebase or requirements, stop and say clearly what is unclear.

### 2. Simplicity first

- Prefer the simplest solution that fully solves the current problem.
- Do not add abstractions, configuration, extensibility, or future-proofing unless the repository already uses that pattern or the task explicitly requires it.
- Avoid speculative generalization — do not build for hypothetical future use cases.
- Prefer fewer moving parts, fewer files, and less code when quality is equal.
- Do not implement features not mentioned in the request.

### 3. Surgical changes only

- Make the minimum necessary change to satisfy the request.
- Do not perform unrelated refactors, renames, or reformats outside the scope of the task.
- Match the existing code style, architecture, naming, and file organization of the repository.
- If your change creates dead code, broken imports, or stale references, clean up only what your change caused.
- For pre-existing dead code, flag it with a comment — do not delete it without explicit request.

### 4. Goal-driven execution

- Before editing, identify the success criteria in concrete, observable terms.
- When debugging, follow this sequence:
  1. Reproduce the issue
  2. Inspect the relevant code path
  3. Form a hypothesis
  4. Apply the smallest fix
  5. Verify the fix
  6. Check for regressions
- When implementing a feature, define what "done" means in observable behavior before writing code.

### Coding preferences

- Read existing code before writing new code.
- Reuse existing utilities, helpers, and patterns before introducing new ones.
- Preserve public APIs unless the task explicitly requests an API change.
- Favor explicitness over cleverness.
- Keep functions and modules focused with no hidden side effects.

### Communication style

- Be concise and concrete.
- Before meaningful edits, briefly state: what the task is, what you plan to change, and any assumptions or risks.
- If confidence is low, say so clearly.
- If multiple valid approaches exist, present the trade-off briefly and recommend one.

### Guardrails

- Never fabricate files, commands, APIs, or test results.
- Never claim to have run a command if you did not run it.
- Never make unrelated cleanup edits during a scoped task.

---

## Part 2 — context-mode Routing Rules (MANDATORY)

You have context-mode MCP tools available. These rules are NOT optional — they protect your context window from flooding. A single unrouted command can dump 56 KB into context and waste the entire session.

### Think in Code — MANDATORY

When you need to analyze, count, filter, compare, search, parse, transform, or process data: **write code** that does the work via `context-mode_ctx_execute(language, code)` and `console.log()` only the answer. Do NOT read raw data into context to process mentally. Your role is to PROGRAM the analysis, not to COMPUTE it. Write robust, pure JavaScript — no npm dependencies, only Node.js built-ins (`fs`, `path`, `child_process`). Always use `try/catch`, handle `null`/`undefined`, and ensure compatibility with both Node.js and Bun. One script replaces ten tool calls and saves 100x context.

### BLOCKED commands — do NOT attempt these

#### curl / wget — BLOCKED
Any shell command containing `curl` or `wget` will be intercepted and blocked by the context-mode plugin. Do NOT retry.
Instead use:
- `context-mode_ctx_fetch_and_index(url, source)` to fetch and index web pages
- `context-mode_ctx_execute(language: "javascript", code: "const r = await fetch(...)")` to run HTTP calls in sandbox

#### Inline HTTP — BLOCKED
Any shell command containing `fetch('http`, `requests.get(`, `requests.post(`, `http.get(`, or `http.request(` will be intercepted and blocked. Do NOT retry with shell.
Instead use:
- `context-mode_ctx_execute(language, code)` to run HTTP calls in sandbox — only stdout enters context

#### Direct web fetching — BLOCKED
Do NOT use any direct URL fetching tool. Use the sandbox equivalent.
Instead use:
- `context-mode_ctx_fetch_and_index(url, source)` then `context-mode_ctx_search(queries)` to query the indexed content

### REDIRECTED tools — use sandbox equivalents

#### Shell (>20 lines output)
Shell is ONLY for: `git`, `mkdir`, `rm`, `mv`, `cd`, `ls`, `npm install`, `pip install`, and other short-output commands.
For everything else, use:
- `context-mode_ctx_batch_execute(commands, queries)` — run multiple commands + search in ONE call
- `context-mode_ctx_execute(language: "shell", code: "...")` — run in sandbox, only stdout enters context

#### File reading (for analysis)
If you are reading a file to **edit** it → reading is correct (edit needs content in context).
If you are reading to **analyze, explore, or summarize** → use `context-mode_ctx_execute_file(path, language, code)` instead. Only your printed summary enters context.

#### grep / search (large results)
Search results can flood context. Use `context-mode_ctx_execute(language: "shell", code: "grep ...")` to run searches in sandbox. Only your printed summary enters context.

### Tool selection hierarchy

1. **GATHER**: `context-mode_ctx_batch_execute(commands, queries)` — Primary tool. Runs all commands, auto-indexes output, returns search results. ONE call replaces 30+ individual calls.
2. **FOLLOW-UP**: `context-mode_ctx_search(queries: ["q1", "q2", ...])` — Query indexed content. Pass ALL questions as array in ONE call.
3. **PROCESSING**: `context-mode_ctx_execute(language, code)` | `context-mode_ctx_execute_file(path, language, code)` — Sandbox execution. Only stdout enters context.
4. **WEB**: `context-mode_ctx_fetch_and_index(url, source)` then `context-mode_ctx_search(queries)` — Fetch, chunk, index, query. Raw HTML never enters context.
5. **INDEX**: `context-mode_ctx_index(content, source)` — Store content in FTS5 knowledge base for later search.

### Output constraints

- Keep responses under 500 words.
- Write artifacts (code, configs, PRDs) to FILES — never return them as inline text. Return only: file path + 1-line description.
- When indexing content, use descriptive source labels so others can `search(source: "label")` later.

### ctx commands

| Command | Action |
|---------|--------|
| `ctx stats` | Call the `stats` MCP tool and display the full output verbatim |
| `ctx doctor` | Call the `doctor` MCP tool, run the returned shell command, display as checklist |
| `ctx upgrade` | Call the `upgrade` MCP tool, run the returned shell command, display as checklist |
| `ctx purge` | Call the `purge` MCP tool with confirm: true. Warns before wiping the knowledge base. |

After /clear or /compact: knowledge base and session stats are preserved. Use `ctx purge` if you want to start fresh.

---

## Repository-specific section

<!-- Fill this in for each project -->
- Build command:
- Dev command:
- Test command:
- Lint command:
- Important directories:
- Architecture notes:
- Project conventions:
- Known gotchas:
