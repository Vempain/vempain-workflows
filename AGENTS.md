# AGENTS.md

## Purpose and scope

- This repository is a **workflow catalog** for other Vempain repos, not an app/service runtime.
- Main assets:
    - reusable workflows in `.github/workflows/`
    - shared composite action in `.github/actions/generate-version/action.yaml`
    - copy-ready templates in `dependabot-templates/`

## Big-picture architecture

- Caller repositories invoke workflows via `uses: Vempain/vempain-workflows/.github/workflows/<file>@main` (see
  `README.md`).
- Most workflows follow a two-lane design:
    - non-`main` refs run validation/test jobs
    - `push` to `main` performs versioning + publish/release side effects
- Versioning is centralized in `.github/actions/generate-version/action.yaml`:
    - reads `VERSION` (or configured file)
    - fetches git tags
    - emits `base_version`, `main_version`, `new_version`
- Release lane typically creates a git tag and GitHub Release after artifact publishing.

## Workflow families and boundaries

- `spring-boot-service.yaml`: Gradle tests on non-main, Docker publish on main, optional `:api:publish` gated by
  merge-commit message.
- `spring-boot-library.yaml`: Gradle library publish to GitHub Maven (no Docker), requires `VEMPAIN_ACTION_TOKEN`.
- `frontend-spa.yaml`: Node/Yarn test lane + Docker publish lane with optional version-bump commit/tag-prefix behavior.
- `frontend-library.yaml`: npm package publish to GitHub Packages (no Docker), always sets package version before
  publish.
- `website.yaml`: mixed backend/frontend pipeline (PHP + Node), builds and pushes two Docker images.
- `rpm-cli-package.yaml`: Java fat-jar -> RPM packaging flow, uploads binary + source RPM artifacts, can attach assets
  to release.

## Project-specific conventions to preserve

- Keep `concurrency: build` in reusable workflows unless you intentionally change cross-run serialization.
- Preserve token fallback pattern where present:
    - `secrets.VEMPAIN_ACTION_TOKEN != '' && secrets.VEMPAIN_ACTION_TOKEN || secrets.GITHUB_TOKEN`.
- Preserve image naming behavior:
    - Spring/SPA workflows derive image from `${{ github.repository }}` and lowercase it.
    - Website workflow expects explicit `backend_image` / `frontend_image` inputs.
- Keep release notes artifact links ecosystem-specific (container, maven, npm, RPM assets).
- In `frontend-spa.yaml`, keep loop prevention guard:
    - `github.actor != 'github-actions[bot]'` when `commit_version_bump` is enabled.

## Integration points and external actions

- Core third-party actions: `actions/checkout`, `actions/setup-node`, `actions/setup-java`,
  `gradle/actions/setup-gradle`, `docker/login-action`, `actions/github-script`, `actions/upload-artifact`.
- Website backend uses `shivammathur/setup-php`; RPM flow uses `naveenrajm7/rpmbuild@master`.
- Dependabot templates are intentionally ecosystem-split by repo type; copy one template to target repo as
  `.github/dependabot.yml`.

## Editing checklist for agents

- When changing version/tag logic, update **both** workflow consumers and `.github/actions/generate-version/action.yaml`
  expectations.
- When adding workflow inputs, update `README.md` tables/examples in the same change.
- Prefer keeping test-job and release-job trigger conditions aligned with existing `if:` patterns.
- For registry/auth changes, verify all env aliases (`PACKAGE_AUTH_TOKEN`, `VEMPAIN_ACTION_TOKEN`, `NODE_AUTH_TOKEN`)
  still resolve correctly.
- If changing RPM release behavior, verify both artifact upload steps and the release-asset attachment script path
  variables remain consistent.

