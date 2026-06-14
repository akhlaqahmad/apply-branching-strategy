---
name: apply-branching-strategy
description: >-
  Set up a standard Git branching strategy on a repository: make `main` the
  mainline, mark `main`/`master` as PR-only (no direct commits), document the
  workflow in AGENTS.md + README, and hand off the admin-only branch-protection
  steps. Use this whenever the user wants to establish or standardize a git
  workflow, branching model, branch protection, PR-only / protected branches,
  "no direct commits to main", a release/tagging process, or asks to "apply our
  branching strategy" to a repo — even if they don't name a specific branch.
---

# Apply branching strategy

Establish a consistent, PR-based Git workflow on the current repository:

- **`main` is the mainline** (the default branch). All new work branches off it and PRs back into it.
- **`main` and `master` (if it exists) are PR-only** — no direct commits or force pushes.
- New work uses **typed branches**, commonly `feature/<feature-name>`, and merges via pull request.

The goal is a workflow that's safe (mainline always builds, every change is reviewable) and predictable for the whole team. Apply it by doing the safe parts directly and clearly handing off the parts that require repo admin.

## Step 1 — Inspect before changing anything

Understand the repo and your own access first, and report it, because the next steps branch on it:

```bash
git remote -v
gh repo view --json nameWithOwner,visibility,defaultBranchRef,viewerPermission
```

What matters:
- **`viewerPermission`** — changing the default branch and setting branch protection need **ADMIN**. With only `WRITE` you can create branches and open PRs, but the protection/default-branch steps must be handed to an admin (Step 4).
- **`visibility`** — on a **private** repo, branch protection may also require a paid plan (GitHub Pro/Team/Enterprise). Note this so the user isn't surprised if it's rejected.
- **`defaultBranchRef`** — the current mainline you'll branch `main` from.

If there's no GitHub remote (e.g. plain git, GitLab, Bitbucket), adapt: still create `main` and document the flow, and give the equivalent protection steps for that host instead of `gh`.

## Step 2 — Create `main` (safe; WRITE is enough)

If `main` doesn't already exist, create it from the current default branch's latest commit so history is preserved:

```bash
git checkout <current-default> && git pull
git checkout -b main
git push -u origin main
```

If `main` already exists, leave it as-is. Never rewrite mainline history.

## Step 3 — Document the strategy (via a PR, to model the flow)

Update **AGENTS.md** (create it if missing) and mirror a short version in **README**'s Contributing section. Do this on a `docs/branching-strategy` branch and open a PR into `main` — the first change should itself follow the new flow.

Document these, and explain the *why* (not just rules) so the team understands the intent:

- `main` is the mainline; `main` + `master` are PR-only (no direct commits). `master`, if present, is legacy history kept around; `main` is the going-forward default.
- **Branch prefixes** (pick by intent):

  | Prefix | For |
  | --- | --- |
  | `feature/` | new functionality (the common case → `feature/<feature-name>`) |
  | `fix/` | bug fixes |
  | `chore/` | deps, tooling, build, config |
  | `docs/` | documentation only |
  | `refactor/` | restructuring with no behavior change |
  | `release/` | release prep (version bump, final fixes) |
  | `hotfix/` | urgent fix branched off a release tag |

- **BAU flow:** `git checkout main && git pull && git checkout -b feature/<name>` → commit focused changes → build/test to verify → push → `gh pr create --base main` → merge when green. Direct pushes to `main` are blocked by protection.
- **Release flow:** branch `release/X.Y.Z` off `main` → bump the project's version/build number → PR into `main` → after merge tag `git tag -a vX.Y.Z -m "…" && git push origin vX.Y.Z` (optionally `gh release create vX.Y.Z`). Hotfixes branch `hotfix/X.Y.(Z+1)` off the tag and PR into `main`.
- A **"Branch protection (admin setup)"** section containing the Step 4 commands.

Adapt the version-bump details to the project's stack — e.g. Xcode `MARKETING_VERSION`/`CURRENT_PROJECT_VERSION`, `package.json`, `pyproject.toml`, `Cargo.toml`, etc.

## Step 4 — Protection + default branch (ADMIN required)

Attempt these. If they fail (GitHub returns **404 or 403** when you lack admin — it hides the endpoint on private repos), **stop and do not pretend they succeeded.** Report clearly and hand the owner these exact commands to run:

```bash
# Make main the default branch
gh repo edit <owner>/<repo> --default-branch main

# Protect main: require a PR before merging, block force pushes — repeat for master
gh api -X PUT repos/<owner>/<repo>/branches/main/protection \
  -F enforce_admins=true -f required_status_checks=null -f restrictions=null \
  -f 'required_pull_request_reviews[required_approving_review_count]=1'

gh api -X PUT repos/<owner>/<repo>/branches/master/protection \
  -F enforce_admins=true -f required_status_checks=null -f restrictions=null \
  -f 'required_pull_request_reviews[required_approving_review_count]=1'
```

UI equivalent: **Settings → General → Default branch** = `main`; **Settings → Branches → Add rule/ruleset** for `main` and `master` with *Require a pull request before merging* + *Block force pushes* enabled.

## Step 5 — Summarize

Report what was applied vs. what still needs an admin, with the exact follow-up commands. Be honest about anything blocked by permissions or plan — a half-applied policy that looks complete is worse than a clearly-flagged TODO.

## Guardrails

- Never commit directly to `main`/`master` — route every change through a typed branch + PR, including the doc change in Step 3.
- Don't force-push or rewrite mainline history.
- Only attempt admin actions you may not have rights for once; on failure, hand them off rather than retrying.
