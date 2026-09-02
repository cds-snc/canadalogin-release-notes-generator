---
name: canadalogin-release-notes
description: "Maintain separate cumulative CanadaLogin What's New pages for Test, Staging, and Production, adding concise changes since each page's last update."
argument-hint: "Optional: provide an environment (test, staging, or prod) and dates, for example: environment=staging since=2026-08-01 to=2026-08-31"
user-invocable: true
---

# CanadaLogin What's New

Maintain one cumulative What's New page for each environment across these products:

| Repository directory | Product name |
|---|---|
| `canadalogin-migration` | Migration App |
| `canadalogin-user-selfservice-webapp` | Manage App |
| `gc-signin-ibm-configuration` | Sign-in, sign-up, recovery flows |

The pages follow a simple release-notes feed: the newest dated update is at the top, each update contains concise user-facing bullets, and older updates remain unchanged below it.

## Inputs and update boundary

1. Parse optional `environment=test|staging|prod`, `since=YYYY-MM-DD`, `from=YYYY-MM-DD`, `to=YYYY-MM-DD`, and `until=YYYY-MM-DD` arguments. `from` and `since` are aliases; `to` and `until` are aliases. Treat `production` as an alias for `prod`. Ordinary phrasing such as `Update staging since August 1 through August 31` is also supported.
2. If an environment is supplied, update only that environment. Without one, update all three environments. Do not infer an environment from incidental words in a product name or change description.
3. Read the existing page for each selected environment before gathering evidence. Its marker is the source of truth:
   - Test: `<!-- CANADALOGIN-WHATS-NEW-TEST-THROUGH: YYYY-MM-DD -->`
   - Staging: `<!-- CANADALOGIN-WHATS-NEW-STAGING-THROUGH: YYYY-MM-DD -->`
   - Production: `<!-- CANADALOGIN-WHATS-NEW-PRODUCTION-THROUGH: YYYY-MM-DD -->`
4. If a page exists and no starting date was supplied, begin on the day after that page's marker. If a page does not exist, use the previous fourteen complete UTC calendar days. If a supplied start is on or before an existing marker, ask whether the user wants to rebuild that page; never duplicate an existing update by default.
5. If no end date was supplied, use the current UTC date. If the requested end date is later than today, clamp history lookups to today and add a note to the affected update explaining the clamp.
6. The interval is inclusive. There is no cadence, archive page, or cross-environment report. Each selected environment receives at most one new update section per run.

## Repository setup and evidence

1. Treat the three repositories and the IBM wiki as subdirectories of the current workspace. If one is missing or is not a Git repository, clone it directly into that path with complete history (`--fetch-depth=0`). Try SSH first and HTTPS second. Clone the wiki to `./gc-signin-ibm-configuration.wiki`. Do not use a workspace-parent or home-directory fallback.
2. Use read-only Git and GitHub operations. Never push, commit, tag, create or delete branches, create or update releases, open or modify PRs or issues, dispatch workflows, or modify source files, changelogs, deployment files, or wiki pages. Only the six What's New files named below may be created or updated.
3. For the two web applications, use the environment file matching the selected page: `.deployed_versions/test.json`, `.deployed_versions/staging.json`, or `.deployed_versions/prod.json`. If the repository uses `.deployed_versions/production.json`, use that for Production. Resolve files and history on the default branch only. A version-file edit is deployment evidence only when it is a reachable state change dated inside the interval.
4. For IBM Test, use committed `main` history at the interval boundaries and map changes under `Flows/`, `Branding/`, `Attributes/`, and `AccessPolicies/` to Flows, Themes, Attributes, and Policies. For IBM Staging and Production, use the wiki Deployment-State page exclusively and use the matching `staging Tag`/`staging Date` or `prod Tag`/`prod Date` values. A `-` value means unavailable.
5. A deployment requires dated evidence: a default-branch environment-file state change, a dated IBM state-table entry, or an authoritative GitHub deployment/release record. A currently present version is not by itself evidence of an in-period deployment.
6. Use the last committed state immediately before the interval as the comparison baseline. Inspect actual diffs between the baseline and the deployed state, not just changelog subjects or commit titles. Remove changes fully reverted during the interval.
7. Read the relevant `CHANGELOG.md` entries for deployed versions in the interval as candidate evidence. Use them to find likely user-facing changes and draft concise wording, but verify every candidate against the actual deployed diff and the selected environment's deployment evidence. Ignore changelog-only entries, unreleased versions, reverted changes, and items that are internal or otherwise excluded.
8. Exclude release-only commits, generated changelog edits, deployment bookkeeping, lock-file-only changes, formatting, comments, routine dependency bumps, and dev tooling. Exclude all Manage App identity-verification work, including related tests, documentation, dependencies, and deployment-only changes.

