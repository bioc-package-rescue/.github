# Bioconductor Package Rescue

This is an effort to autofix or "rescue" Bioconductor packages that are failing
build/check/test on the Bioconductor build system and are marked for deprecation. For some details see the [INSTRUCTIONS.md](INSTRUCTIONS.md). To give feedback to or ask questions or offer help to the human behind this project, email levi.waldron at sph.cuny.edu.

---

## Check the Status

You can see the live status of all rescued packages, their current Bioconductor build status, download stats, and rescue branch/workflow status on our centralized **[Rescue Dashboard](https://github.com/bioc-package-rescue/bioc-rescue-dashboard)**. Packages with high download stats should be prioritized.

Detailed reports and metrics are available in the dashboard repository:
* **[Dashboard Home (README.md)](https://github.com/bioc-package-rescue/bioc-rescue-dashboard)** - Centralized view of all packages, build status, and download stats.
* **[Rescue Fixes Summary (rescue_fixes_summary.md)](https://github.com/bioc-package-rescue/bioc-rescue-dashboard/blob/main/rescue_fixes_summary.md)** - Technical overview and statistics of the categories of programmatic and metadata fixes applied.
* **[Detailed Package Fixes (package_fix_details.md)](https://github.com/bioc-package-rescue/bioc-rescue-dashboard/blob/main/package_fix_details.md)** - Full description of applied fixes, substantive commits, and comprehensive diffs for all 37 packages.
* **[Package Fix Stats (package_fix_stats.csv)](https://github.com/bioc-package-rescue/bioc-rescue-dashboard/blob/main/package_fix_stats.csv)** - A machine-readable breakdown of the number of files changed, additions, and deletions per package.

---

## How it works

1. **Monitoring**: Currently has all packages listed on the Bioconductor Help Wanted page. Could be extended to monitor daily build status DBs.
2. **Rescue Forks**: For packages that are failing checks or flagged for deprecation, we fork (if maintained on GitHub) or clone (if only on git.bioconductor.org) into the [bioc-package-rescue](https://github.com/bioc-package-rescue) organization.
3. **GHA Testing**: We configure each fork to run a [centralized GitHub Actions workflow](https://github.com/bioc-package-rescue/workflows/blob/main/.github/workflows/check-bioc.yml) that builds/checks/tests against the **Bioconductor Devel** docker image. To save time/cost, we only test the default branch and only on Linux, which should be adequate to fix most of the types of issues that threaten deprecation.
4. **Agentic Auto-Fixes**: A locally-running AI coding agents read GHA check logs, diagnoses the ERRORs, and applies fixes as a PR. WARNINGs and NOTEs are ignored.
5. **Verification**: The agent waits for the GHA and the GitHub Copilot PR code review, and updates the PR based on information from both. This process iterates until there are no ERRORs in the GHA build/check, at which point the PR is marked ready for human review.

---

## Information for Package Maintainers

If you are the developer or maintainer:

* **No Intrusive Changes**: We do not have write access to your original repository. All of our fixes are developed and verified entirely within our forks here.
* **Easy Upstream Integration**: Once we get a package's GHA checks to go fully green and a human has reviewed the PR, we will a) submit a PR to your upstream repo if it is on GitHub, or b) allow you to review the PR in this rescue repo before you push the changes to git.bioconductor.org.

