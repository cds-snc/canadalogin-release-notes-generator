# CanadaLogin Release Notes Generator

A Copilot skill and dev container that generate CanadaLogin release notes for any date range as regular Markdown and an accessible GCDS-based HTML report.

Covers the three CanadaLogin source repos: `canadalogin-migration` (Migration App), `canadalogin-user-selfservice-webapp` (Manage App), and `gc-signin-ibm-configuration` (IBM SaaS Configuration, Branding and Flows). Each run produces both an all-environments report and a production-only report at the workspace root.

## Usage

**Prerequisites** — VS Code with the Dev Containers extension and a local Docker-compatible runtime (Docker Desktop, Colima, Rancher, etc.).

1. Open this repo in VS Code and **Reopen in Container**.
2. Ask the Copilot agent to run the `canadalogin-release-notes` skill, optionally with a date range (for example, `2026-08-17 to 2026-08-21`) and an optional cadence keyword (`weekly`, `bi-weekly`, or `monthly`) that splits the range into consecutive sub-reports — for example, `2026-07-01 to 2026-09-01 bi-weekly`. Without a range it uses the previous fourteen complete UTC calendar days; without a cadence it produces one report for the whole range.

The skill is read-only against every source repo and wiki, and drops six files at the repo root:

- `canadalogin-release-notes-YYYY-MM-DD-to-YYYY-MM-DD.{md,html}`
- `canadalogin-release-notes-production-only-YYYY-MM-DD-to-YYYY-MM-DD.{md,html}`
- `index.html` — a GitHub Pages–ready archive listing every report present at the repo root.
- `.nojekyll` — disables Jekyll processing on GitHub Pages.

## Publishing to GitHub Pages or static relase notes website

This repo is not published to Pages or a static website. Publish the reports from a separate repository so the generator stays a clean, read-only tool.

1. Generate reports here by invoking the skill. Re-run for any additional date range you want to include. Each run rewrites `index.html` to cover every report currently at the repo root.
2. Copy the following files into the site repo you have configured for GitHub Pages, preserving filenames:
   - every `canadalogin-release-notes-*.html` and `canadalogin-release-notes-*.md`
   - `index.html`
   - `.nojekyll`
3. Commit and push in the site repo. GitHub Pages serves the reports directly; no build step is required because each HTML file loads its pinned GCDS assets from the official CDN.

## Where things live

- [.github/skills/canadalogin-release-notes/SKILL.md](.github/skills/canadalogin-release-notes/SKILL.md) — full skill definition (inputs, sources, procedure, output format, quality checks).
- [examples/report-example.html](examples/report-example.html) — GCDS-first visual reference for report layout and the small CSS customization layer used around the components.
- [.devcontainer/](.devcontainer/README.md) — dev container with `gh`, `jq`, `openssh-client`, and Python 3.

Default community health files are maintained at https://github.com/cds-snc/.github.
