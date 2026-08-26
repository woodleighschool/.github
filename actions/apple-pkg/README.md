# apple-pkg

Builds a signed macOS installer package from a destination-root hierarchy.

The calling repository owns the installed filesystem under `packaging/root/` and package scripts under `packaging/scripts/`. The action stages that root, optionally adds a signed app under `/Applications`, and runs `pkgbuild` with the native recommended ownership behaviour.

## Usage

```yaml
- name: Build package
  id: package
  uses: woodleighschool/.github/actions/apple-pkg@<sha> # apple-pkg-v1.0.0
  with:
    app: ${{ runner.temp }}/export/Example.app
    identifier: au.edu.vic.woodleigh.Example
    version: ${{ steps.release.outputs.version }}
    signing-identity: ${{ env.PKG_SIGNING_IDENTITY }}
    output: dist/Example-${{ steps.release.outputs.version }}.pkg
```

For a non-app package, prepare a complete destination root and pass it with `payload-root`:

```yaml
with:
  payload-root: ${{ runner.temp }}/package-root
  identifier: au.edu.vic.woodleigh.example
  version: ${{ steps.release.outputs.version }}
  signing-identity: ${{ env.PKG_SIGNING_IDENTITY }}
  output: dist/example-${{ steps.release.outputs.version }}.pkg
```

Reference the action by commit SHA. A branch or tag reference trips zizmor's `unpinned-uses` audit in the calling repository.

## Inputs

| Input              | Required | Default             | Description                                          |
| ------------------ | -------- | ------------------- | ---------------------------------------------------- |
| `app`              | No       |                     | Signed app bundle installed under `/Applications`    |
| `payload-root`     | No       | `packaging/root`    | Destination-root hierarchy overlaid into the package |
| `scripts`          | No       | `packaging/scripts` | Package scripts directory, when it exists            |
| `identifier`       | Yes      |                     | Package identifier                                   |
| `version`          | Yes      |                     | Package version                                      |
| `signing-identity` | Yes      |                     | Developer ID Installer identity used by `pkgbuild`   |
| `output`           | Yes      |                     | Output package path                                  |

At least one of `app` or an existing `payload-root` is required.

## Output

`path` is the signed package path supplied through `output`.

## Behaviour

- The action requires a macOS runner with `codesign`, `ditto`, and `pkgbuild` available.
- It verifies an app before copying it and does not alter its signature.
- It copies with `ditto --norsrc`, excluding resource forks, extended attributes, ACLs, and quarantine metadata.
- It leaves filesystem layout, normal file modes, executable script bits, and unusual permission requirements to the calling repository.
- Notarization and final package verification remain separate steps.
