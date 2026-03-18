---
name: rebase-upstream
description: Rebase this chezmoi dotfiles repo onto upstream while preserving local preferences, even when upstream refactors the affected behavior into new files, commands, skills, or subagents.
---

# Rebase Upstream

Use this skill when rebasing this dotfiles repo onto `upstream/main` or another upstream target.

The goal is to preserve local intent, not just whichever side wins a textual conflict.

If arguments were provided in the calling context, treat them as overrides for the upstream remote, branch, or extra audit focus areas. Otherwise default to `upstream/main`.

## Workflow

### 1. Establish the safety baseline

- Confirm remotes, current branch, working tree status, and recent history.
- Identify the upstream target, default `upstream/main`.
- Create a safety tag before changing history, for example `pre-rebase` on the current `HEAD`.
- If the working tree is dirty, preserve it safely before rebasing. Do not discard or overwrite local changes. Record exactly what was preserved so it can be reapplied and audited later.

### 2. Build a local intent map before rebasing

Do not rely on conflict markers alone. First identify what local-only behavior must survive the rebase.

- Inspect local-only commits relative to upstream.
- Inspect uncommitted changes too, if any exist.
- For each local change, write a compact intent map: what behavior changed, why it exists, and which downstream files currently express that behavior.
- When a local preference appears in one file, search related files for the same concept before rebasing. In this repo that especially includes `AGENTS.md`, `.opencode/skills/**/*.md`, `home/dot_config/opencode/AGENTS.md`, `home/dot_config/opencode/agents/*.md`, `home/dot_config/opencode/commands/*.md`, and any touched config templates.

Treat behavioral preferences as higher priority than file layout. If upstream moved a rule into a subagent, command, or skill, preserve the preference in the new canonical location too.

Example: if the user prefers daily notes to be created when missing, and upstream moved daily note behavior from global memory into `home/dot_config/opencode/agents/daily-note.md`, keep that create-if-missing behavior aligned in the subagent after the rebase. Do not silently keep the old preference only in an obsolete location.

### 3. Rebase onto upstream

- Fetch the upstream remote.
- Rebase local commits onto the chosen upstream branch.
- For git commands that may open an editor, set `GIT_EDITOR="true"`.

### 4. Resolve conflicts by preserving semantics

When conflicts occur:

- Read both sides carefully and identify the behavioral difference, not just the textual difference.
- Keep upstream structural improvements when they are compatible.
- Reapply local preferences into the new structure when upstream has renamed, split, or relocated the relevant logic.
- Never resolve by taking upstream wholesale if that drops a known local preference.
- Never resolve by taking local wholesale if that discards useful upstream restructuring.

If a conflict touches rules, commands, subagents, or skills, audit adjacent files for moved ownership of the same concept before continuing.

### 5. Post-rebase semantic audit

Before considering the rebase complete, verify that local intent survived in the right places.

- Compare `pre-rebase..HEAD` and review the full semantic diff.
- Re-check every item in the local intent map.
- Search for each affected concept across the repo, especially when upstream introduced new subagents, commands, skills, or config files.
- If a user preference is now expressed in a different canonical file than before, update that canonical file so downstream behavior matches the user's preference.
- Reapply any preserved working-tree changes and audit them again after replay.

The audit is not complete until local preferences are aligned across the new file layout.

### 6. Apply and verify dotfile changes

- If the rebase changed chezmoi-managed sources, run `chezmoi apply`.
- If a full `chezmoi apply` fails because unrelated bootstrap scripts hit interactive Homebrew or system-level issues, rerun `chezmoi apply` against the specific changed source paths so the relevant config still deploys without broad side effects.
- Treat brew bundle work triggered by `chezmoi apply` as fire-and-forget once the apply itself succeeds.
- Summarise any conflicts, resolutions, and semantic carry-forwards.

### 7. Meta self-improvement loop

Run a brief rebase retrospective before ending:

- What signals correctly identified local intent early?
- What nearly got lost during the rebase, especially due to upstream refactors or file moves?
- What extra search, audit, or rule would have caught that earlier?

Then improve the workflow immediately:

- If the miss is repo-specific, update local memory in `AGENTS.md`.
- If the lesson is broadly reusable for OpenCode sessions, update global memory in `home/dot_config/opencode/AGENTS.md`.
- If this skill itself missed a safeguard, update `.opencode/skills/rebase-upstream/SKILL.md` in the same session so future rebases inherit the fix.

## Required output

The final response should include:

- the upstream target used
- whether the working tree was dirty and how it was preserved
- each conflict and its semantic resolution
- each local preference that was intentionally carried forward
- any follow-up memory or skill improvements made by the self-improvement loop
