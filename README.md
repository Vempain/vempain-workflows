# vempain-workflows

Common GitHub Actions reusable workflows and Dependabot configuration templates
for all [Vempain](https://github.com/Vempain) repositories.

---

## Reusable workflows

All reusable workflows live in `.github/workflows/` and are called via
`uses: Vempain/vempain-workflows/.github/workflows/<file>@main`.

### 1. `spring-boot-service.yaml`

**For:** Spring Boot services that produce a Docker image (± Maven API artefact).

| Caller                  | `install_exiftool` | `run_test_setup` |
|-------------------------|--------------------|------------------|
| `vempain-admin-backend` | `true`             | `true`           |
| `vempain-file-backend`  | `true`             | `false`          |

**Inputs**

| Name                | Type      | Default                 | Description                                                            |
|---------------------|-----------|-------------------------|------------------------------------------------------------------------|
| `java_version`      | `string`  | `'25'`                  | Temurin JDK version                                                    |
| `install_exiftool`  | `boolean` | `false`                 | Install `exiftool` before tests                                        |
| `run_test_setup`    | `boolean` | `false`                 | Run `./testSetup.sh` before tests                                      |
| `publish_api`       | `boolean` | `true`                  | Publish `:api` sub-project to GitHub Packages (Maven) on merge commits |
| `test_results_path` | `string`  | `service/build/reports` | Path to upload as test-results artefact                                |

**Minimal caller example**

```yaml
# .github/workflows/ci.yaml
name: CI
on:
  pull_request:
    branches: [ main ]
  push:
    branches: [ main ]
    tags: [ 'v*.*.*' ]
permissions:
  contents: write
  packages: write
  id-token: write
jobs:
  ci:
    uses: Vempain/vempain-workflows/.github/workflows/spring-boot-service.yaml@main
    with:
      install_exiftool: true
      run_test_setup: true   # admin-backend only
    secrets: inherit
```

---

### 2. `spring-boot-library.yaml`

**For:** Spring Boot libraries published to GitHub Packages (Maven), no Docker image.

| Caller         |
|----------------|
| `vempain-auth` |

**Inputs**

| Name           | Type     | Default | Description         |
|----------------|----------|---------|---------------------|
| `java_version` | `string` | `'25'`  | Temurin JDK version |

**Secrets** — `VEMPAIN_ACTION_TOKEN` (required)

**Minimal caller example**

```yaml
# .github/workflows/ci.yaml
name: CI
on:
  pull_request:
    branches: [ main ]
  push:
    branches: [ main ]
    tags: [ 'v*.*.*' ]
permissions:
  contents: write
  packages: write
jobs:
  ci:
    uses: Vempain/vempain-workflows/.github/workflows/spring-boot-library.yaml@main
    secrets: inherit
```

---

### 3. `frontend-spa.yaml`

**For:** TypeScript/JS single-page applications that produce a Docker image.

| Caller                   | Notable overrides                                                                                                                                                                       |
|--------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `vempain-admin-frontend` | defaults                                                                                                                                                                                |
| `vempain-file-frontend`  | `use_scoped_registry: true`, `use_immutable_install: true`                                                                                                                              |
| `vempain-site`           | `node_version: '20'`, `use_corepack: true`, `use_immutable_install: true`, `build_command: build`, `commit_version_bump: true`, `push_all_version_tags: false`, `version_tag_prefix: v` |

**Inputs**

| Name                    | Type      | Default                              | Description                                                          |
|-------------------------|-----------|--------------------------------------|----------------------------------------------------------------------|
| `node_version`          | `string`  | `'24'`                               | Node.js version                                                      |
| `use_scoped_registry`   | `boolean` | `false`                              | Configure `@vempain` scope against `npm.pkg.github.com`              |
| `use_immutable_install` | `boolean` | `false`                              | Pass `--immutable` to `yarn install`                                 |
| `use_corepack`          | `boolean` | `false`                              | Run `corepack enable` before Node setup                              |
| `build_command`         | `string`  | `build:production`                   | Yarn script for the production bundle                                |
| `test_extra_flags`      | `string`  | `--passWithNoTests --watchAll=false` | Flags appended to `yarn test`                                        |
| `run_coverage`          | `boolean` | `true`                               | Run `yarn test:coverage`                                             |
| `commit_version_bump`   | `boolean` | `false`                              | Commit `package.json` version bump back to the branch                |
| `push_all_version_tags` | `boolean` | `true`                               | Push `latest`/major/minor/patch tags; set `false` for patch-tag only |
| `version_tag_prefix`    | `string`  | `''`                                 | Tag prefix, e.g. `v`                                                 |

**Secrets** — `VEMPAIN_ACTION_TOKEN` (optional, falls back to `GITHUB_TOKEN`)

**Minimal caller example**

```yaml
# .github/workflows/ci.yaml
name: CI
on:
  pull_request:
  push:
    branches: [ main ]
permissions:
  contents: write
  packages: write
jobs:
  ci:
    uses: Vempain/vempain-workflows/.github/workflows/frontend-spa.yaml@main
    with:
      use_scoped_registry: true
      use_immutable_install: true
    secrets: inherit
```

---

### 4. `frontend-library.yaml`

**For:** TypeScript/JS libraries published to GitHub Packages (npm), no Docker image.

| Caller                  |
|-------------------------|
| `vempain-auth-frontend` |

**Inputs**

| Name           | Type     | Default | Description     |
|----------------|----------|---------|-----------------|
| `node_version` | `string` | `'22'`  | Node.js version |

**Secrets** — `VEMPAIN_ACTION_TOKEN` (required)

**Minimal caller example**

```yaml
# .github/workflows/ci.yaml
name: CI
on:
  pull_request:
    branches: [ main ]
  push:
    branches: [ main ]
    tags: [ 'v*.*.*' ]
permissions:
  contents: write
  packages: write
jobs:
  ci:
    uses: Vempain/vempain-workflows/.github/workflows/frontend-library.yaml@main
    secrets: inherit
```

---

### 5. `website.yaml`

**For:** Repositories that combine a PHP Slim backend and a TypeScript frontend,
each published as a separate Docker image.

| Caller            |
|-------------------|
| `vempain-website` |

**Inputs**

| Name                  | Type     | Default                       | Description                       |
|-----------------------|----------|-------------------------------|-----------------------------------|
| `php_version`         | `string` | `'8.5'`                       | PHP version                       |
| `php_extensions`      | `string` | `bcmath, mbstring, pdo_pgsql` | Comma-separated PHP extensions    |
| `node_version`        | `string` | `'24'`                        | Node.js version                   |
| `backend_image`       | `string` | **required**                  | Full GHCR image name for backend  |
| `frontend_image`      | `string` | **required**                  | Full GHCR image name for frontend |
| `backend_dockerfile`  | `string` | `backend/Dockerfile`          | Path to backend Dockerfile        |
| `backend_context`     | `string` | `backend`                     | Docker build context for backend  |
| `frontend_dockerfile` | `string` | `frontend/Dockerfile`         | Path to frontend Dockerfile       |
| `frontend_context`    | `string` | `frontend`                    | Docker build context for frontend |

**Secrets** — `VEMPAIN_ACTION_TOKEN` (optional, falls back to `GITHUB_TOKEN`)

**Minimal caller example**

```yaml
# .github/workflows/ci.yaml
name: CI
on:
  pull_request:
  push:
    branches: [ main ]
permissions:
  contents: write
  packages: write
jobs:
  ci:
    uses: Vempain/vempain-workflows/.github/workflows/website.yaml@main
    with:
      backend_image: ghcr.io/vempain/vempain-site-backend
      frontend_image: ghcr.io/vempain/vempain-site-frontend
    secrets: inherit
```

---

## Dependabot templates

Ready-to-use Dependabot configurations live in `dependabot-templates/`.
Copy the appropriate file to `.github/dependabot.yml` in your repository.

| Template file                      | For                                                                                          |
|------------------------------------|----------------------------------------------------------------------------------------------|
| `dependabot-java-service.yaml`     | Spring Boot services (`vempain-admin-backend`, `vempain-file-backend`)                       |
| `dependabot-java-library.yaml`     | Spring Boot libraries (`vempain-auth`)                                                       |
| `dependabot-frontend-spa.yaml`     | TypeScript SPA frontends (`vempain-admin-frontend`, `vempain-file-frontend`, `vempain-site`) |
| `dependabot-frontend-library.yaml` | TypeScript libraries (`vempain-auth-frontend`)                                               |
| `dependabot-website.yaml`          | PHP + TypeScript website (`vempain-website`)                                                 |
