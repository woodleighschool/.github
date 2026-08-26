# .github

Org-wide defaults and shared automation for [woodleighschool](https://github.com/woodleighschool), based on [home-operations/.github](https://github.com/home-operations/.github).

## Composite actions

| Action                                     | Purpose                                                 |
| ------------------------------------------ | ------------------------------------------------------- |
| [`apple-notarize`](actions/apple-notarize) | Notarizes and staples a signed macOS product            |
| [`apple-pkg`](actions/apple-pkg)           | Builds a signed macOS installer from a destination root |

Each action has its own README, changelog and `<component>-v<version>` tag. Reference actions by commit SHA in calling repositories.
