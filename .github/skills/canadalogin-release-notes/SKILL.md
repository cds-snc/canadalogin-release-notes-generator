---
name: canadalogin-release-notes
description: "Generate CanadaLogin release notes for a specified date range (defaulting to the previous fourteen complete UTC days) with an optional weekly, bi-weekly, or monthly cadence that splits the range into separate reports. Uses Release Please, Conventional Commits, Git history, .deployed_versions environment files, and the IBM deployment state document. Use when asked for release notes, deployment summaries, or what changed in test, staging, and production."
argument-hint: "Optional: provide a date range (e.g., 2026-08-17 to 2026-08-23), cadence (weekly, bi-weekly, or monthly), and environment (test, staging, or prod)"
user-invocable: true
---

# Release Notes

Generate accurate Test, Staging, and Production Markdown and GCDS-based HTML reports for these three repositories:

| Repository directory | Report name |
|---|---|
| `canadalogin-migration` | Migration App |
| `canadalogin-user-selfservice-webapp` | Manage App |
| `gc-signin-ibm-configuration` | IBM SaaS Configuration, Branding and Flows |

Each report must distinguish code released from code deployed. Lead with what was deployed to the report's environment. For every deployment, describe the user-facing changes introduced since the version or component state that existed immediately before the start date. Use the repository's Release Please changelog, `.deployed_versions` history for the two web applications, and the complete Git history of the IBM deployment wiki page for the IBM configuration repository.

## Inputs and Date Range

1. If the user supplies a start and end date, use that inclusive range. Otherwise use the previous fourteen complete calendar days in UTC. State the selected range in the report.
2. If the user supplies an optional cadence keyword (`weekly`, `bi-weekly`, or `monthly`, case-insensitive) after the range — for example, `2026-07-01 to 2026-09-01 bi-weekly` — split the requested range into consecutive inclusive sub-ranges of that length and generate one full set of reports per sub-range. Without a cadence keyword, treat the whole range as a single sub-range and generate one set of reports. Sub-ranges are date-shifted from the requested start date: `weekly` = 7 days per sub-range, `bi-weekly` = 14 days per sub-range, `monthly` = one calendar month per sub-range (start-day-shifted; for example, 2026-07-15 → 2026-08-14). The final sub-range may be shorter when the requested range does not divide evenly; clamp its end to the requested (or clamped) end date. Skip a sub-range whose start date is strictly after the effective end date.
3. If the user supplies an optional environment argument (`test`, `staging`, or `prod`, case-insensitive), generate reports only for that environment. Normalize `prod` to the Production environment and the `production` filename stem. If no environment argument is supplied, generate reports for all three environments. Accept the environment argument with or without a date range or cadence; do not infer an environment from other wording. If an environment value is supplied outside this set, ask the user to choose `test`, `staging`, or `prod` rather than guessing.
4. Use the last committed state immediately before each sub-range's start date as that sub-range's comparison baseline. Every sub-range independently answers "what was deployed in this sub-range?" and "what changed compared with what was there before it?". Do not reuse baselines across sub-ranges.
5. If the requested (or default) end date is later than the current UTC calendar date, clamp the effective end date to the current UTC date before splitting into sub-ranges. Use the clamped date for every history lookup, keep the original filename dates for the sub-range that contains the clamped end, and add a Notes bullet in that sub-range's reports stating the end date was clamped.
6. Treat the three repositories and the IBM wiki as sibling directories of the current workspace. If a sibling is missing or is not a Git repository, clone it with `fetch-depth: 0` (complete history). Try `git@github.com:cds-snc/<REPO_NAME>.git` first; fall back to `https://github.com/cds-snc/<REPO_NAME>.git`, using a locally configured `gh` or credential helper when the repository is private. If the workspace parent is read-only (for example `/workspaces/` in a dev container), clone to `$HOME/repos/<REPO_NAME>` — and `$HOME/repos/gc-signin-ibm-configuration.wiki` for the wiki — and use that path for all reads. Fail only if neither SSH nor HTTPS succeeds.
7. Use read-only Git and GitHub operations. Never push, commit, tag, create or delete branches, create or update releases, open or modify PRs or issues, dispatch workflows, or modify source files, changelogs, deployment files, or wiki pages. The only files this skill may create or update are the generated Markdown and HTML reports and `index.html` at the workspace root.

