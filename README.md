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
      - uses: actions/checkout@v6

      - uses: h3y6e/gh-skill-update-action@v1
        id: skill-update

      - name: Enable auto-merge
        if: steps.skill-update.outputs.pr-number != ''
        run: gh pr merge "${{ steps.skill-update.outputs.pr-number }}" --auto
        env:
          GH_TOKEN: ${{ github.token }}
```

This action runs `gh skill update --all` by default. If `dir` is set, it runs `gh skill update --dir <dir> --all`.

To use a GitHub App installation token instead of the workflow `GITHUB_TOKEN`, pass it with `token` and configure the required permissions on that App token.

## Inputs

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `token` | no | workflow `github.token` | GitHub token used for `gh` and pull request creation. |
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

The token used by this action must have permissions to push changes and create pull requests. When `token` is omitted, grant the workflow `GITHUB_TOKEN` the required permissions:

```yaml
permissions:
  contents: write
  pull-requests: write
```

`gh` and the `gh skill` command must be available in the runner environment.
