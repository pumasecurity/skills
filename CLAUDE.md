# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

This is a **skill library**, not an application. It publishes Puma Security's AI skills so agent harnesses (Claude Code, etc.) can load them. There is no build, test, or lint pipeline — changes here are documentation edits to `SKILL.md` files and their `references/`.

## Layout

```
skills/
  <skill-name>/
    SKILL.md              # required — frontmatter + body, loaded by the agent
    references/           # optional — lazily loaded from SKILL.md links
```

Currently `skills/pumascan/` is the only skill. Add new skills as sibling directories under `skills/`.

## SKILL.md conventions

Every skill's `SKILL.md` starts with YAML frontmatter that the harness parses to register the skill:

```yaml
---
name: <skill-name>              # must match the directory name
description: <one-sentence>     # shown in skill-picker UI; keep it specific
allowed-tools: Bash, Grep, ...  # tools the skill is permitted to call
---
```

Body guidelines (inferred from `skills/pumascan/SKILL.md`):

- Lead with a one-line statement of what the underlying tool is.
- Document each subcommand as its own `###` section with: invocation example, required-flag table, optional-flag table, worked examples, and expected output.
- For CLI skills, include an **Exit codes** table — agents use exit codes to decide follow-up actions.
- Keep deep reference material (full rule lists, upstream docs) out of `SKILL.md`. Put it under `references/` and link with a note like: *"Only load when the user requests..."*. This keeps the agent's context small on the common path.
- Diagnostic/rule IDs follow the pattern `SEC####`.

## When editing a skill

- Preserve the frontmatter contract (`name`, `description`, `allowed-tools`) — harnesses parse it.
- `name` must match the directory name.
- Don't introduce new top-level directories outside `skills/` without a reason — the README points users at `skills/<name>/SKILL.md` paths.
- The LICENSE at `./LICENSE` is MPL 2.0 for the skills markdown. Keep it in place.
