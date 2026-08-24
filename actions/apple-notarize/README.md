# apple-notarize

Notarizes and staples a signed macOS app using an App Store Connect API key.

## Usage

Run the action on macOS after exporting a Developer ID-signed app:

```yaml
- name: Notarize app
  uses: woodleighschool/.github/actions/apple-notarize@<sha> # apple-notarize-v1.0.0
  with:
    app-path: ${{ runner.temp }}/export/Example.app
    issuer-id: ${{ vars.APPSTORE_ISSUER_ID }}
    api-key-id: ${{ vars.APPSTORE_API_KEY_ID }}
    api-private-key: ${{ secrets.APPSTORE_API_PRIVATE_KEY }}
```

Reference the action by commit SHA. A branch or tag reference trips zizmor's `unpinned-uses` audit in the calling repository.

## Inputs

| Input             | Description                            |
| ----------------- | -------------------------------------- |
| `app-path`        | Path to the signed macOS app bundle    |
| `issuer-id`       | App Store Connect API issuer ID        |
| `api-key-id`      | App Store Connect API key ID           |
| `api-private-key` | App Store Connect API private key text |

## Behaviour

- The action requires a macOS runner with Xcode's `notarytool` and `stapler` available.
- It submits a temporary ZIP containing the app and waits for Apple to accept it.
- It staples and validates the ticket on the original app bundle.
- Signing, provisioning, final signature checks and distribution packaging stay in the calling workflow.