## Environment Sources

### Web applications (Migration App, Manage App)

For `canadalogin-migration` and `canadalogin-user-selfservice-webapp`, use this mapping:

| Report environment | File |
|---|---|
| Test | `.deployed_versions/test.json` |
| Staging | `.deployed_versions/staging.json` |
| Production | `.deployed_versions/prod.json` |

For a repository that uses `.deployed_versions/production.json`, treat it as the Production file. Do not assume both `prod.json` and `production.json` exist. Each file normally contains a top-level `version` property; parse it as JSON. If a file or property is missing, report `Unavailable` and explain why. Never infer an environment version from the repository manifest, `CHANGELOG.md`, or another environment.

Resolve all environment-file history and snapshots against the repository's default branch only. Do not use `--all`, release branches, PR refs, or Release Please branches. A version-file edit on a non-default branch is release preparation, not a deployment, unless a separate authoritative deployment record confirms promotion. Inspect default-branch history with `git log <default-branch> --follow` for commits touching each environment file during the period; those commits are deployment evidence only when they represent a state change reachable from the default branch.

### IBM configuration

Use the deployment-state page exclusively from the repository's wiki, cloned with `fetch-depth: 0`. Do not use a workspace or repository copy of the page; missing wiki access is a report-generation error. Do not use `.deployed_versions` for this repository. The document contains four independent deployment streams — **Themes**, **Attributes**, **Flows**, **Policies**. Parse each table row as a separate component. Use the `File` value as the component name and the `staging Tag`/`staging Date` and `prod Tag`/`prod Date` values as Staging and Production evidence. A `-` value means `Unavailable`, not zero or unchanged. Each component can be released and promoted independently — do not collapse components within a category.

**IBM Test** is represented by the `main` branch, not by a deployment-state column. Determine Test state and in-period changes from committed `main` history at the reporting-period boundaries. Map changed configuration files under `Flows/`, `Branding/`, `Attributes/`, and `AccessPolicies/` to their categories (`Flows`, `Themes`, `Attributes`, and `Policies` respectively). Prefer a state-table component when one exists, matching by normalized name within the category and inspecting nested filenames when needed. When the wiki has no row for a changed Test category, derive the clearest friendly component name from the category and path — this fallback applies to Test only and does not create Staging or Production deployment evidence. Use a Release Please version only when an exact tag or changelog entry can be tied to that component change; otherwise identify Test by the `main` change rather than inventing a version. Never infer IBM Test from Staging or Production.

## IBM Component Naming

Use friendly names for IBM components. Keep the technical source identifier available for internal matching but do not expose it in the report unless it clarifies a user-facing change.

- The technical `default` component maps to the display name `Default`.
- For every other component, infer the display name from the technical identifier and category:
  - Strip machine prefixes and category markers.
  - Convert the remainder into natural title case.
  - Split recognizable compound words and numeric suffixes.
  - Preserve well-known acronyms (for example, `MFA`, `OTP`).
  - Append the category (`Attribute`, `Flow`, `Policy`) only when it helps clarify the name.
- Never expose machine prefixes, raw category markers, normalized identifiers, or numeric part suffixes as the primary name.
- For Themes, omit the word `Theme` from display names. Use `Sign In`, `Password Recovery`, `Phone Recovery`, `Sign Up`.
- Group multi-part Theme families under one friendly display name (for example, all `Phone Recovery` parts appear once). Keep the underlying technical parts distinct for evidence and deployment matching. Include related Branding changes in the family's summary. Inspect each underlying part and consolidate into one summary that identifies which part or parts changed. Omit a Theme family only when none of its underlying parts has a component-specific change. For a merged paired display name, say `No component-specific change identified in either paired component in the tag comparison` only when neither underlying part has a change; otherwise describe the changed part or parts.
- For Access Policies, combine any changed policy components into a single user-facing `Access policies` entry that summarizes the shared outcome. Do not list `Default Policy`, `Default Migration Policy`, `Recover MFA Policy`, or other detailed policy names in the report.
- If an identifier cannot be interpreted confidently, use the clearest human-readable name supported by the category and repository context rather than exposing the raw value.