## Environment pages

Write these files at the workspace root:

| Environment | Markdown | HTML |
|---|---|---|
| Test | `canadalogin-releases-whats-new-test.md` | `canadalogin-releases-whats-new-test.html` |
| Staging | `canadalogin-releases-whats-new-staging.md` | `canadalogin-releases-whats-new-staging.html` |
| Production | `canadalogin-releases-whats-new-production.md` | `canadalogin-releases-whats-new-production.html` |

Each page is independent. Its title, marker, update headings, deployment descriptions, and notes must refer only to its environment. Never copy a Test or Staging change into Production without Production evidence, and never copy a Production change into a lower-environment page without evidence for that lower environment.

## Writing style

Write like concise product release notes. Prefer:

- `You can now ...` for a new capability.
- `We've improved ...` for an enhancement.
- `We've fixed an issue where ...` for a bug fix.
- `<Feature> now ...` when that makes the observable result clearer.

Lead with the product area or flow and describe the visible result. Give each bullet a concrete benefit when the evidence supports one. Keep bullets to one sentence where possible. Do not expose commit IDs, PR numbers, repository URLs, wiki revision IDs, raw configuration identifiers, or internal audit details.

Use friendly names: `Migration App`, `Manage App`, `Sign-in, sign-up, recovery flows`, `Sign In`, `Password Recovery`, `Sign Up`, and `Access policies`. Preserve well-known acronyms such as `MFA` and `OTP`. Do not include identity verification changes in any What's New page; the feature is still under development and behind a feature flag.

## Page format and update behavior

Use `examples/whats-new-example.md` and `examples/whats-new-example.html` as templates. The Markdown and HTML for each environment must contain the same factual content.

The Markdown shape is:

```markdown
# CanadaLogin Test What's New
<!-- CANADALOGIN-WHATS-NEW-TEST-THROUGH: YYYY-MM-DD -->

## Month D, YYYY

### Migration App
`v1.2.3` deployed to Test on Month D, YYYY
- You can now ...

### Manage App
`v1.4.5` deployed to Test on Month D, YYYY
- We've improved ...

### Sign-in, sign-up, recovery flows
- The Sign In flow now ...
```

Use `Test`, `Staging`, or `Production` consistently in the page title, marker, and deployment lines. For Production, use the `production` filename stem shown above. When multiple deployments fall inside one interval, keep one dated update heading and group the affected product areas beneath it. Include a version line only when the evidence provides a version. Group IBM changes by friendly product area rather than raw file or policy name. Omit empty product headings and categories.

On every update for each selected environment:

1. Preserve the existing page title, attribution, marker format, and all older update sections.
2. Replace only that page's through-date marker with its new effective end date.
3. Insert the new update section immediately below the attribution and above the previous update section. Do not rewrite, reorder, or deduplicate older sections.
4. If the interval contains no qualifying changes for that environment, advance its marker and add no empty update section. Report that no What's New entry was required for that environment.
5. If that page's marker already covers the requested interval, make no content change and report that the page is already current.
6. If a deployment has no included user-facing change, include its product and version line followed by `No included user-facing changes.` Do not invent a benefit.

The HTML is an accessible rendering of the same page. Start from `examples/whats-new-example.html`, preserve its pinned GCDS assets, component choices, relative structure, responsive behavior, and inline style block. Use a GCDS `h1`, date headings, `gcds-heading`, `gcds-text`, ordinary lists, and semantic `section` elements. Do not add a hero, archive index, environment cards, or a separate custom stylesheet. Include the environment-specific marker in an HTML comment.

## Quality checks

Before writing, verify:

- Each selected page begins strictly after its own previous through-date marker unless it is a first page.
- Each page contains one current through-date marker and no duplicate update for the same interval.
- New entries use evidence from that page's environment only and describe concrete, observable changes.
- No identity-verification content, audit identifiers, raw technical names, or fully reverted changes appear.
- Versions are shown only when supported by the selected environment's deployment evidence and are prefixed with `v`.
- The Markdown and HTML for each environment contain the same update headings, product names, version lines, and change bullets.
- The HTML parses, loads the pinned GCDS assets from `https://cdn.design-system.canada.ca/`, and preserves semantic heading relationships.
- Only the selected environment's two files were written, or all six files when no environment was supplied.
