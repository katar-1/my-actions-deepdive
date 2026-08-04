# Hands-On 4 - Running Workflows Locally with act

Goal: install `act`, run existing workflows locally, and simulate events with secrets/inputs.

## Steps

### 1. Discover jobs

From the root of a repository containing `.github/workflows` (e.g. the workflow from Exercise 2):

```bash
gh act -l
```

Confirm the job(s) and the events that would trigger them are listed correctly.

### 2. Run the default event

```bash
gh act
```

- The first run will pull a runner image - this may take a while and use significant disk space.
- If prompted, pick the **medium** image size unless you have a specific reason to choose otherwise.

### 3. Run a specific job or event

```bash
gh act -j build
gh act pull_request
```

### 4. Provide secrets and environment variables

1. Create a local `.secrets` file (add it to `.gitignore`!):

   ```
   MY_TOKEN=dummy-value
   ```

2. Create a local `.env` file with a non-secret variable, e.g.:

   ```
   GREETING=hello from the .env file
   ```

3. Add a step to the `build` job in `ci.yml` that echoes both, so you can see how each is wired up:

   ```yaml
   - name: Show secrets and env vars
     env:
       MY_TOKEN: ${{ secrets.MY_TOKEN }}
     run: |
       echo "Secret (should be masked in the log): $MY_TOKEN"
       echo "Env var from .env file: $GREETING"
   ```

4. Run with both files:

   ```bash
   gh act --secret-file .secrets --env-file .env
   ```

5. Confirm in the output that `MY_TOKEN` is masked (`***`) since it came through the `secrets` context, while `GREETING` prints in full since `.env` values are plain environment variables, not secrets.

<details>
  <summary>Solution</summary>

```yaml
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
      - name: Show secrets and env vars
        env:
          MY_TOKEN: ${{ secrets.MY_TOKEN }}
        run: |
          echo "Secret (should be masked in the log): $MY_TOKEN"
          echo "Env var from .env file: $GREETING"
```

</details>

### 5. Simulate a `workflow_dispatch` event with the `environment` input

1. Add a `workflow_dispatch` trigger to `ci.yml` with a required `environment` input:

   ```yaml
   workflow_dispatch:
     inputs:
       environment:
         description: Environment to deploy to
         required: true
         type: string
   ```

2. Update the `deploy` job to use that input instead of the hardcoded `test` value - both for the job's `environment:` and for the input passed to your custom deploy action.
3. Create an `event.json` file describing the event payload:

   ```json
   {
     "inputs": {
       "environment": "staging"
     }
   }
   ```

4. Run it:

   ```bash
   gh act workflow_dispatch -e event.json
   ```

5. Confirm in the logs that the `deploy` job's environment and the echoed URL both reflect `staging`.

<details>
  <summary>Solution</summary>

```yaml
on:
  push:
    branches: ["**"]
  pull_request:
    branches: [main]
  workflow_dispatch:
    inputs:
      environment:
        description: Environment to deploy to
        required: true
        type: string

jobs:
  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}
    steps:
      - uses: actions/checkout@v7
      - id: deploy
        uses: ./.github/actions/deploy
        with:
          environment: ${{ inputs.environment }}
      - run: echo "Deployed to ${{ steps.deploy.outputs.url }}"
```

</details>

### 6. Trigger a run using the GitHub Local Actions extension

The workflow already has everything it needs from step 5 - now trigger a run through a UI instead of the CLI and watch it execute.

1. Save `ci.yml` if you haven't already - `act` reads the workflow straight from disk, no commit/push needed for a local run.
   > The **GitHub Local Actions** extension shells out to an `act` binary directly. Since
   > `act` was installed via the `gh-act` extension, point the extension's binary path
   > setting at the `gh-act` install location (or install a standalone `act` binary too)
   > if it can't find `act` on its own.
2. Open the **GitHub Local Actions** panel in the sidebar and expand `ci.yml`.
3. The extension's input form for `workflow_dispatch` events isn't available here - instead, select the `push` trigger and start the run from there.
4. Watch the run's execution stream live in the panel as the `build` and `deploy` jobs progress.
5. Note that since this run wasn't triggered by `workflow_dispatch`, `inputs.environment` is empty - the `deploy` job's environment and the echoed URL will both come back blank, unlike the CLI run from step 5.

## Checkpoint

- [ ] `gh act -l` lists the expected jobs
- [ ] A workflow ran successfully in a local container
- [ ] `.secrets` and `.env` files supplied, with the workflow echoing both and the secret masked in the log
- [ ] A `workflow_dispatch` event simulated with the `environment` input via `event.json`
- [ ] A `push` run started from the GitHub Local Actions extension and its execution watched live in the panel
