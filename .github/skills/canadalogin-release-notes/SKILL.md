---
name: canadalogin-release-notes
description: "Generate CanadaLogin release notes for a specified date range (defaulting to the previous seven complete UTC days) using Release Please, Conventional Commits, Git history, .deployed_versions environment files, and the IBM deployment state document. Use when asked for release notes, deployment summaries, or what changed in test, staging, and production."
argument-hint: "Optional: provide a date range such as 2026-08-17 to 2026-08-23"
user-invocable: true
---

# Release Notes

Generate accurate all-environments and Production-only Markdown, HTML, and PDF reports for these repositories:

- `canadalogin-migration`
- `canadalogin-user-selfservice-webapp`
- `gc-signin-ibm-configuration`

Use these friendly names in the report:

| Repository directory | Report name |
|---|---|
| `canadalogin-migration` | Migration App |
| `canadalogin-user-selfservice-webapp` | Manage App |
| `gc-signin-ibm-configuration` | IBM SaaS Configuration, Branding and Flows |

For IBM component names, use friendly names in the report when a mapping exists. Keep the technical `File` value available for internal matching, but do not expose it unless it helps explain the user-facing change:

| Technical component | Report name |
|---|---|
| `default` | Default Theme |
| `gcs_themes_common` | Common Theme |
| `gcs_themes_passwordrecovery` | Password Recovery Theme |
| `gcs_themes_passwordrecovery_selection` | Password Recovery Selection Theme |
| `gcs_themes_phonerecovery1` | Phone Recovery Theme |
| `gcs_themes_phonerecovery2` | Phone Recovery Theme |
| `gcs_themes_signin` | Sign In Theme |
| `gcs_themes_signup1` | Sign Up Theme |
| `gcs_themes_signup2` | Sign Up Theme |
| `gcs_attributes_migration_status` | Migration Status Attribute |
| `gcs_flows_api` | API Flow |
| `gcs_flows_migration_solution_redirect` | Migration Redirect Flow |
| `gcs_flows_passwordrecovery` | Password Recovery Flow |
| `gcs_flows_signup` | Sign Up Flow |

List each IBM Theme component separately because Themes are deployed independently, except paired components such as Phone Recovery 1/2 and Sign Up 1/2, which may share one friendly display name in the report. Keep the underlying technical components distinct for evidence and deployment matching. Include related Branding changes in the matching Theme's summary. Do not copy the same generic summary to every Theme: derive each note from the files belonging to that component. Omit a Theme from the user-facing report when no component-specific change is identified. For a paired display name, omit the group only when every underlying part has no component-specific change; keep the group when at least one part has a documented change and summarize only the changed part or parts. When a new IBM component appears that is not in this table, create a concise title-case name from its normalized identifier and retain the technical identifier in the evidence path. Do not expose raw identifiers as the primary display name when a clear friendly name can be formed.

The report must be easy to paste into Slack and must distinguish code released from code deployed. Lead with what was deployed to each environment for each repository. For every deployment, describe only the user-facing changes introduced since the version or component state that existed immediately before the supplied start date. Use the repository's Release Please changelog, `.deployed_versions` history for the two web applications, and the complete Git history of the IBM deployment wiki page for the IBM configuration repository.

## Inputs and Date Range

