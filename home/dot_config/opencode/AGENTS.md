# Global agent rules

These rules apply to **all** OpenCode sessions, across every project.

---

## Two-level rule system

OpenCode loads rules from two locations:

| Level | File | Scope |
|-------|------|-------|
| **Global** | `~/.config/opencode/AGENTS.md` | Every session, every project |
| **Project** | `AGENTS.md` in the project root (or nearest ancestor) | That project only |

Use `~/.config/opencode/AGENTS.md` for rules that should apply universally, personal preferences, workflow conventions, and cross-cutting principles.

Use a project-level `AGENTS.md` for rules specific to that codebase, language conventions, repo layout, build commands, and other project-only conventions.

---

## Never guess schemas or APIs

When working with any configuration schema (Kubernetes, Talos, Helm values, etc.), always read the authoritative source before writing config. Acceptable sources, in order of preference:

1. The actual Go struct definitions or JSON schema from the project source (clone the repo if needed)
2. The official rendered documentation
3. The project's own example files in the same repo

Never trust: blog posts, AI-generated examples, or partial memory of a schema. A wrong key name wastes more time than the lookup takes. This applies even when you are fairly confident -- verify before writing.

If a documentation site does not render usefully via WebFetch, clone the source repo and read the struct tags directly.

For SDK calling conventions (method signatures, argument shapes, etc.), prefer real call sites in existing plugins or examples over generated type declarations. Generated types may reflect an internal calling convention that differs from what external callers use. For OpenCode plugins, read concrete plugin implementations in the source repo (e.g. `packages/slack/src/index.ts`, `packages/github/src/index.ts`) rather than generated SDK type files. When the docs or types are ambiguous, a working plugin in the same repo is the ground truth.

---

## Continuous maintenance (meta-rule)

- After every substantive conversation, review whether `~/.config/opencode/AGENTS.md` or the relevant project `AGENTS.md` needs updating.
- Add a rule when a user corrects agent behavior or establishes a new pattern — write it down so the next session benefits.
- Never remove rules without explicit user confirmation.
- Never silently adopt a convention that isn't written down.
- Keep `~/.config/opencode/AGENTS.md` concise. If it grows unwieldy, propose splitting into topic files.

---

## Subagents

Four specialised subagents handle common delegated tasks. Use them instead of doing these tasks inline.

| Subagent | When to use |
|----------|-------------|
| `@memory` | Persisting rules, conventions, or lessons: updates to global memory (`AGENTS.md`) or local memory (project `AGENTS.md`) |
| `@dotfiles` | Any change to chezmoi-managed files: adding, modifying, or removing config files, scripts, or templates |
| `@daily-note` | Logging achievements or action items to today's Obsidian daily note |
| `@beads-task-agent` | Any beads operation requiring 2+ `bd` commands: status overviews, finding and completing ready work, exploring the issue graph, multi-issue sequences |

When subagent delegations are needed at session close (config changes, memory updates, daily note logging), run them in sequence rather than inline: delegate each concern to its subagent one at a time, wait for each to complete, then proceed to the next. This keeps the primary agent's context clean and each concern isolated.

---

## Custom commands and subtask isolation

- Treat custom commands with `subtask: true` as isolated subagent executions, not as continuations of the current session.
- If a command depends on the current session context, conversation state, or an in-progress retrospective, do not configure it with `subtask: true`.
- Prefer running retrospectives, reviews, and other session-aware commands in the current session. Use `subtask: true` only when isolation matters more than shared context.

---

## Global memory and local memory

The user refers to `~/.config/opencode/AGENTS.md` as **global memory**. When asked to "commit something to global memory", delegate to the `@memory` subagent.

The user refers to the `AGENTS.md` in the current project root, or nearest ancestor, as **local memory**. When asked to "commit something to local memory", delegate to the `@memory` subagent.

Never edit `~/.config/opencode/AGENTS.md` directly, it is managed by chezmoi and will be overwritten. The `@memory` subagent knows this and handles it correctly.

The same chezmoi rule applies to all dotfiles under `~/.config/`, always edit the source in `~/.local/share/chezmoi/` and apply from there. Use the `@dotfiles` subagent for this.

For rules specific to the chezmoi dotfiles repo itself, the target file is `~/.local/share/chezmoi/AGENTS.md`; tell the `@memory` subagent this explicitly.

When `@memory` updates any memory file, it must review nearby rules in that file for contradictions, duplication, and scope conflicts.

When contradictions, duplication, or scope conflicts are found, reconcile them in the same edit instead of appending another overlapping rule.

Choose the narrowest correct scope for each rule: keep cross-cutting behavior in global memory, and keep project-specific conventions in local memory.

If a rule in global memory is actually project-specific, move it to the relevant local memory file and remove or narrow the global version in the same change. Do not leave the project-specific rule behind in global memory.

---

## Writing style

- Never use em dashes (---, &mdash;, or the Unicode character —). Use a comma, colon, or restructure the sentence instead. This applies at the point of writing — do not generate em dashes and fix them later. Treat this as a hard constraint during composition, not a post-hoc linting step.

---

## Daily note logging

When something is achieved during a session (a task completed, a ticket updated, a document published, etc.), ask the user first whether they want it logged to their daily note. Only delegate to the `@daily-note` subagent after the user confirms. Never log proactively or assume the user wants a log entry, even at session end.

Daily note conventions:
- Do not add an H1 heading with the date; the filename already conveys it.
- Do not use daily notes as a task tracker. Only log what has been completed.
- Group entries under H2 headings that link to the relevant project note (e.g. `## [[Projects/foo/Bar|Bar]]`).
- Keep entries as plain bullet points describing accomplishments.

---

## Post-task retrospective and self-improvement

After completing any non-trivial task, perform a brief critical retrospective before closing out:

