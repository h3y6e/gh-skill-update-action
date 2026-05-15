# gh-skill-update-action

Run `gh skill update` and create a pull request if needed.

## Usage

```yaml
name: update skills

on:
  schedule:
    - cron: "0 0 * * *"
  workflow_dispatch:

permissions:
  contents: write
  pull-requests: write

jobs:
  update-skills:
    runs-on: ubuntu-slim
    timeout-minutes: 5

    steps:
      - uses: actions/create-github-app-token@v3
        id: app-token
        with:
          app-id: ${{ secrets.APP_ID }}
          private-key: ${{ secrets.PRIVATE_KEY }}

      - uses: actions/checkout@v6
        with:
          token: ${{ steps.app-token.outputs.token }}

      - uses: h3y6e/gh-skill-update-action@v1
        id: skill-update
        with:
          token: ${{ steps.app-token.outputs.token }}

      - name: Enable auto-merge
        if: steps.skill-update.outputs.pr-number != ''
        run: gh pr merge "${{ steps.skill-update.outputs.pr-number }}" --auto
        env:
          GH_TOKEN: ${{ steps.app-token.outputs.token }}
```

This action runs `gh skill update --all` by default. If `dir` is set, it runs `gh skill update --dir <dir> --all`.

## Inputs

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `token` | yes | | GitHub token used for `gh` and pull request creation. |
| `dir` | no | | Directory containing skills. When empty, `gh skill update` uses its default scan behavior. |
| `branch` | no | `gh-skill-update-action/patch` | The pull request branch name. |
| `commit-message` | no | `[gh-skill-update-action] automated change` | The message to use when committing changes. |
| `title` | no | `Changes by gh-skill-update-action` | The title of the pull request. |
| `body` | no | `Automated changes by gh-skill-update-action` | The body of the pull request. |

## Outputs

| Name | Description |
| --- | --- |
| `pr-number` | The pull request number. Empty when no pull request was created or updated. |
| `pr-url` | The pull request URL. Empty when no pull request was created or updated. |

## Requirements

The caller workflow must check out the repository before using this action.

The token must have permissions to push changes and create pull requests.

Recommended permissions:

```yaml
permissions:
  contents: write
  pull-requests: write
```

`gh` and the `gh skill` command must be available in the runner environment.
