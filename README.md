# CanadaLogin Release Notes Generator

A Copilot skill and dev container that generate CanadaLogin release notes for any date range as regular Markdown and accessible GCDS-based HTML reports.

Covers the three CanadaLogin source repos: `canadalogin-migration` (Migration App), `canadalogin-user-selfservice-webapp` (Manage App), and `gc-signin-ibm-configuration` (IBM SaaS Configuration, Branding and Flows). Each run produces separate Test, Staging, and Production reports at the workspace root. Test and Staging reports retain additional evidence-backed detail useful to lower-environment audiences; the Production report remains concise.

## Usage

**Prerequisites** — VS Code with the Dev Containers extension and a local Docker-compatible runtime (Docker Desktop, Colima, Rancher, etc.).

1. Open this repo in VS Code and **Reopen in Container**.
2. Ask the Copilot agent to run the `canadalogin-release-notes` skill, optionally with a date range (for example, `2026-08-17 to 2026-08-21`), cadence keyword (`weekly`, `bi-weekly`, or `monthly`), and environment (`test`, `staging`, or `prod`). For example, `2026-08-17 to 2026-08-21 staging` generates only the Staging reports, while `2026-08-17 to 2026-08-21` generates reports for all three environments. The environment can also be supplied without a date range, such as `prod`, which uses the default range. Without a cadence it produces one report for the whole range.

The skill is read-only against every source repo and wiki. It drops the selected environment's Markdown and HTML reports, or all six report files when no environment is specified, plus `index.html` at the repo root:

- `canadalogin-release-notes-test-YYYY-MM-DD-to-YYYY-MM-DD.{md,html}`
- `canadalogin-release-notes-staging-YYYY-MM-DD-to-YYYY-MM-DD.{md,html}`
- `canadalogin-release-notes-production-YYYY-MM-DD-to-YYYY-MM-DD.{md,html}`
- `index.html` — a portable archive listing every report present at the repo root.

## Publishing to a static website

This repo is not itself a website. Publish the reports from a separate repository or static hosting service so the generator stays a clean, read-only tool. GitHub Pages is one example; any host that serves static HTML and Markdown files will work.

1. Generate reports here by invoking the skill. Re-run for any additional date range you want to include. Each run rewrites `index.html` to cover every report currently at the repo root.
2. Copy the following files into the site repository or hosting directory, preserving filenames:
   - every `canadalogin-release-notes-*.html` and `canadalogin-release-notes-*.md`
   - `index.html`
3. Deploy the files with your hosting provider. No build step is required because each HTML file loads its pinned GCDS assets from the official CDN.

## Where things live

- [.github/skills/canadalogin-release-notes/SKILL.md](.github/skills/canadalogin-release-notes/SKILL.md) — full skill definition (inputs, sources, procedure, output format, quality checks).
- [examples/report-example.html](examples/report-example.html) — GCDS-first visual reference for report layout and the small CSS customization layer used around the components.
- [.devcontainer/](.devcontainer/README.md) — dev container with `gh`, `jq`, `openssh-client`, and Python 3.

Default community health files are maintained at https://github.com/cds-snc/.github.