## Scope Exclusions

**Manage App identity verification.** For `canadalogin-user-selfservice-webapp`, exclude all identity-verification features, fixes, refactors, tests, documentation, dependency work, and deployment changes. Do not mention identity verification in summaries, environment sections, or data-gap notes. If a deployment contains only excluded identity-verification work, report the deployment and say `No included user-facing changes`. Apply this exclusion before drafting or consolidating bullets — do not retain a generic localization, dependency, test, or accessibility note when its only supporting changes are identity-verification files.

## Procedure

Run steps 1–9 once per selected environment and sub-range defined in Inputs and Date Range, using that sub-range's inclusive dates as the reporting period. When no environment was selected, run them for Test, Staging, and Production. Step 10 runs once, after every sub-range has produced the reports for the selected environment(s).

1. **Determine web-app environment state.** For each web app, read the version for the selected environment at the start of the period and at the end from the default branch (`git show <default-branch>:<file>` or the closest commit at or before those dates), and list default-branch commits that touched that environment file during the period. Those commits are deployment evidence when they represent a state change.
2. **Determine IBM state.** Inspect every revision of the IBM wiki deployment-state page during the period, including the closest snapshot before the period, the first snapshot in the period, and the final snapshot at or before the period end. The dated table entries are deployment evidence; the document's auto-generated timestamp is document metadata, not proof of change. Preserve revision date, author, commit, and page diff as internal evidence only. Independently, inspect `main` history at the period boundaries and list commits during the period that changed files under IBM component categories — those represent Test changes. Map each changed Test file to a state-table component by category and normalized name when a row exists; otherwise derive the friendly component from category and path (Test only). Keep the friendly category and component name attached to every version — a release of one component must not be described as a release of the whole IBM repository.
3. **Compute deltas per component.** For every deployment that changed during the period, compare the pre-period state with the newly deployed state and summarize the intervening delta, even when the deployment skipped intermediate releases. For each candidate note, inspect the corresponding commit diff and changed files — not only the changelog subject — and trace it to a concrete implementation or configuration change before including it. For web apps, use `CHANGELOG.md` sections (Features, Bug Fixes, Performance Improvements, Code Refactoring, Tests, Miscellaneous Chores, Continuous Integration, Documentation, Code Style); accept `1.2.3` and `v1.2.3` heading forms. If no matching changelog entry exists, use the commits between the previous and new release tag, preferring merged PR titles from `gh pr list` when authenticated. For IBM Themes, inspect both the matching theme configuration and any related branding files changed between the two tags, and attribute the note to the specific Theme component when the files allow it. For IBM Attributes, Flows, and Policies, use path-restricted tag-to-tag comparison. If a component-specific diff is not available after all repository-history lookups, omit the component change entry rather than inventing a summary. If a deployment tag is a hotfix or patch tag, identify and compare its confirmed base tag first. If the deployment-state page does not preserve the pre-period component state, resolve it from the full wiki page history, the component's deployment history, the closest earlier state-document snapshot, the tag ancestry, or the nearest base tag — keep this comparison internal.
4. **Drop reverted work.** Check the full history through the end of the period for reversions. Exclude any change whose net effect was undone in the period, even if the original commit, release, or IBM tag appears in the interval. Compare the effective end-of-period state with the pre-period baseline; only mention an introduced-and-reverted change when a subsequent non-reverting change restored a distinct final behavior that remains at period end.
5. **Classify surviving changes into two tiers.** Filter out release-only noise (`chore(main): release …`, generated `CHANGELOG.md` edits, deployment bookkeeping such as `ci: deploy X to Y`, lock-file-only bumps, formatter or whitespace changes, comment-only edits) — never present these as product changes. From what remains, keep:
   - **User-facing** bullets: behaviour or content a person using the app or configuration would notice. Prefer these.
   - **Under the hood** bullets: important non-user-facing work engineers, operators, or security reviewers care about — security-relevant dependency upgrades (CVE fixes, advisory patches, major runtime bumps), meaningful refactors of shared modules or auth/session handling, test-coverage additions for critical flows, observability or logging improvements, build/CI hardening with reliability or supply-chain impact, infrastructure or configuration changes with runtime impact.

   Exclude from both tiers: routine minor/patch dependency bumps with no runtime impact, GitHub Actions version-only bumps, dev-tooling updates (linters, formatters, editorconfig), Renovate/Dependabot lock-file maintenance without a stated security or behaviour reason, documentation-only edits unless they change user-visible content, and any content covered by Scope Exclusions.

    Keep the Under the hood list short and high-signal for Production. Test and Staging reports may retain a fuller set of concrete implementation, configuration, testing, observability, and operational details when they help their lower-environment audience understand what is being validated. Omit the list entirely when nothing qualifies.
