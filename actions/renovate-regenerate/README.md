# renovate-regenerate

Vendored from [home-operations/.github at `42a71fd`](https://github.com/home-operations/.github/tree/42a71fdfc6874da0f003ac4177f8ef48f55e262f/actions/renovate-regenerate),
under the included [Apache-2.0 license](LICENSE). The two action definitions retain
upstream's generation, artifact, commit and push steps. Local changes add a target
repository/head, optional mise tool selection and optional workflow-write permission,
and disable mise caching.

Regenerates committed files on Renovate pull requests and pushes them to the pull request branch. The Mend-hosted Renovate app cannot run `postUpgradeTasks`, so a dependency bump that feeds generated output (a chart README, a values schema, a bundled `dist/`) would otherwise fail the repository's drift check.

It is two actions, run in two jobs:

- `renovate-regenerate` checks out the pull request head, runs a mise task and uploads the changes to `paths` as a patch artifact. It never sees a write token.
- `renovate-regenerate/push` mints a GitHub App token, applies the patch and pushes the files matched by `paths` as the App's bot user.

The task runs dependency code (`npm ci`, `go run`, tool binaries Renovate just bumped), so keeping it out of the job that holds the token means nothing it leaves running can read the token.

## Central signing

The organization's `renovate-regenerate.yaml` workflow dispatches these actions in
separate jobs. It checks that the requested commit is still the head of an open,
same-repository Renovate PR against `main` before generation and again before
pushing. Only the push job receives the signing App credentials; its token is
scoped to the requested repository.

The generation job denies cache access with `cache-mode: none`, so dependency code
cannot seed caches for other workflows on `main`. Safe Settings also disables mise
caching before it receives the signing key.

The caller supplies `repository` (a name within this organization), `pull-request`,
`head-sha`, `task` and newline-separated `paths`. Optional `tools` limits mise
installation; `workflows: true` requests permission to commit generated workflows.
The task and file paths are owned by the caller. Both jobs use `$/` references,
which load these actions from the workflow run's commit without a checkout.

Target repositories must be public: generation and PR checks use the central
workflow's read-only `GITHUB_TOKEN`, which cannot read another private repository.
The caller needs its existing dispatch App credentials, scoped to Actions write
on this repository. It dispatches the workflow and waits for its result.

## Action API example

Organization callers use central signing above. This example shows the action API
inside a workflow that holds the signing credentials.

```yaml
---
name: Renovate Regenerate

on:
  pull_request:
    branches:
      - main

concurrency:
  group: ${{ github.workflow }}-${{ github.event.pull_request.number }}
  cancel-in-progress: true

permissions:
  contents: read

jobs:
  regenerate:
    if: >-
      ${{ github.event.pull_request.user.login == 'renovate[bot]'
      && github.event.pull_request.head.repo.full_name == github.repository }}
    name: Regenerate
    runs-on: ubuntu-26.04
    cache-mode: none
    permissions:
      contents: read
    outputs:
      changed: ${{ steps.regenerate.outputs.changed }}
    steps:
      - name: Regenerate
        id: regenerate
        uses: woodleighschool/.github/actions/renovate-regenerate@<sha> # renovate-regenerate-v1.0.0
        with:
          task: generate
          paths: |
            charts/*/README.md
            charts/*/values.schema.json

  push:
    if: ${{ needs.regenerate.outputs.changed == 'true' }}
    name: Push
    needs:
      - regenerate
    runs-on: ubuntu-26.04
    permissions:
      contents: read
    steps:
      - name: Push
        uses: woodleighschool/.github/actions/renovate-regenerate/push@<sha> # renovate-regenerate-v1.0.0
        with:
          client-id: ${{ secrets.BOT_CLIENT_ID }}
          private-key: ${{ secrets.BOT_APP_PRIVATE_KEY }}
          paths: |
            charts/*/README.md
            charts/*/values.schema.json
```

Reference both actions by the same commit SHA. A branch or tag reference trips zizmor's `unpinned-uses` audit in the calling repository.

## Inputs

### renovate-regenerate

| Input        | Required | Description                                                                   |
| ------------ | -------- | ----------------------------------------------------------------------------- |
| `repository` | no       | Repository name within the workflow owner; defaults to the current repository |
| `head-sha`   | no       | PR head commit; defaults to the pull-request event                            |
| `tools`      | no       | Space-separated mise tools; defaults to all configured tools                  |
| `task`       | yes      | mise task that regenerates the files                                          |
| `paths`      | yes      | Newline-separated git pathspecs of the generated files to keep                |

Output `changed` is `true` when the task changed a file matched by `paths`.

### renovate-regenerate/push

| Input                  | Required | Description                                                                     |
| ---------------------- | -------- | ------------------------------------------------------------------------------- |
| `repository`           | no       | Repository name within the workflow owner; defaults to the current repository   |
| `head-sha`             | no       | PR head commit; defaults to the pull-request event                              |
| `head-ref`             | no       | PR branch; defaults to the pull-request event                                   |
| `permission-workflows` | no       | Set to `write` for generated workflow files; otherwise no workflow permission   |
| `client-id`            | yes      | Client ID of the GitHub App that commits                                        |
| `private-key`          | yes      | Private key of the GitHub App that commits                                      |
| `paths`                | yes      | Same pathspecs as `renovate-regenerate`; anything else in the patch is left out |

## Behaviour

- The commit has to come from a GitHub App, not `GITHUB_TOKEN`. A push made with `GITHUB_TOKEN` does not start a new workflow run, so the required check would never report on the regenerated commit.
- Renovate treats a branch with commits from another author as modified and stops rebasing it. The shared Renovate preset lists the App's bot email in `gitIgnoredAuthors` so it keeps rebasing.
- Renovate rebuilds its branch from the base branch when it rebases, dropping the regenerated commit, so the workflow runs again and pushes a fresh one. A run on an already regenerated head finds nothing to change and stops.
- Set `paths` to what the repository's drift check compares, and pass the same list to both actions. Other files the task touches, such as a mise lockfile, are not committed. Every pathspec has to match a file in the repository, or `git add` fails.
- The push job stages only its own `paths` from the patch, because the patch comes from the job that ran dependency code, which could have replaced it.
- The commit is built on the pull request's head SHA and pushed without force. If Renovate pushed in the meantime, the push is rejected and the run for the new head takes over.
