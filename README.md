# CanadaLogin Release Notes Generator

Tooling for generating CanadaLogin release notes from the source repositories' Git history for any date range — no manual copy/paste required. Produces Slack-ready Markdown, an accessible HTML report styled with a muted Government of Canada palette, and a PDF rendered from that HTML.

## What it covers

Reports summarize what was deployed to Test, Staging, and Production for the CanadaLogin stack:

- `canadalogin-migration` — reported as **Migration App**
- `canadalogin-user-selfservice-webapp` — reported as **Manage App**
- `gc-signin-ibm-configuration` — reported as **IBM SaaS Configuration, Branding and Flows**

For each reporting period the generator produces two report families:

- **All environments** — `canadalogin-release-notes-YYYY-MM-DD-to-YYYY-MM-DD.{md,html,pdf}`
- **Production only** — `canadalogin-release-notes-production-only-YYYY-MM-DD-to-YYYY-MM-DD.{md,html,pdf}`

## Data sources

- **Web apps** — deployed versions are read from `.deployed_versions/{test,staging,prod}.json` in each app repo. Release notes for each environment transition come from `CHANGELOG.md` (Release Please) and, when needed, the underlying commits.
- **IBM configuration** — deployment state is read exclusively from the [`Deployment-State.md`](https://github.com/cds-snc/gc-signin-ibm-configuration/wiki/Deployment-State) page of the `gc-signin-ibm-configuration` wiki. Test state is determined from `main` history. Human-friendly change summaries are derived from tag-to-tag diffs restricted to each component's `Themes/`, `Attributes/`, `Flows/`, `Policies/`, and related `Branding/` paths.

The generator only performs read-only Git and GitHub operations. It never writes to any source repository or wiki.

## How to run it

The skill is described in [.github/skills/release-notes/SKILL.md](.github/skills/release-notes/SKILL.md) and is intended to be invoked by a GitHub Copilot agent inside VS Code (or by a GitHub Actions workflow — see below).

1. Open this repository in VS Code and **Reopen in Container** (uses the dev container in [.devcontainer/](.devcontainer/README.md)).
2. Ensure GitHub access is available inside the container. The skill clones each source repo with SSH first (`git@github.com:cds-snc/<repo>.git`) and falls back to HTTPS. In GitHub Actions the workflow token is used for HTTPS.
3. Ask the Copilot agent to run the `release-notes` skill, optionally passing a date range such as `2026-08-17 to 2026-08-21`. Without a range, the previous seven complete UTC calendar days are used.
4. The generator writes six files at the repo root — the two Markdown/HTML/PDF families listed above.

The dev container ships headless Chromium, `gh`, `jq`, `openssh-client`, and Python 3 so it can clone the source repos, parse deployed-versions JSON, and render the HTML report to PDF without any additional setup.

## Report style

- **Slack Markdown** — copy-paste-ready. Environment labels are rendered as bold uppercase tokens (`*TEST*`, `*STAGING*`, `*PROD*`) so the reader can immediately tell which environment a change applies to.
- **HTML** — self-contained, accessible document with a muted, low-alarm Government of Canada palette. Environment labels are shown as coloured pill badges (navy for TEST, slate for STAGING, dark green for PROD). No mastheads or banner strips.
- **PDF** — rendered from the HTML by headless Chromium so print styling matches on-screen styling.
- Every report distinguishes **user-facing** bullets from an optional **Under the hood** sub-list for important non-user-facing work (security-relevant dependency upgrades, meaningful refactors, notable observability or CI hardening).
- Identity-verification content from `canadalogin-user-selfservice-webapp` is excluded everywhere in the report.
- Audit links (commit IDs, PR numbers, repository URLs) are never included in the report.

## Repository layout

- [.github/skills/release-notes/SKILL.md](.github/skills/release-notes/SKILL.md) — the full skill definition (inputs, date range rules, environment sources, procedure, output format, and quality checks).
- [.devcontainer/](.devcontainer/README.md) — dev container that provides Chromium, `gh`, `jq`, Python 3, and an SSH client.
- `.gitignore` — excludes generated `canadalogin-release-notes-*.{md,html,pdf}` artifacts.

Note that default community health files are maintained at https://github.com/cds-snc/.github
