# apply-branching-strategy

A [Claude Code](https://claude.com/claude-code) skill that sets up a standard,
PR-based Git branching strategy on any repository — and documents it so the whole
team follows the same flow.

## What it does

- Makes **`main`** the mainline (default branch); creates it from the current default if missing.
- Marks **`main`** and **`master`** as **PR-only** — no direct commits or force pushes.
- Standardizes **typed branches** (`feature/<name>`, `fix/`, `chore/`, `docs/`, `refactor/`, `release/`, `hotfix/`).
- Documents the **BAU flow** and a **release/tagging flow** in `AGENTS.md` + the README.
- Handles permissions honestly: it does the safe parts (create `main`, write docs via a PR)
  and **hands off the admin-only steps** (branch protection, default-branch change) with the
  exact `gh` commands, rather than pretending to do what your token can't.

It's stack-agnostic — version bumps adapt to Xcode, `package.json`, `pyproject.toml`, `Cargo.toml`, etc.

## Install

Clone straight into your Claude skills directory so it installs as a `/` command:

```bash
git clone https://github.com/akhlaqahmad/apply-branching-strategy.git \
  ~/.claude/skills/apply-branching-strategy
```

Or copy just the `SKILL.md` into `~/.claude/skills/apply-branching-strategy/`.

New skills are picked up at the **next** Claude Code session start.

## Usage

In any repository, either invoke it explicitly:

```
/apply-branching-strategy
```

…or just describe the goal — it auto-triggers on requests like *"set up branch protection"*,
*"make main PR-only, no direct commits"*, or *"apply our branching strategy to this repo"*.

## Requirements

- [Claude Code](https://claude.com/claude-code)
- `git`, and the [`gh` CLI](https://cli.github.com/) for the GitHub automation steps
- **Admin** on the target repo (and, for private repos, a plan that includes protected
  branches) to apply branch protection / change the default branch. Without admin the skill
  still creates `main` and the docs, and hands you the admin commands.

## License

[MIT](LICENSE) © Akhlaq Ahmad
