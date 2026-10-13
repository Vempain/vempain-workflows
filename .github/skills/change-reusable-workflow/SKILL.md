---
name: change-reusable-workflow
description: Change a reusable GitHub workflow or composite action of vempain-workflows (two-lane validation/release design, generate-version, token fallbacks, image naming, release notes) without breaking the consuming repositories. Use for any CI change.
---

# Change a reusable workflow in vempain-workflows

- Families and consumers: `spring-boot-service.yaml` (admin, file, website backends), `spring-boot-library.yaml`
  (`vempain-auth`,
  `vempain-common`), `frontend-spa.yaml` (admin, file, website frontends, `vempain-site`), `frontend-library.yaml`
  (`vempain-auth-frontend`,
  `vempain-common-frontend`, `vempain-rt-editor`, `vempain-rt-renderer`), `rpm-cli-package.yaml` (`vempain-cli`);
  `website.yaml` is a
  legacy template that must be updated before reuse. Consumers call
  `uses: Vempain/vempain-workflows/.github/workflows/<file>@main`.
- Two-lane design: non-`main` refs run validation/tests; `push` to `main` versions and publishes (Docker image, Maven,
  npm or RPM), then
  tags and creates a GitHub Release (`.github/actions/create-release`, `release-notes-instructions.md`).
- Versioning is centralised in `.github/actions/generate-version/action.yaml` (`VERSION` file + git tags ->
  `base_version`,
  `main_version`, `new_version`); a change there must be checked against every consumer's expectations.
- Preserve: `concurrency: build`; the token fallback
  `secrets.VEMPAIN_ACTION_TOKEN != '' && secrets.VEMPAIN_ACTION_TOKEN || secrets.GITHUB_TOKEN`;
  lowercased `${{ github.repository }}` image names (website workflow takes explicit `backend_image`/`frontend_image`);
  the
  `github.actor != 'github-actions[bot]'` guard of `frontend-spa.yaml` when `commit_version_bump` is on;
  ecosystem-specific release note links.
- New inputs: update the `README.md` tables and examples in the same change; keep test-job and release-job `if:`
  conditions aligned with the
  existing patterns; verify the env aliases (`PACKAGE_AUTH_TOKEN`, `VEMPAIN_ACTION_TOKEN`, `NODE_AUTH_TOKEN`) still
  resolve.
- RPM changes: keep both artifact upload steps and the release-asset script paths consistent.
- Validate with `actionlint` where available and by running the consumer's workflow on a branch.