1. If the user supplies a start and end date, use that inclusive range.
2. Otherwise use the previous seven complete calendar days in UTC. State the selected range in the report.
3. Use the last committed environment state immediately before the start date as the comparison baseline. The reporting period answers “what was deployed in this range?”; the release notes answer “what changed compared with what was there before this range?”
4. If the supplied or default end date is later than the current UTC calendar date, clamp the effective end date to the current UTC date, use the clamped date for every history lookup, keep the original filename dates for the report artifacts, and add a Notes bullet stating that the end date was clamped.
5. Treat the repositories as sibling directories of the current workspace unless the user provides different paths. If any of the three scoped repositories (`canadalogin-migration`, `canadalogin-user-selfservice-webapp`, `gc-signin-ibm-configuration`) is not already present at the expected sibling path, clone it from `git@github.com:cds-snc/<REPO_NAME>.git` using `fetch-depth: 0` so complete history is available. If the sibling directory cannot be created (for example, when the workspace parent is a read-only mount like `/workspaces/` in a dev container), fall back to `$HOME/repos/<REPO_NAME>` for the clone destination and use that path for all subsequent reads; apply the same rule to the IBM wiki (`$HOME/repos/gc-signin-ibm-configuration.wiki`). If the SSH clone fails (for example, when no SSH key with repository read access is configured, or when the SSH endpoint is unreachable), fall back to the HTTPS remote `https://github.com/cds-snc/<REPO_NAME>.git` with the same full-history requirement, using an available token (such as the workflow token in GitHub Actions or a locally configured `gh`/credential helper) when the repository is private. Fail the report generation only if neither SSH nor HTTPS clone succeeds. Apply the same SSH-then-HTTPS fallback to the IBM wiki clone described under Environment Sources.
6. Use read-only Git and GitHub operations in every accessed repository and wiki. Never push, commit, tag, create or delete branches, create or update releases, open or modify pull requests or issues, dispatch workflows, or modify source files, changelogs, deployment files, or wiki pages. The only files the skill may create or update are the generated Markdown, HTML, and PDF reports described below in the release-notes workspace.
7. When run by GitHub Actions, grant the workflow only the permissions required to read source repositories and publish the Pages artifact. Use read-only checkout credentials for the three source repositories and the wiki; never configure a write-capable remote or use a token with repository write permissions.

## Environment Sources

For `canadalogin-migration` and `canadalogin-user-selfservice-webapp`, use this mapping:

| Report environment | File |
|---|---|
| Test | `.deployed_versions/test.json` |
| Staging | `.deployed_versions/staging.json` |
| Production | `.deployed_versions/prod.json` |

For a scoped repository that uses `.deployed_versions/production.json`, treat it as the Production file. Do not assume that `prod.json` and `production.json` both exist.

Each JSON file normally contains a top-level `version` property. Parse it as JSON. If a file or property is missing, report `Unavailable` and explain why. Never infer an environment version from the repository manifest, `CHANGELOG.md`, or another environment.

For `gc-signin-ibm-configuration`, use `Deployment-State.md` exclusively from the repository's GitHub wiki. The wiki is a separate Git repository at `git@github.com:cds-snc/gc-signin-ibm-configuration.wiki.git`; clone it with `fetch-depth: 0` so every page revision is available. If the SSH clone fails (for example, when no SSH key with wiki read access is configured, or when the SSH endpoint is unreachable), fall back to the HTTPS remote `https://github.com/cds-snc/gc-signin-ibm-configuration.wiki.git` with the same `fetch-depth: 0` requirement, using the workflow token in GitHub Actions or an equivalent locally configured credential when needed. Do not use a workspace or repository copy of `Deployment-State.md`, and fail the report generation if neither the SSH nor HTTPS clone succeeds or the page is unavailable. Do not look for or use `.deployed_versions` for this repository. The document contains independent deployment streams under these headings:

- Themes
- Attributes
- Flows
- Policies

Parse each table row as a separate component. Use the `File` value as the component name and the `staging Tag`/`staging Date` and `prod Tag`/`prod Date` values as Staging and Production evidence. A `-` value means `Unavailable`, not version zero or no change. Do not collapse components within a category: each component can be released and promoted independently.

IBM Test is represented by the `main` branch, not by a deployment-state column. Determine the Test state and in-period changes from committed `main` history at the reporting-period boundaries. Match a state-table component to the changed configuration file by normalized component name within its category. For example, `Flows/gcs_flows_signup.json` maps to the `gcs_flows_signup` row; also inspect category directories and nested filenames when the component name is not a direct filename match. For each matched component, report the `main` commit SHA/date and the merged change or PR that put the file into Test. Use a Release Please version only when an exact tag or changelog entry can be tied to that component change; otherwise identify Test by the `main` commit rather than inventing a version. Never infer IBM Test from Staging or Production.

## Procedure

1. Verify each path is a Git repository and obtain its remote URL and default branch. If the expected sibling directory for any of the three scoped repositories does not exist or is not a Git repository, clone it from `git@github.com:cds-snc/<REPO_NAME>.git` with `fetch-depth: 0`; if the SSH clone fails, fall back to `https://github.com/cds-snc/<REPO_NAME>.git` with the same full-history requirement before continuing.
2. For `canadalogin-migration` and `canadalogin-user-selfservice-webapp`, for each environment file that exists:
   - Read the version at the start of the reporting period with `git show <default-branch>:<file>` or the closest commit at the start date.
   - Read the version at the end of the period from the closest commit at or before the end date.
   - Inspect `git log --follow` for commits touching that file during the period. These commits are deployment evidence.
   - Show Test, Staging, and Production for every repository. For each environment, state the deployed version and date when it changed during the reporting period, or say `No deployment recorded this period` when it did not. Include release notes only for environments that changed.
