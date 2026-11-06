# .github

Org-wide defaults and shared automation for [woodleighschool](https://github.com/woodleighschool), based on [home-operations/.github](https://github.com/home-operations/.github).

## Composite actions

| Action                                               | Purpose                                                       |
| ---------------------------------------------------- | ------------------------------------------------------------- |
| [`apple-notarize`](actions/apple-notarize)           | Notarizes and staples a signed macOS product                  |
| [`apple-pkg`](actions/apple-pkg)                     | Builds a signed macOS installer from a destination root       |
| [`renovate-regenerate`](actions/renovate-regenerate) | Regenerates files and pushes them from a separate signing job |

Each action has its own README, changelog and `<component>-v<version>` tag. Reference actions by commit SHA in calling repositories.

## Renovate

Repositories extend `local>woodleighschool/.github:renovate-config`, the preset in [`renovate-config.json`](renovate-config.json). Rules for a single repository belong in its own `.renovaterc.json5`.
