# CanadaLogin Release Notes Generator

A Copilot skill and dev container that generate CanadaLogin release notes for any date range as Slack-ready Markdown, an accessible HTML report, and a PDF.

Covers the three CanadaLogin source repos: `canadalogin-migration` (Migration App), `canadalogin-user-selfservice-webapp` (Manage App), and `gc-signin-ibm-configuration` (IBM SaaS Configuration, Branding and Flows). Each run produces both an all-environments report and a production-only report at the workspace root.

## Usage

**Prerequisites** — VS Code with the Dev Containers extension and a local Docker-compatible runtime (Docker Desktop, Colima, Rancher, etc.).

1. Open this repo in VS Code and **Reopen in Container**.
2. Ask the Copilot agent to run the `canadalogin-release-notes` skill, optionally with a date range (for example, `2026-08-17 to 2026-08-21`). Without a range it uses the previous seven complete UTC calendar days.

The skill is read-only against every source repo and wiki, and drops six files at the repo root:

- `canadalogin-release-notes-YYYY-MM-DD-to-YYYY-MM-DD.{md,html,pdf}`
- `canadalogin-release-notes-production-only-YYYY-MM-DD-to-YYYY-MM-DD.{md,html,pdf}`

## Where things live

- [.github/skills/canadalogin-release-notes/SKILL.md](.github/skills/canadalogin-release-notes/SKILL.md) — full skill definition (inputs, sources, procedure, output format, quality checks).
- [.devcontainer/](.devcontainer/README.md) — dev container with headless Chromium (for PDF), `gh`, `jq`, `openssh-client`, and Python 3.

Default community health files are maintained at https://github.com/cds-snc/.github.

## Planned

Wrap this skill in a GitHub Actions workflow that runs on a schedule (and on manual dispatch with a custom date range), commits or uploads the six generated files, and publishes an archive of past reports to GitHub Pages (or an equivalent static site) so the release notes are browsable in a single place instead of only inside the workspace.