3. For `gc-signin-ibm-configuration`:
   - Clone the wiki with complete history and inspect every revision of `Deployment-State.md`, including the closest committed snapshot before the reporting period, the first snapshot in the period, and the final snapshot at or before the period end. Use `git log --follow --all -- Deployment-State.md` and `git show <revision>:Deployment-State.md` for the page history; do not use a shallow clone.
   - Inspect all page revisions and `git log --follow` commits touching `Deployment-State.md` during the period. The dated table entries are deployment evidence; the document's auto-generated timestamp is document-generation metadata, not proof that every component changed. Preserve revision date, author, commit, and page diff as internal evidence, but never expose commit IDs, links, or technical audit details in the report.
   - Inspect `main` history at the start and end of the period and list commits during the period that changed configuration files under `Themes/`, `Attributes/`, `Flows/`, or `Policies/`. These commits represent changes in Test because `main` determines Test.
   - Map each changed configuration file to its state-table component by normalized category and component name. Use the Test commit, changed file, and associated PR or Conventional Commit internally to identify the change, but report only the friendly component name, date, and human-readable summary.
   - Show Test, Staging, and Production headings for IBM. Under each heading, report only components whose state changed during the reporting period; say `No deployment recorded this period` when none changed. For each changed component, report the environment, tag or `main` commit, deployment/change date, previous state, and human-friendly notes for the delta from the state before the period.
   - Keep the category and friendly component name attached to every version. A release of one component must not be described as a release of the whole IBM repository.
4. Determine the release content for each version or component transition:
   - Read `CHANGELOG.md`, which is generated by Release Please.
   - Match release headings by exact version, accepting common forms such as `1.2.3` and `v1.2.3`.
   - Use the configured sections: Features, Bug Fixes, Performance Improvements, Code Refactoring, Tests, Miscellaneous Chores, Continuous Integration, Documentation, and Code Style.
   - Compare the deployed version with the previous deployed version, not with the latest repository version. Summarize the complete intervening delta, even when the deployment skipped one or more releases.
   - For every candidate user-facing note, inspect the corresponding commit diff and changed files, not only the changelog subject. Trace the note to a concrete implementation or configuration change before including it.
   - Rewrite raw commit language as a specific outcome or capability. Name the affected flow, page, control, locale, timing behavior, validation, tracking behavior, or dependency where the evidence supports it. Avoid vague phrases such as `updated URL`, `improved handling`, or `related page updates` when the changed file reveals the actual behavior.
   - Combine overlapping bullets when they describe the same underlying change, and remove a generic parent bullet when its detailed bullets already describe the complete behavior. Do not claim a change based solely on a release version, generated changelog edit, or deployment bookkeeping.
   - Use PR titles and commit summaries from the changelog as source material, but do not include PR numbers, commit IDs, commit links, or repository URLs in the user-facing report.
   - If no matching changelog entry exists, use the commits between the previous and new release tag, preferring merged PR titles from `gh pr list` when authenticated. Summarize the result without exposing commit IDs, PR numbers, or links.
   - For IBM Staging or Production deployments, always produce a human-friendly change summary for each changed component. Compare the previous deployed tag with the newly deployed tag using the relevant category/component paths, then summarize the resulting commits or file changes. A line such as `v1.13.0 deployed` is incomplete on its own.
   - If `Deployment-State.md` does not preserve the previous deployed tag, resolve it before writing the report from the full wiki page history, the component's deployment history, the closest earlier state-document snapshot, the tag ancestry, or the nearest base tag. For example, compare `v1.12.0.1` with `v1.12.0` when the deployed tag is a `.1` hotfix and the tag history confirms that base. Do not report the previous version as unavailable while an exact wiki revision or repository tag comparison is possible.
   - For IBM Themes, inspect both the matching `Themes/` configuration and any related `Branding/` files changed between the two tags. State the user-visible result, such as updated sign-in, sign-up, recovery, MFA, page content, styling, or localization behavior. Attribute the note to the specific Theme component when the changed files allow it. Give every independently deployed Theme its own summary; never reuse one release-wide sentence for all Theme rows. For a merged paired display name, say `No component-specific change identified in either paired component in the tag comparison` only when neither underlying part has a change; otherwise describe the changed part or parts and omit that statement.
   - For IBM Attributes, Flows, and Policies, use the same tag-to-tag path-restricted comparison. Include the component's actual configuration or behavior change, not only its version number. If the tags cannot be compared or no component-specific diff is available after all repository-history lookups, say `Change details unavailable from repository history` rather than inventing a summary.
