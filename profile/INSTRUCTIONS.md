# Bioconductor Package Rescue Instructions

This document describes how to maintain the `bioc-rescue-dashboard`, manage rescue repositories, and update the centralized GitHub Actions check workflow.

---

## Package List: `packages.csv`

**`packages.csv`** (in the root of the `bioc-rescue-dashboard` repository) is the single source of truth for all tracked packages. Every script reads from it — nothing is hard-coded in the scripts themselves.

### Format

```csv
package,type,source
BPRMeth,Software,deprecated
VariantAnnotation,Software,voluntarily-listed
myCustomPkg,Software,manual
```

| Column    | Values                                            |
|-----------|---------------------------------------------------|
| `package` | Bioconductor package name                         |
| `type`    | `Software`, `ExperimentData`, or `AnnotationData` |
| `source`  | `deprecated`, `voluntarily-listed`, or `manual`   |

### Adding a Package Manually

Edit `packages.csv` directly in the `bioc-rescue-dashboard` repository, add a row with `source=manual`, and commit. The next dashboard run will pick it up automatically.

---

## Configuration

Two scripts hard-code the local workspace root path. If running on a different machine, update `WORKSPACE_ROOT` near the top of these files (located in the `bioc-rescue-dashboard` repository):

- `bioc-rescue-dashboard/scripts/clone_repos.py`
- `bioc-rescue-dashboard/scripts/update_reusable_workflows.py`

Default: `/Users/Levi/git/bioc-package-rescue`

---

## Typical Workflow

Commands must be run from the `bioc-rescue-dashboard` directory. Run in order when setting up fresh or after Bioconductor publishes updates:

```bash
cd bioc-rescue-dashboard

# 1. Sync package list from Bioconductor Help Wanted page → packages.csv
python scripts/sync_packages.py
git add packages.csv && git commit -m "Sync packages.csv"

# 2. Create rescue repos on GitHub (fork or clone from git.bioconductor.org)
python scripts/rescue_repos.py

# 3. Clone rescue repos locally
python scripts/clone_repos.py

# 4. Apply centralized GHA workflow to all local checkouts
python scripts/update_reusable_workflows.py

# 5. Regenerate the README.md dashboard
python scripts/update_deprecated_packages.py
git add README.md && git commit -m "Update dashboard"
```

Steps 2–5 are idempotent — already-existing repos, clones, and workflows are skipped automatically.

---

## Script Reference

All script paths below are relative to the `bioc-rescue-dashboard` repository.

### `scripts/sync_packages.py` — Update the package list

