# GitHub Actions Deep Dive - Day 2 Hands-On Template

This repository is the **template repo** for the Day 2 ("Actions Development Deep Dive") hands-on labs: Codespaces, VS Code, `gh`, and `act`.

> This repo is meant to be marked as a **GitHub template repository** (Settings &rarr; check "Template repository"). Each participant creates their own copy from it - never work directly in this repo.

## Getting your own copy

1. On GitHub, click **Use this template** &rarr; **Create a new repository** and pick your own account/namespace.
   (CLI alternative: `gh repo create my-actions-deepdive --template spindev-actions-workshop/actions-deepdive --private --clone`)
2. Continue with [Hands-On Lab 01](#hands-on-labs--time-estimates) below.

## Hands-On Labs & time estimates

| # | Lab | Duration |
|---|-----|----------|
| 01 | [Codespaces Setup](hol/01-Codespaces-Setup.md) (create your copy, open a Codespace, verify the toolchain, connect local VS Code to it) | ~20 min |
| 02 | [Developing GitHub Actions in VS Code](hol/02-VSCode-Actions-Extension.md) (GitHub Actions extension, IntelliSense, job outputs, a custom deploy action) | ~25 min |
| 03 | [GitHub CLI Deep Dive](hol/03-GitHub-CLI.md) (`gh auth`, `gh repo create`, `gh workflow`, `gh run`, `gh api`) | ~20 min |
| 04 | [Running Workflows Locally with `act`](hol/04-Act-Local.md) (CLI runs, secrets/inputs, the GitHub Local Actions extension) | ~25-30 min |

## Contents

- `hol/` - the hands-on labs listed above