5. Filter out release-only noise such as `chore(main): release ...`, generated `CHANGELOG.md` edits, deployment bookkeeping (`ci: deploy X to Y`, `ci: release X to prod`), lock-file-only bumps, formatter or whitespace-only changes, and commit-message or comment-only edits. Never present these as product changes.
   - After removing that noise, keep two tiers of surviving changes:
     - **User-facing** bullets describe behaviour or content a person using the app or configuration would notice. Prefer these; state the outcome or capability.
     - **Under the hood** bullets describe important non-user-facing work that engineers, operators, or security reviewers care about, even when end users see nothing new. Include changes such as: security-relevant dependency upgrades (CVE fixes, advisory patches, major-version bumps of runtime libraries), meaningful refactors that touch shared modules or reduce risk (for example, extracting a shared component, replacing a deprecated API, reworking auth/session handling), test-coverage additions for critical flows, observability or logging improvements, build/CI hardening that affects reliability or supply-chain safety, and infrastructure or configuration changes with runtime impact.
   - Exclude from both tiers: routine minor/patch dependency bumps with no runtime impact, GitHub Actions version-only bumps, dev-only tooling updates (linters, formatters, editorconfig), Renovate/Dependabot lock-file maintenance without a stated security or behaviour reason, documentation-only edits unless they change user-visible content, and any change the Manage App identity-verification exclusion (procedure step 6) covers.
   - Keep the Under the hood list short — a few high-signal bullets per environment at most. If nothing qualifies, omit the sub-list entirely rather than padding it. Under the hood bullets follow the same anti-audit rules: no commit IDs, PR numbers, or links.
6. For `canadalogin-user-selfservice-webapp`, exclude all identity-verification features, fixes, refactors, tests, documentation, dependency work, and deployment changes from the release notes. Do not mention identity verification in the summary, environment sections, or data-gap notes. If a deployment contains only excluded identity-verification work, report the deployment and say `No included user-facing changes`.
   - Apply this exclusion before drafting or consolidating bullets. Do not retain a generic localization, dependency, test, or accessibility note when its only supporting changes are identity-verification files.
7. Consolidate identical release content across environments. Explain promotions briefly, for example: `1.30.0 deployed to Test; Staging and Production were unchanged.`
8. Do not claim that a version was deployed during the period merely because it is currently in an environment file or state table. A deployment during the period requires dated file history, a dated state-table entry, or an authoritative GitHub deployment/release record.
9. If a repository or IBM state category has no deployment data, include a short note only when it prevents determining what was deployed. Do not clutter the report with unchanged or unavailable environments.
10. If the working tree has uncommitted changes, do not use them as historical evidence; mention that the report is based on committed history.
11. Write the completed reports to the workspace root as `canadalogin-release-notes-YYYY-MM-DD-to-YYYY-MM-DD.md`, `canadalogin-release-notes-YYYY-MM-DD-to-YYYY-MM-DD.html`, and `canadalogin-release-notes-YYYY-MM-DD-to-YYYY-MM-DD.pdf`, using the supplied inclusive dates. Create all three files after all evidence and quality checks are complete. If any file already exists, replace it with the newly generated report for the same date range.
12. Generate a second production-only report from the same evidence. Use `canadalogin-release-notes-production-only-YYYY-MM-DD-to-YYYY-MM-DD.md`, `canadalogin-release-notes-production-only-YYYY-MM-DD-to-YYYY-MM-DD.html`, and `canadalogin-release-notes-production-only-YYYY-MM-DD-to-YYYY-MM-DD.pdf`. The production-only report must include only Production deployment status and Production changes for every scoped repository; omit Test and Staging headings, entries, summaries, and notes entirely. Its title must identify it as `CanadaLogin Production Release Notes` and its summary must describe only Production activity.

## Output Format

