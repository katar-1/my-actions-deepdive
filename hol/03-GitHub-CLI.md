# Hands-On 3 - GitHub CLI Deep Dive

Goal: authenticate with `gh`, drive workflows from the terminal, and query the API directly.

## Prerequisites

- The workflow (`ci.yml`, with `build` and `deploy` jobs) created in [02-VSCode-Actions-Extension.md](02-VSCode-Actions-Extension.md)

## Steps

### 1. Authenticate

```bash
gh auth status
gh auth token   # prints the active token - keep it secret, don't paste it anywhere
```

- Note which scopes are granted (e.g. `repo`, `workflow`).
- If `workflow` scope is missing, re-run `gh auth login` and select it.

> **Codespaces gotcha:** if `gh auth status` reports you're not logged in, or `gh auth login`
> warns that `GITHUB_TOKEN` is being used for authentication, an environment variable is
> overriding your stored credentials. This auto-injected token is repo-scoped and can't
> create repos or dispatch workflows, causing `Resource not accessible by integration`
> errors later on. Fix it with:
>
> ```bash
> unset GITHUB_TOKEN
> gh auth login --web -h github.com
> ```
>
> If it keeps reappearing in new terminals, it's set as a Codespaces secret - check
> repo → **Settings → Secrets and variables → Codespaces**, or your account's
> [Codespaces secrets](https://github.com/settings/codespaces).

### 2. Create a repository with the `gh repo create` wizard

1. Run the command with no flags and follow the interactive wizard:
   ```bash
   gh repo create
   ```
   - Choose to create a new repository from scratch (or from a template), give it a name (e.g. `my-actions-deepdive-demo`), and pick a visibility - this just creates the repo on GitHub, no local clone needed.
2. Alternatively, skip the wizard and pass flags directly:
   ```bash
   gh repo create my-actions-deepdive-demo --public
   ```
3. Confirm it was created:
   ```bash
   gh repo view my-actions-deepdive-demo
   ```

### 3. Explore workflows and runs

```bash
gh workflow list
gh workflow view "GitHub Actions Demo"
gh run list --limit 5
```

### 4. Trigger a run and watch it live

```bash
# trigger a workflow_dispatch run (add -f <name>=<value> if your workflow defines inputs)
gh workflow run "GitHub Actions Demo"

# find the run that was just queued
gh run list --limit 1

# stream logs until it finishes
gh run watch <run-id>

# view full logs afterwards, and confirm the deploy job's URL output is in there
gh run view <run-id> --log
```

### 5. Call the REST API directly

```bash
# basic GET
gh api repos/{owner}/{repo}

# list the last 5 workflow runs, filtered with jq
gh api repos/{owner}/{repo}/actions/runs --paginate | jq '.workflow_runs[0:5] | .[].status'

# structured output via built-in --json/--jq (no gh api needed)
gh run list --json databaseId,status,conclusion --jq '.[0]'
```

### 6. Bonus: GraphQL

```bash
gh api graphql -f query='query { viewer { login } }'
```

GraphQL lets you fetch exactly the fields you need in one request instead of multiple
REST calls. For example, get the current user plus their 5 most recently pushed
repositories, each with its own selected fields:

```bash
gh api graphql -f query='
  query {
    viewer {
      login
      name
      repositories(first: 5, orderBy: {field: PUSHED_AT, direction: DESC}) {
        nodes {
          nameWithOwner
          isPrivate
          stargazerCount
          defaultBranchRef {
            name
          }
        }
      }
    }
  }
'
```

Notice how the shape of the response mirrors the shape of the query - only the fields
you list (`login`, `name`, `nameWithOwner`, etc.) are returned, unlike REST endpoints
which send the full object.

## Checkpoint

- [ ] Authenticated and verified scopes
- [ ] Repository created on GitHub with `gh repo create`
- [ ] Triggered a `workflow_dispatch` run from the CLI
- [ ] Watched a run live and viewed its logs
- [ ] Queried run data with `gh api` + `jq`
