# bioc-package-rescue

The `bioc-package-rescue` organization exists to help recover Bioconductor packages
that are failing build or check. It tracks packages that need rescue, creates and
maintains per-package rescue repositories, applies a centralized GitHub Actions
workflow for Bioconductor checks, and supports the "go until green" process of
iterating on fixes until packages pass `R CMD check` and BiocCheck without ERRORs
or WARNINGs.
