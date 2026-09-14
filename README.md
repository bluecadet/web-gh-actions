# web-gh-actions

This repository contains reusable GitHub Actions for web projects.

## Available Actions

### [`ensure-composer-package`](actions/ensure-composer-package/action.yml)

Enforce minimum Composer package versions and optionally auto-install missing dependencies in CI.

**Typical use:** Ensuring teams have the required version of shared CI tools without bloating every project's `composer.json`.

**Inputs:**
- `package` (required): Composer package name (e.g. `symfony/console`)
- `min-version` (required): Minimum version required (e.g. `6.1.0`)
- `dev`: Require as dev dependency (`true`/`false`, default: `false`)
- `working-directory`: Directory to run composer in (default: `.`)
- `mode`:
  - `enforce`: Require/upgrade if needed (default)
  - `fail`: Fail if not met
  - `dry-run`: Only check, do not modify

**Example usage:**
```yaml
- uses: ./actions/ensure-composer-package
  with:
    package: symfony/console
    min-version: 6.1.0
    dev: false
    mode: enforce
```

See [`actions/ensure-composer-package/README.md`](./actions/ensure-composer-package/README.md) for full documentation.

---

## Reusable Workflows

### [`drupal-test-runner.yml`](.github/workflows/drupal-test-runner.yml)

Runs PHPCS, PHPStan, and PHPUnit (with coverage) for a single Drupal core / PHP / MariaDB combination against a Drupal module. Generic across any Bluecadet Drupal module -- it derives the module name from `$GITHUB_REPOSITORY` and auto-discovers submodule `tests/src` directories.

Not called directly by module repos -- see `drupal-tests-and-standards.yml` below, which drives it.

### [`drupal-tests-and-standards.yml`](.github/workflows/drupal-tests-and-standards.yml)

Orchestrates `drupal-test-runner.yml` across a matrix, so the matrix-management logic (which Drupal core / PHP / MariaDB combos to test, PR vs. scheduled/push behavior) lives here once instead of being copy-pasted into every module.

GitHub doesn't allow a `push`/`pull_request`/`schedule` trigger to live outside the repo it watches, so each module still needs a small **stub** workflow. Everything else -- the matrix, the exclude rules, which jobs run for which event -- is read from a config file (`.github/drupal-ci.yml` by default) in the calling module's own repo.

**1. Add `.github/drupal-ci.yml` to the module:**

```yaml
phpunit_target: tests/src

pr_matrix:
  drupal_core: ['10.6.x', '11.3.x']
  php_version: ['8.2', '8.3', '8.4']
  mariadb_version: ['10.6']
  exclude:
    - drupal_core: '10.6.x'
      php_version: '8.4'
    - drupal_core: '11.3.x'
      php_version: '8.2'

full_matrix:
  drupal_core: ['10.5.x', '10.6.x', '11.2.x', '11.3.x']
  php_version: ['8.2', '8.3', '8.4']
  mariadb_version: ['10.4', '10.6']
  exclude:
    - drupal_core: '10.5.x'
      php_version: '8.4'
    - drupal_core: '11.2.x'
      php_version: '8.2'
```

`pr_matrix` runs on pull requests; `full_matrix` runs on `push` and the monthly `schedule`. Both accept any keys valid under a GitHub Actions `strategy.matrix` block (arrays plus an optional `exclude` list).

**2. Add the stub workflow to the module** (`.github/workflows/drupal-tests-and-standards.yml`):

```yaml
name: Module tests and standards
on:
  workflow_dispatch:
    inputs:
      drupal_core:
        required: true
        default: '11.2.x'
        type: string
      php_version:
        required: true
        default: '8.4'
        type: string
      mariadb_version:
        required: true
        default: '10.6'
        type: string
      phpunit_target:
        required: true
        default: 'tests/src'
        type: string
  schedule:
    - cron: '0 6 1 * *'
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
jobs:
  ci:
    permissions:
      contents: read
      pull-requests: write
    uses: bluecadet/web-gh-actions/.github/workflows/drupal-tests-and-standards.yml@v1.x
    with:
      event_name: ${{ github.event_name }}
      dispatch_drupal_core: ${{ inputs.drupal_core }}
      dispatch_php_version: ${{ inputs.php_version }}
      dispatch_mariadb_version: ${{ inputs.mariadb_version }}
      dispatch_phpunit_target: ${{ inputs.phpunit_target }}
```

The `event_name` input has to be forwarded explicitly -- inside a called workflow, `github.event_name` always reads `workflow_call`, not the event that triggered the caller.

Pin `@v1.x` to whatever this repo's current release tag is; don't track a branch directly, so a bad change here can't break every module's CI at once.

**Tagging note:** always cut releases as a **lightweight tag** (`git tag vX.Y.Z <sha>`), not an annotated tag (`git tag -a`). `drupal-tests-and-standards.yml` calls `drupal-test-runner.yml` via a relative `./...` path -- when the outer `@vX.Y.Z` ref is an annotated tag, GitHub resolves the top-level call fine but fails to dereference the tag object for that *nested* relative call ("workflow was not found"), even though the file is right there in the tagged commit. Confirmed 2026-09-14 while piloting on `bluecadet_utilities`.

## Usage

To use an action from this repository in your workflow, reference it with a relative path:

```yaml
jobs:
  composer-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: ./actions/ensure-composer-package
        with:
          package: symfony/console
          min-version: 6.1.0
```

## Contributing

1. Fork the repository and create your branch.
2. Add or update actions in the `actions/` directory.
3. Update this README with documentation for new actions.
4. Open a pull request.

## License

MIT
