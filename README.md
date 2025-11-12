# web-gh-actions

This repository contains reusable GitHub Actions for web projects.

## Actions

### [`ensure-composer-package`](actions/ensure-composer-package/action.yml)

Checks if a Composer package is installed at a minimum version, and can require or upgrade it if needed.

**Inputs:**
- `package` (required): Composer package name (e.g. `symfony/console`)
- `min-version` (required): Minimum version required (e.g. `6.1.0`)
- `dev`: Require as dev dependency (`true`/`false`, default: `false`)
- `working-directory`: Directory to run composer in (default: `.`)
- `mode`: `enforce` (require/upgrade if needed), `fail` (fail if not met), or `dry-run` (default: `enforce`)

**Example usage:**
```yaml
- uses: ./actions/ensure-composer-package
  with:
    package: symfony/console
    min-version: 6.1.0
    dev: false
    mode: enforce
```
