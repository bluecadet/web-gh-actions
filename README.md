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
