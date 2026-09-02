# CanadaLogin Release Notes Generator

A Copilot skill and dev container that maintain separate cumulative CanadaLogin What's New pages for Test, Staging, and Production as regular Markdown and accessible GCDS-based HTML.

Covers the three CanadaLogin source repos: `canadalogin-migration` (Migration App), `canadalogin-user-selfservice-webapp` (Manage App), and `gc-signin-ibm-configuration` (Sign-in, sign-up, recovery flows). Each environment has its own cumulative What's New page, and each page is updated from its own last-update marker.

## Usage

**Prerequisites** — VS Code with the Dev Containers extension and a local Docker-compatible runtime (Docker Desktop, Colima, Rancher, etc.).

1. Open this repo in VS Code and **Reopen in Container**.
2. Ask the Copilot agent to run the `canadalogin-release-notes` skill. Requests can use natural language or named arguments in any order:
   - `Update the What's New pages.`
   - `Update staging since August 1, 2026 through August 31, 2026.`
   - `environment=prod since=2026-08-01 to=2026-08-31`

   The supported environment values are `test`, `staging`, and `prod`; omitting the environment updates all three pages. Without a starting date, each page continues from its own marker. A new page uses the previous fourteen complete UTC calendar days. Ambiguous dates and unsupported environment values require clarification rather than being guessed.

The skill is read-only against every source repo and wiki. It updates these files at the workspace root:

- `canadalogin-releases-whats-new-test.{md,html}`
- `canadalogin-releases-whats-new-staging.{md,html}`
- `canadalogin-releases-whats-new-production.{md,html}`

## Publishing to a static website

This repo is not itself a website. Publish the reports from a separate repository or static hosting service so the generator stays a clean, read-only tool. GitHub Pages is one example; any host that serves static HTML and Markdown files will work.

1. Update the desired environment page here by invoking the skill. Each run prepends a dated update section and preserves earlier entries.
2. Copy the relevant `canadalogin-releases-whats-new-*.html` and, if desired, matching Markdown file into the site repository or hosting directory.
3. Deploy the files with your hosting provider. No build step is required because each HTML file loads its pinned GCDS assets from the official CDN.

## Where things live

- [.github/skills/canadalogin-release-notes/SKILL.md](.github/skills/canadalogin-release-notes/SKILL.md) — full skill definition (update boundaries, sources, writing style, output format, and quality checks).
- [examples/whats-new-example.md](examples/whats-new-example.md) — Markdown page template.
- [examples/whats-new-example.html](examples/whats-new-example.html) — accessible GCDS-based page template.
- [.devcontainer/](.devcontainer/README.md) — dev container with `gh`, `jq`, `openssh-client`, and Python 3.

Default community health files are maintained at https://github.com/cds-snc/.github.
