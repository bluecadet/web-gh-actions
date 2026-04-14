# Ensure Composer Package Action

A GitHub Action that validates a specific Composer package is installed at a minimum required version in your CI environment. Optionally auto-installs the package if missing.

## Use Case

This action is useful in CI pipelines where you:
- Want to enforce a minimum dependency version across multiple projects
- Can't rely on all developers having the dependency in their `composer.json`
- Need automatic fallback installation if the package is missing

## Inputs

### `package` (required)

The Composer package namespace to check/ensure (e.g., `bluecadet/bc_composer_ci_tools`, `drupal/coder`).

### `min-version` (required)

Minimum version constraint, using standard Composer version syntax:
- `1.x-dev` — Any 1.x development version
- `1.0.0` — Exact version or higher
- `^1.0` — Compatible version (semver caret range)
- `~1.0` — Approximately this version

### `dev` (optional, default: `"false"`)

Whether the package is a dev dependency. Set to `"true"` for packages installed with `--dev`.

Options: `"true"` or `"false"`

### `working-directory` (optional, default: `"."`)

Directory where to run composer commands (relative to workspace root).

### `mode` (optional, default: `"enforce"`)

How to handle validation failures:
- `enforce` — Fail the job if package is missing or below min version
- `warn` — Continue job but print a warning (non-blocking)

### `ensure-install` (optional, default: `"false"`)

Auto-install package behavior:
- `auto` — Install if not found or below min version
- `true` — Always run `composer require` (even if already installed)
- `false` — Check only, never install

## Outputs

None. The action either succeeds (package installed at min version) or fails/warns based on `mode`.

## Examples

### Basic usage with auto-install

Ensure `bluecadet/bc_composer_ci_tools` is installed at `1.x-dev`, auto-installing if missing:

```yaml
- name: Enforce bc_composer_ci_tools
  uses: bluecadet/web-gh-actions/actions/ensure-composer-package@v1.0.1
  with:
    package: bluecadet/bc_composer_ci_tools
    min-version: 1.x-dev
    dev: "true"
    ensure-install: "auto"
```

### Enforce mode without auto-install

Check that `drupal/coder ^8.3.0` is installed, fail if missing:

```yaml
- name: Verify coder is installed
  uses: bluecadet/web-gh-actions/actions/ensure-composer-package@v1.0.1
  with:
    package: drupal/coder
    min-version: "^8.3.0"
    dev: "true"
    ensure-install: "false"
    mode: "enforce"
```

### Warn mode for optional checks

Check a package but don't block the job if missing:

```yaml
- name: Check optional package (non-blocking)
  uses: bluecadet/web-gh-actions/actions/ensure-composer-package@v1.0.1
  with:
    package: some/optional-pkg
    min-version: "^2.0"
    dev: "true"
    mode: "warn"
    ensure-install: "false"
```

## Common Issue: Double Backslash in Report Class Names

When used with PHPCS report classes like `Bluecadet\PHPCS\Report\MarkdownGithub`, ensure the namespace is properly escaped in YAML:

```bash
# Correct (double backslash)
./vendor/bin/phpcs --report=Bluecadet\\PHPCS\\Report\\MarkdownGithub

# Incorrect (single backslash)
./vendor/bin/phpcs --report=Bluecadet\PHPCS\Report\MarkdownGithub  # ❌ Will fail
```

## Related Packages

- [`bluecadet/bc_composer_ci_tools`](https://github.com/bluecadet/bc_composer_ci_tools) — Custom PHPCS report formatter for GitHub Actions
