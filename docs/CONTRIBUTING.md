# Contributing Guidelines — VoltaGrid Analytics

> Language: [🇺🇸 English](CONTRIBUTING.md) | [🇪🇸 Español](CONTRIBUTING.es.md)

> **Note (P4):** this guide was adapted from `voltiagrid-api`. Generic workflow (branches, commits, PRs) is final.
> Sections marked `TODO(P4)` need the real Power BI / curated-connection setup once the model exists.

This guide covers the full workflow: `git clone` → setup → branch → commit → PR → review → merge.
It combines [Conventional Commits](https://www.conventionalcommits.org/) (like the CoDecide reference repo) with the team's **Jira (KAN)** workflow.

---

## 0. Prerequisites

- Git, Python 3.12, Docker + Docker Compose.
- A GitHub account with access to `VoltiaGrid-Medicion-Inteligente/voltiagrid-analytics`.
- A Jira account. Your local git email **must match** your Jira email, otherwise Smart Commits won't link:
  ```bash
  git config user.name "Your Name"
  git config user.email "you@jira-email.com"
  ```

## 1. Clone and setup (first time only)

```bash
# HTTPS (simplest)
git clone https://github.com/VoltiaGrid-Medicion-Inteligente/voltiagrid-analytics.git
cd voltiagrid-analytics

# or SSH (if you use SSH keys)
# git clone git@github.com:VoltiaGrid-Medicion-Inteligente/voltiagrid-analytics.git
# cd voltiagrid-analytics

<!-- TODO(P4): replace with the real local run: where the curated sample lives, how to refresh the .pbix. -->
```

Rules:

- **Never commit credentials or connection strings.** Sample data only, no customer secrets.
- <!-- TODO(P4): document the curated connection (read-only) without exposing keys. -->

## 2. Branch Strategy

```
main ────────────── stable branch, PRs merge here (demo / release)
  ├── feature/KAN-12-short-description
  ├── fix/KAN-13-short-description
  ├── refactor/KAN-14-short-description
  ├── docs/KAN-15-short-description
  └── chore/KAN-16-short-description
```

### Branch Naming Convention

```
<type>/KAN-<number>-<short-description>
```

| Type | When to use | Example |
|------|-------------|---------|
| `feature/` | New functionality / user story | `feature/KAN-12-meter-consumer` |
| `fix/` | Bug fix | `fix/KAN-13-login-redirect-loop` |
| `refactor/` | Restructure without behavior change | `refactor/KAN-14-extract-meter-service` |
| `chore/` | Tooling, dependencies, config, CI | `chore/KAN-16-upgrade-pytest` |
| `docs/` | Documentation only | `docs/KAN-15-document-f3-simulator` |
| `test/` | Tests only | `test/KAN-16-losses-control-query` |

- Jira key in **UPPERCASE** (`KAN-12`, not `kan-12`) so Jira auto-links branch → issue.
- Description in **kebab-case**, short but meaningful, English preferred.
- All branches come from up-to-date `main`.

### Rules

- **Never push directly to `main`.** All changes via Pull Request.
- Any commit pushed directly to `main` will be reverted/deleted.
- One branch per Jira issue/task. If the task grows, split the issue, don't grow the branch.
- Keep `main` green: pull before branching.

Create a branch:

```bash
git checkout main
git pull origin main
git checkout -b feature/KAN-12-meter-consumer
```

## 3. Conventional Commits + Jira

### Format

```
KAN-<number> <type>(<scope>): <description>
```

- The `KAN-XX` prefix keeps Jira automation (branch/commit/PR all linked).
- The rest follows Conventional Commits.

### Types

| Type | When to use |
|------|-------------|
| `feat` | New feature |
| `fix` | Bug fix |
| `refactor` | Neither fix nor feature |
| `style` | Formatting only (no production change) |
| `docs` | Documentation only |
| `chore` | Build, deps, tooling, CI |
| `test` | Add/modify tests |
| `perf` | Performance improvement |

### Scopes (this repo)

Model: `model`, `facts`, `dims`, `queries`
Dashboard: `powerbi`, `visuals`
Cross-cutting: `config`, `ci`, `docs`, `deps`

### Examples (copy the style)

```
KAN-12 feat(facts): add consumption fact at meter-interval grain
KAN-13 fix(powerbi): correct losses visual to match control query
KAN-14 refactor(model): extract shared time dimension
KAN-15 docs(queries): document reconciliation queries for the 6 questions
KAN-16 test(queries): add control query for demand-response savings
KAN-15 chore(config): add curated sample path override by env
```

### Rules

- **Jira key first, UPPERCASE** (`KAN-12`, not `kan-12`).
- **Description in English, imperative present tense:** "add" not "added"/"adds".
- **Lowercase description, no trailing period**, concise (<72 chars if possible).
- One logical change per commit. Two unrelated fixes → two commits.
- Small commits preferred. 20+ files in one commit → split it.

Good split:

```
KAN-12 feat(facts): add losses fact at transformer-day grain
KAN-12 feat(queries): add control query reconciling losses visual
```

Bad:

```
KAN-12 feat: add lots of stuff   # 35 files, 1200 additions
```

Fix a message before pushing:

```bash
git commit --amend -m "KAN-12 feat(facts): correct message"
# if already pushed to YOUR branch only:
git push --force-with-lease
```

### Smart Commits (Jira automation)

Only works if `git config user.email` == Jira email:

```
KAN-12 #comment ready for review
KAN-12 #done
```

Use them in a separate commit or in the PR description — don't mix with code changes silently.

## 4. Pull Request Workflow

1. Update from `main`, run checks:

   ```bash
   git checkout feature/KAN-12-meter-consumer
   git pull --rebase origin main
   python -m pytest tests/ -v
   ```

2. Push and open a PR **targeting `main**:

   ```bash
   git push -u origin feature/KAN-12-meter-consumer
   ```

3. PR title = same as commit format (Jira key + Conventional):

   ```
   KAN-12 feat(facts): add consumption fact at meter-interval grain
   ```

4. PR description (required template):

   ```markdown
   ## What
   Brief description of the change.

   ## Why
   Reason + Jira issue (e.g. KAN-12).

   ## How to test
   <!-- TODO(P4): fill the real check. Skeleton: -->
   1. Refresh the .pbix against the curated sample
   2. Run control queries — dashboard numbers must match

   ## Screenshots / evidence (if applicable)
   ```

5. Wait for review + green CI (lint, tests, secret scan). Address comments with **new commits**, don't rewrite history under review.
6. Maintainer merges into `main` via reviewed PR (squash by default, keeping `KAN-XX` in title). `main` must always stay green and demo-ready.

### Pre-PR checklist

- [ ] Branch from latest `main`, name `type/KAN-XX-kebab-case`.
- [ ] Commits `KAN-XX type(scope): english imperative description`.
- [ ] `pytest` green locally (or in Docker).
- [ ] No secrets/`.env`/credentials in diff (`git status`, `git diff --check`).
- [ ] Every visual reconciles with its control query (numbers must match).
- [ ] PR targets `main`, title + template filled.

## 5. What gets your PR rejected

- Direct push to `main`.
- Missing Jira key or lowercase key (`kan-12`).
- Non-Conventional message (`added stuff`, `fix style.`, capitalised sentence).
- Giant commit, mixed concerns, or a visual with no control query.
- Secrets or private connection strings in the repo.