6. **Consolidate within each environment report.** Each report covers one environment only, so do not summarize promotions or unchanged states from the other environments. Combine overlapping bullets that describe the same underlying change, and remove a generic parent bullet when its detailed children already describe the complete behavior. In Test and Staging reports, keep distinct, evidence-backed details when they support validation or troubleshooting instead of collapsing them into a release-level summary.
7. **Require dated deployment evidence.** A deployment during the period requires dated Git file history, a dated state-table entry, or an authoritative GitHub deployment/release record. A version currently present in an environment file or state table is not by itself evidence of an in-period deployment.
8. **Working tree.** If any repository has uncommitted changes, do not use them as historical evidence. Add a Notes bullet stating the report is based on committed history.
9. **Write the artifacts.** For each selected environment and sub-range, after all evidence and quality checks are complete, write that environment's Markdown and HTML files to the workspace root using the sub-range's inclusive dates in the filename, replacing any existing files with the same names. With no environment selector, write all six files per sub-range:
    - `canadalogin-release-notes-test-YYYY-MM-DD-to-YYYY-MM-DD.md`
    - `canadalogin-release-notes-test-YYYY-MM-DD-to-YYYY-MM-DD.html`
    - `canadalogin-release-notes-staging-YYYY-MM-DD-to-YYYY-MM-DD.md`
    - `canadalogin-release-notes-staging-YYYY-MM-DD-to-YYYY-MM-DD.html`
    - `canadalogin-release-notes-production-YYYY-MM-DD-to-YYYY-MM-DD.md`
    - `canadalogin-release-notes-production-YYYY-MM-DD-to-YYYY-MM-DD.html`

    The Test report contains only Test status and Test changes for every repository. Its title is `CanadaLogin Test Release Notes` (no colon), its summary describes only Test activity, and its Notes describe only Test evidence and scope. The Staging report follows the same rules with `CanadaLogin Staging Release Notes`. The Production report uses the existing `CanadaLogin Production Release Notes` title and contains only Production status, Production changes, and Notes about Production activity. No report includes headings, entries, summaries, or notes for another environment. When an environment selector is supplied, do not create reports for the other environments.
10. **Regenerate the archive.** After every sub-range has been written, scan the workspace root for every `canadalogin-release-notes-*.html` file, group by date range (up to one Test, one Staging, and one Production report per range), and sort newest first by end date, then start date. Rewrite `index.html` from `examples/index-example.html`, preserving its document structure, GCDS component choices, header metadata, semantic sections, archive-entry layout, and inline style block. Replace the sample archive entries with one entry per discovered date range: render the range as a heading and list the available formats as relative links (HTML and Markdown for Test, Staging, and Production), omitting formats that are not present on disk. Replace `index.html` on every run. Do not delete existing report files, and do not publish these artifacts from this repository — they are copied manually to a static hosting site.

## Writing Style

Every user-facing bullet must describe a concrete change and, when the diff supports it, its practical benefit. Apply the **benefit lens** to every bullet before finalizing it:

1. What can users now do — or no longer have to do — that they couldn't before?
2. Which page, flow, control, route, locale, timing, validation, tracking, or configuration value changed?
3. Whose day gets better: end user, admin, operator, or security reviewer?

Only claim benefits the implementation evidence supports. Do not infer accessibility, performance, or user impact from a commit title or presumed intent.

**Preferred sentence pattern.** `<Concrete change on <named page / flow / control>>, so <user or operational benefit>.` Vary the phrasing naturally — the point is that every bullet identifies both *what* changed and *why it matters*.

**Before / after example.**

- Before: `Updated URL for change-sign-in-method page.`
- After: `Manage sign-in guidance now links to the current Canada.ca method-update page, so users land on the working page instead of a broken link.`

**Forbidden generic wording.** Do not use `was updated`, `was changed`, `updated URL`, `improved handling`, or `related page updates` when the diff reveals the actual behavior. For redirect and recovery changes, name the journey, the condition, and the destination. For UI or content changes, name the page or control and the resulting visible behavior. A version-only line such as `v1.13.0 deployed` is never sufficient on its own.

**Summary tone.** Two or three plain sentences that name the changes an end user or operator would care most about. Avoid vague or promotional language.

**No audit identifiers.** Do not include commit IDs, commit links, PR numbers, PR links, repository URLs, wiki revision IDs, or any other technical audit identifiers anywhere in the Markdown or HTML reports. Use PR titles and commit summaries only as source material. This rule applies to user-facing bullets, Under the hood bullets, methodology notes, and summaries.

**Versions.** Show only the latest version deployed to each environment: use `` `version` deployed YYYY-MM-DD `` for a change and `` `version` unchanged `` when the environment did not change. Use `No deployment recorded this period` only when the environment version is unavailable. Retain deployment dates where useful; omit previous versions from the report. Prefix every semantic version shown to users with `v` (for example, `v1.12.14`) — normalize the prefix only for internal matching, never emit a bare version.

## Output Format

Return links to every dated file generated for the selected environment(s). With no environment selector, return links to all six dated files. The Markdown reports use conventional `#` and `##` headings, ordinary Markdown lists, tables only when they improve comparison, and inline code for versions. Do not add a preamble or commentary around the Markdown content.

Each HTML report contains the same factual content as its Markdown counterpart in an accessible GCDS document. Use `examples/report-example.html` as the template — start from its document structure, pinned CDN links, component choices, class names, and style block. Change the report-specific environment title, date, attribution, summary, status content, change content, and methodology notes; do not redesign the page.

HTML structure to preserve. The source example may show the complete visual pattern, but every generated file renders only its named environment. The complete status-grid instruction below applies to the source styling only:

- Plain report header with a GCDS `h1`, the date on its own line, and `AI-GENERATED REPORT.` on its own line. No hero banner, masthead, decorative banner, or product strip.
- Summary in `gcds-notice`. Use `gcds-heading` and `gcds-text` for body content. Application sections use the single-column `gcds-grid`. Preserve the blue top accent on each `.app-block` and the IBM `.ibm-block`.
- Each generated report contains only one environment. The source template's status-grid styling is retained for visual reference; generated output renders only the status cell for its named environment and never includes another environment's status, changes, or summary.
- All three web-app status cells together in the existing status grid. Apply `.is-deployed` when the status says `deployed`, `.is-unchanged` when it says `unchanged`; use neutral styling for unavailable states. The accent describes the status text, not the environment name — Production is not automatically green.
- After each app status cell, put included change bullets in the labelled group for the report's environment (`TEST changes`, `STAGING changes`, or `PROD changes`). Never include another environment's status, changes, or summary. Place qualifying Under the hood content inside the same environment change group, after that environment's user-facing bullets; never render it as an app-level subsection detached from the environment.
- IBM section as the same-sized GCDS `h3` as the app titles, with its blue top accent. Include only the report's `.ibm-environment` block and compact status cell. Apply `.is-deployed` when that environment has one or more included component changes and `.is-unchanged` when it has none. Preserve the category headings and list changed components under that environment and category. Test and Staging may include the extra concrete component and operational detail permitted above.
- Notes section and transparent bordered `gcds-details` methodology block. Preserve semantic headings, `aria-labelledby` / `aria-label` relationships, responsive behavior, and relative document structure.