Write concise Slack-formatted text to the dated Markdown reports and return links to all six dated report files. The all-environments Markdown report must be directly copyable into Slack without exposing Markdown heading markers. The production-only Markdown report follows the same Slack format but contains only Production status and changes. Do not use `#` or `##` headings, tables, long audit inventories, or separate sections for unchanged environments. Use Slack-native formatting: single asterisks for bold section and repository names, short bullets beginning with `-`, and backticks only for versions when useful. Do not include commit links, PR links, repository URLs, commit IDs, PR numbers, or other audit-only links in either report. Do not add a preamble, commentary, or fenced code block around the Markdown report content.

Each HTML report must contain the same factual content as its corresponding Markdown report in a self-contained, accessible HTML document. Use a restrained Government of Canada Design System look and feel with a muted, low-alarm palette: a dark navy/slate accent (for example, `#26374a`) for section titles and left-border rules, high-contrast typography on a light background, simple bordered panels, and responsive spacing. Do not render a Government of Canada masthead or a separate `CanadaLogin` banner strip; open the document directly with the report title. Reserve red exclusively for the small "AI-generated" attribution or a subtle rule accent — never for full-width banners, section backgrounds, or repeated section titles. Make environment labels visually obvious: render `TEST`, `STAGING`, and `PROD` as uppercase pill or badge headings (letter-spaced, bold, with a filled muted accent background and light text) so the reader can immediately tell which environment a change belongs to; use per-environment shades within the muted palette (for example, dark navy for TEST, medium slate for STAGING, dark green for PROD) rather than warning colours. Use colon-free report titles, semantic sections, a clear title, an `AI-generated report.` attribution, app-level separation, environment status rows, and IBM category groupings. Include a small responsive stylesheet inline so the files can be opened directly without a server or external assets. Do not add audit links or technical identifiers to either HTML report.

Generate each PDF from its corresponding completed HTML report using a standards-compliant browser print-to-PDF process. The dev container ships headless Chromium at `/usr/bin/chromium`; invoke it directly rather than through `$BROWSER`, which may resolve to the VS Code CLI in an attached shell and silently drop the print flags. Use `/usr/bin/chromium --headless=new --no-sandbox --disable-gpu --no-pdf-header-footer --print-to-pdf=<absolute-output-path>.pdf "file://<absolute-input-path>.html"` with absolute paths on both sides. Preserve the HTML report's Government of Canada styling, page margins, readable typography, section separation, and accessibility-oriented contrast. Each PDF must contain the same factual content as its corresponding Markdown and HTML report and must not expose audit links or technical identifiers.

```markdown
*CanadaLogin Release Notes*
YYYY-MM-DD to YYYY-MM-DD

AI-generated report.

*Summary*
One or two upbeat, concise sentences highlighting the period's meaningful deployments, promotions, and user-facing progress.

*Deployments*

*Migration App*

*TEST* — `1.2.3` unchanged

*STAGING* — `1.2.3` unchanged

*PROD* — `1.2.3` deployed YYYY-MM-DD, previously `1.2.2`.
- Human-friendly change summary.
- Human-friendly change summary.
- Under the hood:
  - Notable non-user-facing change, such as a security-relevant dependency upgrade or a meaningful refactor of a shared module.

Make the environment obvious. Always render each environment label as a standalone, uppercase, bold token — `*TEST*`, `*STAGING*`, `*PROD*` — followed by an em dash and the status. Do not use lowercase, sentence-case, or unbolded environment names in either the web-app blocks or the IBM category groupings. Leave each app's environment lines together as one block. Use `No deployment recorded this period` only when the environment version is unavailable; use `` `version` unchanged `` when the committed environment state is known and did not change. Include the `Under the hood:` bullet (with its nested sub-bullets) only when at least one qualifying non-user-facing change exists for that environment.

*Manage App*

*TEST* — `1.2.3` deployed YYYY-MM-DD, previously `1.2.2`.
- Human-friendly change summary, excluding identity-verification work.

*STAGING* — `1.2.3` unchanged
*PROD* — `1.2.3` unchanged

*IBM SaaS Configuration, Branding and Flows*

*TEST*

Flows
- Sign Up Flow — updated in Test on YYYY-MM-DD: human-friendly change summary.

*STAGING*

Themes
- Default Theme - `v1.13.0` deployed YYYY-MM-DD, previously `v1.12.0`: human-friendly summary of the Theme or related Branding changes.
- Common Theme - `v1.13.0` deployed YYYY-MM-DD, previously `v1.12.0`: human-friendly summary of the Theme or related Branding changes.

Flows
- component-name - `v1.13.0` deployed YYYY-MM-DD, previously `v1.12.0`: human-friendly summary of the Flow changes.

*PROD*

Flows
- component-name - `v1.12.0.1` deployed YYYY-MM-DD, previously `v1.12.0`: human-friendly summary of the Flow changes.

Use the same category grouping for IBM Attributes and Policies when those categories have changed. Do not combine categories into one generic component list. Keep unchanged IBM components as a concise status line, and omit categories with no changed or relevant components.

For IBM, group only changed components under the environment where they changed. Keep the category and friendly component name visible. Include the configuration path and commit/PR only as internal evidence; do not put them in the Slack report. Use the state-table tag/date for Staging and Production only when it helps identify what was deployed.

*Notes*
- ...
```

