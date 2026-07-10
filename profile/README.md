# Bioconductor Package Rescue

This is an effort to autofix or "rescue" Bioconductor packages that are failing
build/check/test on the Bioconductor build system and are marked for deprecation.

---

## Check the Status

You can see the live status of all rescued packages, their current Bioconductor build status, download stats, and rescue branch/workflow status on our centralized **[Rescue Dashboard](https://github.com/bioc-package-rescue/bioc-rescue-dashboard)**. Packages with high download stats should be prioritized.

---

## How it works

1. **Monitoring**: Currently has all packages listed on the Bioconductor Help Wanted page. Could be extended to monitor daily build status DBs.
2. **Rescue Forks**: For packages that are failing checks or flagged for deprecation, we fork (if maintained on GitHub) or clone (if only on git.bioconductor.org) into the `bioc-package-rescue` organization.
3. **GHA Testing**: We configure each fork to run a centralized GitHub Actions workflow (`bioc-package-rescue/workflows`) that automatically tests the package against the **Bioconductor Release** and **Bioconductor Devel** docker images. To save time/cost, we only test the default branch and only on Linux, which should be adequate to fix most of the types of issues that threaten deprecation.
4. **Agentic Auto-Fixes**: A locally-running AI coding agents read GHA check logs, diagnoses the issues, and apply fixes as a PR.
5. **Verification**: The agent waits for the GHA and the GitHub Copilot PR code review, and updates the PR based on information from both. This process iterates until there are no ERRORs or WARNINGs in the GHA build/check, at which point the PR is marked ready for human review.

---

## Information for Package Maintainers

If you are the developer or maintainer:

* **No Intrusive Changes**: We do not have write access to your original repository. All of our fixes are developed and verified entirely within our forks here.
* **Easy Upstream Integration**: Once we get a package's GHA checks to go fully green and a human has reviewed the PR, we will a) submit a PR to your upstream repo if it is on GitHub, or b) allow you to review the PR in this rescue repo before you push the changes to git.bioconductor.org.

