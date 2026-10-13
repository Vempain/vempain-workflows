---
name: onboard-repository-ci
description: Wire a Vempain repository to CI with the right reusable workflow and Dependabot template (Spring Boot service or library, SPA, frontend library, RPM CLI). Use when creating a repository or changing its CI entry point.
---

# Onboard a repository to the Vempain CI

1. Pick the workflow: Spring Boot service with a Docker image -> `spring-boot-service.yaml` (optional `:api:publish`
   gated by the merge
   commit message); Java library -> `spring-boot-library.yaml` (GitHub Maven, needs `VEMPAIN_ACTION_TOKEN`); Vite SPA ->
   `frontend-spa.yaml`
   (tests on PRs, Docker image on `main`, optional version-bump commit); npm library -> `frontend-library.yaml` (version
   from `VERSION`,
   publish to GitHub Packages on `main`); Java CLI -> `rpm-cli-package.yaml`.
2. Add `.github/workflows/ci.yaml` in the repository calling
   `uses: Vempain/vempain-workflows/.github/workflows/<file>@main` with the inputs
   documented in `README.md`; add a `VERSION` file where the workflow derives versions from it.
3. Copy the matching template from `dependabot-templates/` to `.github/dependabot.yaml` (the `.yaml` spelling); backends
   consuming GitHub
   Packages artifacts add a `maven-github` registry authenticated with the org secrets `VEMPAIN_ACTION_USER`/
   `VEMPAIN_ACTION_TOKEN` and
   `registries: "*"`.
4. Frontends resolve `@vempain/*` from GitHub Packages through `.yarnrc.yml`
   (`npmAuthToken: "${VEMPAIN_ACTION_TOKEN-}"`); backends use
   `gpr.user`/`gpr.token` or `GITHUB_ACTOR`/`GITHUB_TOKEN`.
5. Copy `.github/git-commit-instructions.md` from a sibling repository so Copilot commit messages follow the
   Intent/Changes/Impact format.
6. Update this repository's `README.md` consumer list when a new caller appears.