For the production-only report, use the same structure and wording rules as the all-environments report, but title it `CanadaLogin Production Release Notes` without a colon, place the date on the next line, and include only each repository's Production status and changes. Do not include `TEST`, `STAGING`, IBM Test evidence, IBM Staging evidence, or notes that describe non-production activity. A known unchanged Production version remains a concise `` `version` unchanged `` line; use `No deployment recorded this period` only when Production evidence is unavailable.

Omit empty change categories. Keep release notes user-facing and concise: describe the outcome or capability rather than repeating raw commit prefixes. Retain versions and dates where useful, but omit audit links and technical identifiers from the report.
Always include `AI-generated report.` immediately below the report title.

## Quality Checks

Before returning the report, verify:

- All three scoped repositories appear under Deployments.
- Each repository shows Test, Staging, and Production; unchanged environments use a concise status line rather than a full inventory.
- The comparison baseline is the committed state immediately before the reporting period.
- `gc-signin-terraform` is excluded.
- IBM configuration is sourced exclusively from the GitHub wiki's complete `Deployment-State.md` history, with independent Themes, Attributes, Flows, and Policies streams.
- The IBM wiki is cloned with `fetch-depth: 0`; all page revisions needed for the reporting-period baseline and deployment evidence are available before drafting.
- A local `Deployment-State.md` copy is never used as a fallback; missing wiki access is a report-generation error.
- IBM Test is sourced from `main` history and mapped to state-table components through changed configuration files.
- IBM Test changes are matched using the category, component, path, and `main` commit/PR internally, but the report uses the friendly component name and human-readable summary.
- IBM Staging and Production entries include what changed in the component between the previous and deployed tags; a tag-only statement is not sufficient.
- Every included change bullet can be traced internally to a changed file and commit or tag comparison, and its wording describes the evidenced behavior rather than repeating a generic commit subject.
- Overlapping bullets are consolidated, generic parent bullets are removed when detailed bullets cover the same change, and excluded-only changes do not survive as generic notes.
- Any Under the hood bullets included describe security-relevant dependency work, meaningful refactors, notable test/observability additions, or infrastructure/CI changes with runtime impact; routine dependency bumps, lock-file maintenance, dev-tooling updates, and release chores are not surfaced.
- The report contains no commit links, PR links, repository URLs, commit IDs, or PR numbers.
- If an IBM deployment tag is a hotfix or patch tag, the report identifies and compares its confirmed base tag before summarizing the component change.
- Self-service identity-verification content is excluded everywhere in the report.
- Every reported version came from the corresponding environment file or an explicitly identified release tag.
- A deployment claim has dated Git evidence, a dated IBM state-table entry, or an authoritative GitHub record.
- Release-only commits are not presented as product changes.
- The date range, timezone, missing files, and assumptions are stated.
- The final report is saved as `canadalogin-release-notes-YYYY-MM-DD-to-YYYY-MM-DD.md` at the workspace root and is directly copyable into Slack.
- The matching `canadalogin-release-notes-YYYY-MM-DD-to-YYYY-MM-DD.html` report is saved at the workspace root and contains the same factual content in a self-contained accessible HTML layout.
- The matching `canadalogin-release-notes-YYYY-MM-DD-to-YYYY-MM-DD.pdf` report is saved at the workspace root, generated from the HTML report, and preserves its Government of Canada styling.
- The production-only Markdown, HTML, and PDF reports use the `canadalogin-release-notes-production-only-YYYY-MM-DD-to-YYYY-MM-DD` filename prefix and contain no Test or Staging content.
