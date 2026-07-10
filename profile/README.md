# Bioconductor Package Rescue

Welcome to the **Bioconductor Package Rescue** organization! 🧬🚀

This organization is a cooperative initiative dedicated to keeping valuable Bioconductor packages alive, healthy, and building successfully on the latest R and Bioconductor releases.

---

## 🔍 What is this organization for?

When a Bioconductor package starts failing its daily builds (resulting in `ERROR` or `TIMEOUT` status) or is scheduled for deprecation, it risks being removed from the next Bioconductor release. 

Our mission is to **rescue** these packages by automatically identifying, diagnosing, and fixing build and check issues. We do this by combining **automated AI coding agents** (like Google Antigravity) with a centralized **GitHub Actions (GHA) testing infrastructure**.

---

## ⚙️ How it works

1. **Active Monitoring**: We track the official Bioconductor Help Wanted page and daily build status DBs.
2. **Rescue Forks**: For packages that are failing checks or flagged for deprecation, we fork or clone them here into the `bioc-package-rescue` organization.
3. **Centralized Testing**: We configure each fork to run a centralized GitHub Actions workflow (`bioc-package-rescue/workflows`) that automatically tests the package against both the **Bioconductor Release** and **Bioconductor Devel** docker images.
4. **Agentic Auto-Fixes**: We deploy AI coding agents to read GHA check logs, diagnose the issues (e.g., deprecated R/package APIs, broken documentation syntax, or missing dependencies), and apply fixes on separate branches.
5. **PR and Verification**: Fixes are submitted as Pull Requests on our forks and tested. Once the check suite goes completely green, the PR is marked ready for review.

---

## 🙋‍♂️ Information for Package Maintainers

If you are the developer or maintainer of a package listed in this organization:

* **No Intrusive Changes**: We do not have write access to your original repository. All of our fixes are developed and verified entirely within our forks here.
* **Easy Upstream Integration**: Once we get a package's GHA checks to go fully green, you can review our pull requests and merge the fixes back into your original upstream repository at your convenience.
* **We Welcome Collaboration**: You are welcome to join us, review the proposed PRs, submit issues, or take over maintenance of your rescued fork!
* **Opt-Out**: If you do not want your package's fork hosted here, please open an issue in our [bioc-rescue-dashboard](https://github.com/bioc-package-rescue/bioc-rescue-dashboard) repository and we will remove it immediately.

---

## 📈 Check the Status

You can see the live status of all rescued packages, their current Bioconductor build status, download stats, and rescue branch/workflow status on our centralized **[Rescue Dashboard](https://github.com/bioc-package-rescue/bioc-rescue-dashboard)**.
