# Hands-On 2 - Developing GitHub Actions in VS Code

Goal: install the GitHub Actions extension and build a workflow using only IntelliSense/autocomplete.

## Steps

### 1. Verify the extension

1. Confirm the **GitHub Actions** extension is installed and you're signed in (see [01-Codespaces-Setup.md](01-Codespaces-Setup.md) if not).
2. Open your repository folder in VS Code so the extension can detect `.github/workflows`.

### 2. Create a workflow using only autocomplete

1. Create the folder `.github/workflows/` if it doesn't exist yet.
2. Create a new file `ci.yml` inside it.
3. Without copy-pasting, use `Cmd+Space` to trigger suggestions and build the following, letting IntelliSense fill in the keys:
   - A `name` for the workflow
   - An `on` trigger for `push` (any branch) and `pull_request` (targeting `main`) and `workflow_dispatch`
   - A job named `build` running on `ubuntu-latest` with steps: `actions/checkout@v7`, then a step that prints `github.actor` and `github.event_name` and exposes both as job `outputs` (e.g. `actor` and `event_name`)
   - A second job named `deploy` that:
     - only runs after `build` has completed successfully
     - runs on `ubuntu-latest`
     - deploys to an `environment` named `test` (just reference the name - no need to create the environment in the repo settings first)
     - has a single step that runs `echo "Deploying to test environment"`
4. Hover over `actions/checkout@v7` to confirm the extension shows the action's description and inputs.
5. Intentionally introduce an error (e.g. misspell `runs-on` as `run-on`) and confirm the extension flags it with a red squiggle before you save.
6. Fix the error, save the file.

<details>
  <summary>Solution</summary>

```yaml
name: GitHub Actions Demo

on:
  push:
    branches: ["**"]
  pull_request:
    branches: [main]
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      actor: ${{ steps.info.outputs.actor }}
      event_name: ${{ steps.info.outputs.event_name }}
    steps:
      - uses: actions/checkout@v7
      - id: info
        run: |
          echo "Triggered by: ${{ github.actor }}"
          echo "Event: ${{ github.event_name }}"
          echo "actor=${{ github.actor }}" >> "$GITHUB_OUTPUT"
          echo "event_name=${{ github.event_name }}" >> "$GITHUB_OUTPUT"

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment: test
    steps:
      - run: echo "Deploying to test environment"
```

</details>

### 3. Trigger and inspect a run

1. Commit and push the workflow file.
2. Open the **GitHub Actions** panel in the sidebar, find your workflow, and confirm the push triggered a run.
3. If your workflow also supports `workflow_dispatch`, add that trigger and use the panel's **Run workflow** action to trigger it manually.
4. Click into the run from the panel and open the logs for both the `build` and `deploy` jobs directly inside VS Code.

### 4. Create a custom deploy action in the same repo

1. Create the folder `.github/actions/deploy/` and add an `action.yml` file inside it.
2. Using `Cmd+Space` autocomplete, define a **composite** action with:
   - A `name` and `description`
   - An input `environment` (required, with a `description`)
   - An output `url` (with a `description`), sourced from a step output
   - A single `run` step (`shell: bash`) that writes a fake URL to `$GITHUB_OUTPUT` based on the `environment` input
3. Update the `deploy` job in `ci.yml` to call your action with `uses: ./.github/actions/deploy`, pass `environment: test` as an input, give the step an `id`, then add a following step that echoes the action's `url` output.
4. Hover over `uses: ./.github/actions/deploy` in `ci.yml` and confirm the GitHub Actions extension shows the description, inputs, and outputs read straight from your local `action.yml`.
5. Commit, push, and trigger a run - confirm the deployed URL shows up in the logs.

<details>
  <summary>Solution</summary>

`.github/actions/deploy/action.yml`:

```yaml
name: Deploy
description: Deploys the application to an environment
inputs:
  environment:
    description: The environment to deploy to
    required: true
outputs:
  url:
    description: The URL of the deployed environment
    value: ${{ steps.set-url.outputs.url }}
runs:
  using: composite
  steps:
    - id: set-url
      shell: bash
      run: echo "url=https://${{ inputs.environment }}.example.com" >> "$GITHUB_OUTPUT"
```

`ci.yml` `deploy` job:

```yaml
  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment: test
    steps:
      - uses: actions/checkout@v7
      - id: deploy
        uses: ./.github/actions/deploy
        with:
          environment: test
      - run: echo "Deployed to ${{ steps.deploy.outputs.url }}"
```

</details>

## Checkpoint

- [ ] Extension installed and signed in
- [ ] Workflow authored using autocomplete only
- [ ] `build` and `deploy` jobs created, with `deploy` referencing the `test` environment
- [ ] `build` job prints and outputs `github.actor` and `github.event_name`
- [ ] Validation error observed and fixed
- [ ] Run triggered and logs viewed from within VS Code
- [ ] Custom `deploy` action created with an `environment` input and a `url` output
- [ ] Action metadata (description, inputs, outputs) shown correctly on hover in VS Code
- [ ] Deployed URL displayed in the workflow run logs