1. **What went right:** note any approaches or tools that worked well.
2. **What went wrong:** identify mistakes, incorrect assumptions, wasted steps, or anything that required correction.
3. **Actionable lessons:** for each thing that went wrong, delegate to `@memory` to review global memory. Prefer strengthening or clarifying broadly applicable rules over adding incident-specific ones. The acceptable outcomes are: add a new rule, strengthen or clarify the wording of an existing rule, or add a brief note confirming an existing rule already covers the lesson. "No new rule needed" is NOT an acceptable reason to skip delegating. The delegation must happen; the outcome of that delegation may be a no-op, but the review itself is mandatory.

The retrospective does not need to be verbose. A single sentence per point is enough. The goal is that each session leaves global memory slightly better than it found it, by making it more accurate and reusable, not more tied to one incident. Silent self-improvement is acceptable; only surface the retrospective to the user if a rule was added or if the user would benefit from knowing.

When turning a concrete incident into a persisted lesson:

- Use the incident as evidence, but do not make the incident itself the rule.
- Write the rule at the highest useful level of abstraction that would prevent the same class of mistake elsewhere.
- Do not generalise so far that the rule becomes vague or non-actionable.

If a retrospective, subtask, or delegated review identifies a lesson that should be persisted, the primary agent remains responsible for ensuring the `@memory` delegation actually happens before the loop is considered complete. Reporting that memory still needs updating does not satisfy the requirement. If a subtask cannot perform the update itself, resume control and perform the delegation explicitly.

This rule applies retroactively: if a session ends without a retrospective, perform one before the final response.

When `chezmoi apply` triggers a brew bundle run (via an onchange script), treat it as fire-and-forget. Once the apply itself succeeds (config files deployed), do not wait on the brew bundle background process or treat a bash timeout as an error.

---

## Problem-solving attitude

Never suggest the user accept a limitation, take a shortcut, or move on when a proper solution may exist. Keep investigating until the problem is actually solved. Suggesting workarounds as a final answer is not acceptable.

---

## Browsing code in git repositories

When you need to read source code from a public or private git repository, clone it locally with `git clone` first. Do not curl GitHub API endpoints, fetch raw file URLs, or use the GitHub web UI to browse code. Cloning is faster, more reliable, and gives full access to the entire codebase. This applies even for a single file lookup.

---

## Toolchain execution policy (mise required)

When a runtime or CLI can be managed by `mise`, agents must run it through `mise`.

- Prefer `mise x -- <command>` (or `mise exec ... -- <command>`) for ad hoc commands.
- Use `mise run <task>` for project tasks when available.
- Do not call managed toolchains directly when a `mise` invocation is possible.

## Python package management (mise + uv required)

Python workflows must use `mise` and `uv` together.

- Never run `python`, `python3`, `pip`, `pip3`, or raw `uv` directly.
- Run Python commands as `mise x -- uv ...` (or `mise exec ... -- uv ...`).
- Install project dependencies with `mise x -- uv add <package>`.
- Install global Python CLI tools with `mise x -- uv tool install <package>`.
- Run scripts with dependencies using `mise x -- uv run --with <package> <script-or-command>`.
- For one-shot Python commands outside a uv project, use `mise x -- uv run --no-project python ...`.
- Before project-scoped uv commands, ensure the directory is bootstrapped by uv (`pyproject.toml` and `.python-version`). If missing, run `mise x -- uv init .` first.
- Never use `--break-system-packages`.

For uv projects, rely on global mise config to auto source or create the project `.venv` via `python.uv_venv_auto = "create|source"`.

---

## Python guard (shell.env shims)

When OpenCode spawns a shell, it sets `OPENCODE=1` natively. The `.zshenv` PATH array detects this and conditionally prepends `~/.local/share/opencode-shims`, which contains shims for `python`, `python3`, `pip`, `pip3`, `uv`, and `uvx`. Each shim checks `$_` to see if it was invoked via `mise`; if not, it blocks with an error.

- Do not prefix commands with `wolf` or any other wrapper. Shims are injected via PATH automatically.
- Do not modify files in `~/.local/share/opencode-shims/`.

---

## Commit discipline

Before committing, run `git diff --staged` and make an objective assessment of whether the change is both atomic and in-scope based on the context of the request. Do not commit unrelated changes that happened to be modified in the working tree.

Commit messages use semantic commit format: `type(scope): description`. Types: `feat`, `fix`, `chore`. Scope is optional but use it when relevant. Always check `git log --oneline` before committing to match the repo's existing style.

---

## Directory access behavior

- Do not run proactive directory existence checks before path operations (for example, `ls /tmp` before writing to `/tmp`).
- Attempt the intended operation first.
- If it fails because a directory is missing, create the directory and retry.
- Only do a pre-check when a tool explicitly requires it.

---

## Shell command granularity

- When a task crosses a policy boundary or mixes concerns, prefer the smallest atomic command that completes the immediate next step.
- Do not chain filesystem cleanup, chezmoi source edits, and git operations into one shell command.
- If cleanup is needed outside the source tree, perform that cleanup as a separate step, then run repo or git commands separately.
- Keep source edits, deployed-target cleanup, `chezmoi apply`, and git commit or push as separate commands unless a later step strictly depends on the previous one and stays within the same concern.
- Keep destructive or policy-sensitive shell actions isolated so the intent is easy to inspect before execution.

---

## Issue Tracking

This project uses **bd (beads)** for issue tracking.
Run `bd prime` for workflow context, or install hooks (`bd hooks install`) for auto-injection.

**Quick reference:**
- `bd ready` - Find unblocked work
- `bd create "Title" --type task --priority 2` - Create issue
- `bd close <id>` - Complete work
- `bd dolt push` - Push beads to remote

For full workflow details: `bd prime`