Load the pinned `@gcds-core/components@1.5.0` stylesheet and module from `https://cdn.design-system.canada.ca/`. A small local style block is allowed only as represented by `examples/report-example.html`. Do not add an external or separate custom stylesheet, and do not replace GCDS components with custom controls or a parallel component system. Do not use `gcds-card` for deployment facts because cards are navigational. Use colon-free report titles.

### Markdown template

```
# CanadaLogin Test Release Notes
YYYY-MM-DD to YYYY-MM-DD

AI-GENERATED REPORT.

## Summary
Two or three plain sentences naming the period's most meaningful Test deployments and user-facing progress. Include additional evidence-backed validation detail when it is useful to the Test audience.

## Deployments

### Migration App

#### TEST
- `v1.2.3` unchanged

### Manage App

#### TEST
- `v1.2.3` deployed YYYY-MM-DD:
  - Concrete user-facing change on <page / flow / control>, so <benefit>. (Identity-verification work is excluded from this report.)

### IBM SaaS Configuration, Branding and Flows

#### TEST

##### Flows
- Sign Up Flow — updated in Test on YYYY-MM-DD: concrete change and benefit.

## Notes
- The range is inclusive and uses UTC. This report covers committed Test evidence only.
```

Group each app's single report-environment status and changes together. Include an Under the hood subsection only when at least one qualifying non-user-facing change exists for that environment. Use the same category grouping for IBM Attributes and Policies when those categories change. Within Policies, combine changed policy components into one `Access policies` summary per the IBM Component Naming rule. Omit empty change categories. Test and Staging reports may contain more distinct, concrete detail than Production reports when supported by the evidence. The Production report keeps the concise Production structure and wording rules and is titled `CanadaLogin Production Release Notes` (no colon).

Render this template once for Test, once for Staging, and once for Production, replacing the environment heading, status, changes, summary, and Notes with evidence for that environment only. Use the `production` filename stem and existing report title for the Production rendering.

## Quality Checks

Before writing the artifacts, verify:

- The selected environment has one dated report in Markdown and HTML for each sub-range. When no environment selector is supplied, three dated report files exist for each sub-range: Test, Staging, and Production, each in Markdown and HTML.
- All three repositories appear under Deployments in every generated report, and each report contains only its named environment with a status of `deployed`, `unchanged`, or `No deployment recorded this period` per Writing Style rules.
- Every displayed version begins with `v`; no previous versions are shown.
- Every user-facing change bullet names a concrete change and, where the diff supports it, its practical benefit — no forbidden generic wording.
- Every IBM Test, Staging, or Production change names the concrete configuration or behavior change; a tag-only line is not sufficient.
- Every Under the hood subsection sits inside a specific environment change group and is absent when that environment has no qualifying non-user-facing change.
- Reverted-and-restored work is present only when a distinct final behavior remains at period end.
- All deployment claims have dated Git evidence, a dated IBM state-table entry, or an authoritative GitHub record; web-app evidence is resolved from the default branch only.
- The report contains no commit IDs, PR numbers, commit links, PR links, repository URLs, or wiki revision identifiers.
- Manage App identity-verification content is absent everywhere in the report.
- The selected environment's dated report files are written to the workspace root with the correct filenames, and each file contains no other environment's content. With no selector, all six dated report files are present.
- The HTML reports follow `examples/report-example.html`, load the pinned GCDS assets from the official CDN, and apply `.is-deployed` / `.is-unchanged` accents based on status text rather than environment name.
- When a cadence was supplied, the sub-ranges together cover the requested range with no gaps or overlaps; each sub-range has reports for the selected environment(s); and the archive lists one entry per sub-range's date range.
- `index.html` exists at the workspace root; it preserves the `examples/index-example.html` structure and inline styles, links to every Test, Staging, and Production `canadalogin-release-notes-*.html` file currently on disk grouped by date range, and every referenced link resolves to an existing file.
