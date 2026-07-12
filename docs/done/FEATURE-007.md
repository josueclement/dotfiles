# FEATURE-007 — `dotnet-release` skill — DONE

## Summary

Added a new Claude Code skill, `dotnet-release`, that drives a new NuGet library-version release
end-to-end: it makes the **in-repo edits** (bump `<Version>`, write `<PackageReleaseNotes>`, prepend
`RELEASENOTES.md` renaming any `(unreleased)` heading, refresh README badges + the what's-new callout,
apply selective non-coupled NuGet bumps in `Directory.Packages.props` while holding the version-coupled
Avalonia/Carbon/Phosphor sets) and then **prints — never runs —** the outward-facing runbook
(Release build/test, merge, `git tag`, `dotnet pack`, `dotnet nuget push`, post-publish verification).
The NuGet API key is never stored or echoed.

The skill is scoped as the **final phase of a `FEATURE-NNN` work item** and defers to `dev-workflow`
for branch/commit/tag ownership + the doc-freshness sweep, to `dotnet-solution-config` for the CPM file
it edits, and to `git-repo-hygiene` for line endings — cross-referencing rather than restating them.
A bundled `templates/RELEASE.md` (parameterized from the LicenseManager runbook) is offered into the
target repo's `docs/`, create-only-if-missing.

## Files/modules touched

**Created**
- `.claude/skills/dotnet-release/SKILL.md` — frontmatter (`name`+`description` only) + body: title/rule
  (release = final phase of a FEATURE item), When to use / Not for, Execution boundary (in-repo edits;
  print never run; no API key), In-repo edits (Version, PackageReleaseNotes, RELEASENOTES prepend/rename,
  README badges + callout + TFM line), NuGet package refresh (outdated check, non-coupled bumps, held-back
  coupled sets, `old → new` logging), Packable-library prerequisites (PackageId, README/LICENSE pack,
  PackageReleaseNotes, `GeneratePackageOnBuild` OFF), Runbook — print never run (+ detect tag/solution/
  default-branch), App vs library, Cross-references, `## Bundled files`, `## Common mistakes`, closing
  convention caveat.
- `.claude/skills/dotnet-release/templates/RELEASE.md` — the human release checklist (pre-flight → merge →
  tag → pack → push → verify), parameterized from `/home/jo/Dev/Enigma.LicenseManager/docs/RELEASE.md`
  with `{{PACKAGE_ID}}` / `{{SOLUTION}}` / `{{LIB_CSPROJ}}` / `{{LIB_DIR}}` / `{{DEFAULT_BRANCH}}`
  placeholders (a leading HTML-comment legend documents them). Standardized on `GeneratePackageOnBuild`
  OFF and detect/default-bare tags (see Deviations).
- `docs/done/FEATURE-007.md` — this record.

**Modified**
- `docs/roadmap.md` — FEATURE-007 → `IN PROGRESS`, then `DONE`.
- `docs/plan/FEATURE-007.md` — status → `IN PROGRESS`, then `DONE`.

## Deviations & follow-ups

- **No deviations from the plan's 11 validated design decisions.** Name matches the directory; in-repo-edit
  vs. print-only boundary honored; NuGet policy (outdated check + held-back coupled sets + `old → new`
  logging) and tag detect/default-bare + `GeneratePackageOnBuild` OFF applied; frontmatter/`## Common
  mistakes`/cross-reference conventions followed; template bundled un-dotted under `templates/`.
- **Known reference divergence (flagged in the plan, note 3).** The two source repos disagree —
  `Enigma.Cryptography` uses `GeneratePackageOnBuild` off + bare tags, the `Enigma.LicenseManager` runbook
  uses on + `v`-prefixed. The skill standardizes on **off + detect/default-bare** as planned; anyone
  aligning LicenseManager later should note this.
- **Three clarifying edits made after the dry-run** (gaps a real release hits; clarifications, not
  redesigns, and consistent with the plan): (a) the runbook now says use the library `.csproj` for the
  build/test pre-flight when the repo has **no solution** (single packable project); (b) added a
  **default-branch detection** note (`git remote show origin`; house repos use `main`/`master`), parallel
  to the existing tag-detection note, plus a line that the merge/tag/push steps assume an initialized repo
  with a remote; (c) rewrote the "version is touched in three places" sentence, which overstated the edits
  — the NuGet badge auto-tracks (leave it alone) and the TFM line only changes if the TFM set moved, so the
  real per-release edits are just `<Version>` + the callout.
- **Dry-run friction recorded but not actioned (follow-ups):** the dry-run agent also flagged (i) the
  "offer it" vs. "create only if missing" phrasing for `docs/RELEASE.md` reads ambiguously in a
  non-interactive run; (ii) whether the `Dependencies` bullet overrides pure "match existing sub-section
  style"; (iii) whether `<PackageReleaseNotes>` prose should include dependency bumps and how to handle
  long prose in one XML element; (iv) that the coupled-set "hold back by default" could be stated up front
  in the refresh section rather than only inferable from Common mistakes. These are minor wording
  refinements; left for a future pass to avoid scope creep beyond the plan.
- **Line endings (CRLF):** all authored files are LF / UTF-8 (verified — no CRLF in the skill).
  Recommendation only; normalization is the user's call per `dev-workflow` / `git-repo-hygiene`.

## Build/test evidence

Documentation/prompt artifact — there is no build step or test suite (DoD criteria 1–2 satisfied by the
well-formed-artifact equivalent). Verification performed:

- **Well-formedness:** frontmatter is `name`+`description` only, `name: dotnet-release` matches the
  directory; all planned body sections present; body ends with `## Common mistakes` + the closing caveat.
- **Acceptance criteria 1–5:** SKILL.md + `templates/RELEASE.md` exist with the required content; the skill
  makes only in-repo edits and prints (never runs) `git tag` / `dotnet pack` / `dotnet nuget push` with no
  API key stored; the NuGet-update policy (outdated + held-back coupled sets + `old → new` logging), tag
  detect/default-bare, and `GeneratePackageOnBuild` OFF are all documented; it cross-references
  `dev-workflow` / `dotnet-solution-config` / `git-repo-hygiene` without restating them.
- **Acceptance criterion 6 — dry-run (independently verified):** a fresh general-purpose agent, given only
  the skill and a sample library repo (csproj at `<Version>1.1.0</Version>` with no `<PackageReleaseNotes>`,
  a `(unreleased)` `RELEASENOTES.md`, README with badges + a 1.1 callout, and a CPM file mixing
  `Newtonsoft.Json` with a coupled Avalonia set), released **1.2.0**: it bumped `<Version>`, added
  `<PackageReleaseNotes>` ("… See RELEASENOTES.md …"), renamed the `(unreleased)` heading to `v1.2.0` and
  logged a `Dependencies` `old → new` block, updated the README callout (1.1 → 1.2), bumped
  `Newtonsoft.Json` 13.0.1 → 13.0.3 while **holding back** the coupled Avalonia set (logged as held),
  created `docs/RELEASE.md` from the template with placeholders filled, and **printed without executing**
  the tag/pack/push/build/test runbook (no API key materialized). All six acceptance criteria met.
