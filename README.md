# opencode-config

Personal OpenCode configuration — global behavior rules (Karpathy-style) + context-mode MCP routing rules.

## What's inside

| File | Purpose |
|------|---------|
| `AGENTS.md` | Main config: engineering behavior rules + context-mode routing rules |

## Structure of AGENTS.md

`AGENTS.md` is split into two parts:

### Part 1 — Engineering Behavior Rules (Karpathy-style)

Inspired by [Andrej Karpathy's observations](https://x.com/karpathy) on LLM coding pitfalls:

- **Think before coding** — clarify ambiguity, state assumptions, propose simpler alternatives
- **Simplicity first** — minimal solution, no speculative abstractions or future-proofing
- **Surgical changes only** — minimum diff, match existing style, no unrelated refactors
- **Goal-driven execution** — define success criteria, verify before claiming done

### Part 2 — context-mode Routing Rules

Based on [context-mode](https://github.com/mksglu/context-mode) MCP plugin. Protects context window from flooding by routing high-output commands through sandboxed tools.

- Blocks `curl`, `wget`, inline HTTP in shell
- Redirects large-output shell commands to `context-mode_ctx_batch_execute`
- Routes file analysis through `context-mode_ctx_execute_file`
- Routes web fetching through `context-mode_ctx_fetch_and_index`

## Usage

### Global (recommended for personal use)

Copy `AGENTS.md` to your OpenCode global config directory:

```bash
mkdir -p ~/.config/opencode
cp AGENTS.md ~/.config/opencode/AGENTS.md
```

This applies the rules to all projects by default.

### Per-project

Copy `AGENTS.md` to your project root and fill in the **Repository-specific section** at the bottom:

```bash
cp AGENTS.md /path/to/your/project/AGENTS.md
```

Then edit the bottom section:

```md
## Repository-specific section
- Build command: pnpm build
- Dev command: pnpm dev
- Test command: pnpm test
- Lint command: pnpm lint
- Important directories: src/app, src/components
- Architecture notes: Next.js App Router + TypeScript
- Project conventions: ...
- Known gotchas: ...
```

> Project-level `AGENTS.md` takes precedence over the global one in OpenCode.

## Credits

- Behavior rules inspired by [Andrej Karpathy](https://x.com/karpathy) and [forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills)
- Context routing rules from [mksglu/context-mode](https://github.com/mksglu/context-mode)
