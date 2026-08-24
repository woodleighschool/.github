# apple-notarize

Notarizes and staples a signed macOS app or installer package using an App Store Connect API key.

## Usage

Run the action on macOS after creating a Developer ID-signed product:

```yaml
- name: Notarize app
  uses: woodleighschool/.github/actions/apple-notarize@<sha> # apple-notarize-v2.0.0
  with:
    product-path: ${{ runner.temp }}/export/Example.app
    issuer-id: ${{ vars.APPSTORE_ISSUER_ID }}
    api-key-id: ${{ vars.APPSTORE_API_KEY_ID }}
    api-private-key: ${{ secrets.APPSTORE_API_PRIVATE_KEY }}
```

Reference the action by commit SHA. A branch or tag reference trips zizmor's `unpinned-uses` audit in the calling repository.

## Inputs

| Input             | Description                                            |
| ----------------- | ------------------------------------------------------ |
| `product-path`    | Path to a signed macOS app bundle or installer package |
| `issuer-id`       | App Store Connect API issuer ID                        |
| `api-key-id`      | App Store Connect API key ID                           |
| `api-private-key` | App Store Connect API private key text                 |

## Behaviour

- The action requires a macOS runner with Xcode's `notarytool` and `stapler` available.
- It submits apps in a temporary ZIP and submits installer packages directly.
- It staples and validates the ticket on the original product.
- Signing, provisioning, final signature checks and distribution packaging stay in the calling workflow.
