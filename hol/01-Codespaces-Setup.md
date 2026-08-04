# Hands-On 1 - Codespaces Setup

Goal: create your own copy of the `actions-deepdive` template repository and get a working Codespace with the full toolchain (VS Code, GitHub Actions extension, `gh`, `act`, Docker) - no local installs required.

## Steps

### 1. Create your own copy of the template repo

1. Open the `actions-deepdive` template repository (link shared by the trainer).
2. Click **Use this template** &rarr; **Create a new repository**.
3. Choose your own account/namespace, give it any name (e.g. `my-actions-deepdive`), and create it.
   - CLI alternative: `gh repo create my-actions-deepdive --template <org>/actions-deepdive --private --clone`

### 2. Open a Codespace

1. On your new repository, go to **Code** &rarr; **Codespaces** &rarr; **Create codespace on main**.
2. Wait for the container to build - this can take a minute the first time.

### 3. Verify the toolchain

1. **Docker**: install the **Docker** extension (publisher Microsoft) from the Extensions view (`Ctrl+Shift+X` / `Cmd+Shift+X`), then verify the daemon works:
   ```bash
   docker run hello-world
   ```
2. **GitHub Actions extension**: search **GitHub Actions** (publisher GitHub), install. Signing in usually happens automatically using your Codespaces GitHub identity; otherwise sign in manually from the new sidebar panel.
3. **GitHub CLI**: already preinstalled in Codespaces - confirm you're authenticated:
   ```bash
   gh --version
   gh auth status
   ```
4. **act**: install it as a `gh` CLI extension instead of a standalone binary:
   ```bash
   gh extension install https://github.com/nektos/gh-act
   ```
5. Verify with a run, choosing the **medium** runner image when prompted:
   ```bash
   gh act --version
   ```
6. **GitHub Local Actions** extension: install this VS Code extension for a UI on top of act - browse workflows/jobs and trigger `act` runs without leaving the editor.
   > Note: nested containers in Codespaces can be slower and occasionally hit limitations that don't exist on a local Docker install - if you hit issues in later exercises, pair with someone running act locally.

### 4. Connect your local VS Code to the Codespace

Instead of working in the browser, you can connect your locally installed VS Code to the same Codespace container - your files, terminal, extensions, and Docker daemon all run remotely, so nothing changes except where the editor UI runs.

1. In your local VS Code, install the **GitHub Codespaces** extension (publisher GitHub) from the Extensions view.
2. Open the Command Palette (`Ctrl+Shift+P` / `Cmd+Shift+P`) and run **Codespaces: Sign in** if you're not already signed in with the same GitHub account.
3. Run **Codespaces: Connect to Codespace...** and pick the Codespace you created in step 2.
   - CLI alternative: from a local terminal, `gh codespace code` opens local VS Code connected to a Codespace you pick.
4. Wait for the new window to connect - the status bar in the bottom-left corner shows the Codespace name once connected.
5. Open a terminal in this window (`` Ctrl+` ``) and confirm it's running inside the Codespace, not on your local machine, by re-running the same checks from step 3:
   ```bash
   docker run hello-world
   gh auth status
   ```
6. Note that any extensions installed while connected (e.g. **GitHub Actions**, **Docker**) install into the remote Codespace, not your local machine - they'll be there next time you connect from anywhere.

## Checkpoint

- [ ] Own copy of the template repository created
- [ ] Codespace created and running
- [ ] `docker run hello-world` succeeds
- [ ] GitHub Actions extension installed and signed in
- [ ] `gh auth status` shows you as logged in
- [ ] `gh act --version` runs (act installed via the `gh-act` extension)
- [ ] Local VS Code connected to the Codespace and a terminal command run successfully from it
