# CanadaLogin release-notes devcontainer

This dev container provides the read-only tooling used by the `canadalogin-release-notes` skill.

## What's included

Built on `mcr.microsoft.com/devcontainers/base:bookworm`, which already ships Git, `curl`, and common shell utilities. On top of the base image the container installs:

- **GitHub CLI (`gh`)** — repository history and metadata over the GitHub API.
- **OpenSSH client** — optional SSH access to GitHub as a fallback when HTTPS cloning fails. A `~/.ssh/config` entry for `github.com` with `StrictHostKeyChecking accept-new` is created for the `vscode` user.
- **`jq`** — parsing `.deployed_versions/*.json` files.
- **Chromium** (headless-capable) plus `fonts-liberation` and `fonts-noto-color-emoji` — HTML-to-PDF rendering of the generated reports. `$BROWSER` is set to `/usr/bin/chromium`.
- **Python 3** with `pip` and `venv` — helper scripts and virtual environments.

Container environment:

- `GIT_TERMINAL_PROMPT=0` prevents interactive Git credential prompts, so a failed HTTPS clone fails fast instead of hanging.
- `safe.directory` is preconfigured for the mounted workspace via `GIT_CONFIG_*` env vars.
- The `postCreateCommand` sets `init.defaultBranch=main` and `pull.ff=only` globally for the `vscode` user.

The workspace is mounted at `/workspaces/${localWorkspaceFolderBasename}` (for this repository, `/workspaces/canadalogin-release-notes-generator-poc`).

## Use in VS Code

1. Have Docker running locally (Docker Desktop, Colima, Rancher, or another Docker-compatible runtime) — or open the repository in a GitHub Codespace, which uses this same container.
2. Open this folder in VS Code and run **Dev Containers: Reopen in Container**.
3. Authenticate to GitHub inside the container if you need API or clone access to private repos:
   - HTTPS: `gh auth login` (recommended default).
   - SSH: forward your SSH agent into the container or mount a key into `~/.ssh/`. The `accept-new` host policy avoids the first-connection prompt.

The container starts without an SSH agent. SSH is only needed when the HTTPS clone fallback in the `canadalogin-release-notes` skill is exercised.

## PDF generation

From inside the container, render a report HTML file to PDF with headless Chromium:

```bash
"$BROWSER" --headless --disable-gpu --no-pdf-header-footer \
  --print-to-pdf=report.pdf \
  "file://$PWD/report.html"
```

Adjust the paths to point at the generated report you want to convert.

## Read-only guarantee

The source repositories and the IBM wiki must remain read-only. Only the generated Markdown, HTML, and PDF report artifacts should be written inside the workspace.