Fetches the Bioconductor [Help Wanted page](https://bioconductor.org/developers/help_wanted/) and merges it into `packages.csv`:
- New packages from the **Deprecated** and **Voluntarily Listed** sections are added.
- Existing rows (including `source=manual` entries) are **never removed or overwritten**.

### `scripts/rescue_repos.py` — Create GitHub repos

For each package in `packages.csv` with build status `ERROR` or `TIMEOUT`, creates a public repository in the `bioc-package-rescue` organization:
- If the package has an upstream **GitHub repository**, it is **forked**.
- Otherwise it is **cloned from `git.bioconductor.org`** and pushed as a new repo.
- Packages whose org repo already exists are skipped.

Requires the `gh` CLI authenticated to an account with organization-level repository creation permissions.

### `scripts/clone_repos.py` — Clone repos locally

Clones each rescue repo (ERROR/TIMEOUT packages only) from `git@github.com:bioc-package-rescue/<pkg>` into the local workspace. Already-existing local directories are skipped.

### `scripts/update_reusable_workflows.py` — Push GHA workflow stubs

Writes the minimal caller stub (see below) to `.github/workflows/check-bioc.yml` in every local checkout with ERROR/TIMEOUT status, commits with `Antigravity <gemini@google.com>` as author, and pushes. Already-up-to-date repos are skipped. The commit author is hardcoded; update the script if using a different agent.

### `scripts/update_deprecated_packages.py` — Regenerate dashboard README

Reads `packages.csv`, fetches the current Bioconductor build status and most recent complete-year download statistics, and rewrites `README.md` with two tables: **Deprecated Packages** and **Voluntarily Listed / Manual packages**.

### `scripts/submit_upstream.py` — Submit PRs to upstream parents

Automates creating a Pull Request back to the original upstream repository.
Usage: `python scripts/submit_upstream.py <package_name> <fix_branch_name>`
- Resolves the upstream parent of the rescue fork.
- Adds the upstream remote and fetches the default branch.
- Creates a clean integration branch excluding rescue-specific configurations (like `.github/workflows/check-bioc.yml` or `.Rbuildignore` additions unless substantive).
- Pushes the branch and opens a GitHub Pull Request to the original maintainers.

### `scripts/generate_report.py` — Generate fix statistics

Generates comprehensive reports of the fixes applied during the rescue process.
- Parses git histories across all rescued packages to identify commits authored by the rescue agents.
- Aggregates insertion/deletion line counts and links to actual commits/PRs.
- Outputs `package_fix_details.md` and `package_fix_stats.csv` into the `bioc-rescue-dashboard` root.

---

## Centralized GHA Workflow

### Central repository

- **URL**: `https://github.com/bioc-package-rescue/workflows`
- **File**: `.github/workflows/check-bioc.yml`

This holds the full workflow definition. The checks are run only against the **Bioconductor Devel** Docker image (`bioconductor/bioconductor_docker:devel`). To update or change the check parameters, modify this workflow file, commit, and push to `main`. All package repositories pick up the change on their next run — no per-repo edits needed.

### Package-level caller stub

Each rescued package repository contains a minimal `.github/workflows/check-bioc.yml`:

```yaml
name: R-CMD-check-bioc

on:
  push:
    branches:
      - main
      - master
      - devel
  pull_request:

jobs:
  run-check:
    uses: bioc-package-rescue/workflows/.github/workflows/check-bioc.yml@main
```

Run `update_reusable_workflows.py` to batch-apply or refresh this stub across all local checkouts.

> [!IMPORTANT]
> If the upstream repository already has existing GitHub Actions workflow files under `.github/workflows/`, they must be removed or replaced entirely with our centralized rescue workflow stub (`check-bioc.yml`) to prevent duplicate or incompatible runs. However, just as with other rescue-specific files, these workflow changes must never be included in the upstream PRs targeting the original package repository.

---

## Fixing Packages ("Go Until Green")

Once rescue repos exist and the GHA workflow is running, the goal is to make each
repo pass `R CMD check` and BiocCheck with no ERRORs. WARNINGs and NOTEs are
informational and can be ignored.

### Ground Rules

- **One package at a time** — complete the full fix loop for one package before
  starting the next.
- **Always open a PR** — never push fixes directly to the default branch. PRs allow GitHub Copilot review before merging.
- **Rescue fork only** — fixes stay in the `bioc-package-rescue` fork. Upstream PRs
  to the original maintainer's repo are out of scope and are done manually.
- **Never silence errors by removing or disabling tests** — this includes deleting
  test files, commenting out test blocks, wrapping them in `if (FALSE) {...}`, or
  using `\dontrun{}` in examples. Fix the underlying problem instead. The only
  exceptions are code genuinely not intended to be tested: pseudocode used for
  illustration, or code that cannot run in CI/CD (e.g., requires live credentials,
  proprietary hardware, or interactive user input). In those cases, `\dontrun{}`
  is appropriate and must include a comment explaining why.
- **GHA is the authority** — the local machine may differ from the
  GHA environment (x86_64 Ubuntu inside Bioconductor Docker). Always confirm fixes
  via the GHA run, not just local `Rscript` output.
- **Include the co-author trailer** in every commit — use the trailer for the agent making the changes (substitute accordingly for other agents):
  ```
  Co-authored-by: Antigravity <gemini@google.com>
  ```
- **Handling non-standard files** — If you find non-standard files committed by the original maintainers (e.g., log files, build artifacts), do not delete them — add them to `.Rbuildignore` instead.
- **Avoiding line-ending problems (CRLF vs LF)** — Some upstream repositories (especially those originally maintained on Windows) contain files with Windows-style CRLF (`\r\n`) line endings. To prevent massive, misleading git diffs that inflate insertion/deletion statistics and obscure substantive changes:
  1. Always check if the original package files use CRLF line endings.
  2. If they do, configure Git locally within that repository to disable automatic line-ending conversion before making or staging edits:
     ```bash
     git config core.autocrlf false
     ```
  3. Ensure that your text editor or IDE saves edits using the repository's original line endings (CRLF or LF), or use a utility/script to restore CRLF to any modified files before staging and committing.
- **Never remove deprecation hooks** — If the package has `.Deprecated()` calls or load-time warnings in `.onAttach`/`.onLoad` (typically in `R/zzz.R`), do not delete or silence them.

### Single-Package Fix Loop

```
1. Fetch latest GHA logs
   env GITHUB_TOKEN="" gh run list --workflow=check-bioc.yml --repo bioc-package-rescue/<PKG>
   env GITHUB_TOKEN="" gh run view <RUN_ID> --repo bioc-package-rescue/<PKG> --log-failed

2. Parse all ERRORs; ignore WARNINGs and NOTEs

3. Create a fix branch
   git -C <PKG> checkout -b fix/<short-description>

4. Apply fix(es) to source files

5. Push the branch → GHA is triggered automatically
   git -C <PKG> push origin fix/<short-description>

6. Wait for GHA to complete, then check the result
   env GITHUB_TOKEN="" gh run list --workflow=check-bioc.yml --repo bioc-package-rescue/<PKG>

7a. No ERRORs → open a PR
    env GITHUB_TOKEN="" gh pr create --repo bioc-package-rescue/<PKG> \
      --title "Fix R CMD check errors" --body "..." --base <default-branch>

7b. Still failing → return to step 1 and iterate

8. Monitor and address feedback — wait for and address feedback from GitHub Copilot reviews on the pull request. Push any necessary fixes
to the branch until the PR is approved and merged.
```

### Common Error Classes

| Error class | Severity | Examples | Typical fix |
|-------------|----------|----------|-------------|
| Deprecated API | ERROR | `graph.incidence()` → `graph_from_biadjacency_matrix()` | Update R code or example |
| Hard dependency broken | ERROR | Upstream package removed or renamed | Update `DESCRIPTION` imports |
| Failing unit test | ERROR | Test written against a stale API | Fix the test |
| Vignette build failure | ERROR | LaTeX / knitr error | Update vignette |
| Missing `\\link` package anchor | WARNING | `\\link{Foo}` without package qualifier | Change to `\\link[pkg]{Foo}` in Rd |
| Lost braces in Rd | WARNING | `\\itemize` where `\\describe` is required | Fix Rd markup |
| `\\usage` / signature mismatch | WARNING | Function arguments changed since Rd was written | Update Rd or re-roxygenize |
| BiocCheck: missing/invalid `biocViews` | ERROR | `biocViews` absent or not from the controlled vocabulary | Add valid terms to `DESCRIPTION` |
| BiocCheck: `T`/`F` shorthand | WARNING | `if (T)` instead of `if (TRUE)` | Replace with `TRUE`/`FALSE` throughout |
| BiocCheck: `require()` in function body | WARNING | `require(pkg)` inside a function | Use `pkg::fun()` or move to `Imports` |

### Iterating Over All Packages

1. **Triage** — for every package in `packages.csv` that has a rescue repo, fetch
   the latest GHA result and record the error class and severity.
2. **Fix & iterate** — work through the queue one package at a time using the loop
   above.
3. **Upstream PR Submission** — once the rescue PR is verified green, you must open a pull request back to the original upstream parent repository *without* including `.github/workflows/check-bioc.yml` or other rescue-specific files in the diff.

   You can automate this by running the submit script from the dashboard repository:
   ```bash
   python scripts/submit_upstream.py <package_name> <fix_branch_name>
   ```
   This will automatically isolate your package code changes and open a PR upstream.

4. **Dashboard** — after a batch of PRs is merged, regenerate the README and package fix reports:
   ```bash
   cd bioc-rescue-dashboard

   # Regenerate the main landing page table
   python scripts/update_deprecated_packages.py

   # Regenerate the substantive commits, diffs, and lines-changed statistics reports
   python scripts/generate_report.py

   git add README.md package_fix_details.md package_fix_stats.csv
   git commit -m "Update dashboard and package fix reports

Co-authored-by: Antigravity <gemini@google.com>"
   git push origin main
   ```
