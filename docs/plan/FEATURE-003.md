# FEATURE-003 — `git-repo-hygiene` skill

**Status:** IN PROGRESS

_Single-phase item. Planned via `/interview` (v7); build with `/build FEATURE-003`._

## Objective

Add a reusable Claude Code skill, `git-repo-hygiene`, that scaffolds three cross-platform
repo-config files — `.gitignore`, `.gitattributes`, `.editorconfig` — into a .NET/Visual Studio
git repository, and documents CRLF→LF line-ending remediation (`git add --renormalize`).

## Context

The source content already exists: the `.gitignore` is the stock GitHub `VisualStudio.gitignore`
(what `dotnet new gitignore` emits) plus one custom `.idea/` Rider line, taken verbatim from
`/home/jo/Dev/JCL/Enigma.Cryptography/.gitignore`; the `.gitattributes` and `.editorconfig`
were provided verbatim in the request. This is a documentation/prompt artifact in the `dotfiles`
repo. No sibling git/config skill exists — the closest is `dev-workflow` (owns VCS + CRLF rules),
which this skill cross-references rather than restates.

## Design decisions (validated in the interview)

1. **Name:** `git-repo-hygiene` (noun/topic style, matching sibling skills; `name` matches the directory).
2. **.gitignore source:** bundle a **frozen copy** of the exact VS/dotnet `.gitignore` (incl. the custom `.idea/` Rider line) as a template the skill copies in — deterministic, offline, no SDK dependency. Refreshed manually if upstream drift matters.
3. **Existing-file behavior:** **create-only-if-missing** — write each of the three files only when absent; if present, leave untouched and report (optionally diff) so the user decides. Never clobber.
4. **Trigger scope:** `.NET/Visual Studio` repo **setup** + **CRLF/LF remediation** (the `git renormalize` use-case).
5. **.editorconfig content:** **minimal, verbatim** from the draft — EOL / charset / final-newline only; no code-style rules.
6. **House conventions applied:** frontmatter is only `name`+`description`; body ends with `## Common mistakes`; the skill cross-references `dev-workflow` for branch/commit/CRLF-ownership; the `git add --renormalize` sequence is **presented for the user to run** — the skill never auto-runs or commits (per `dev-workflow`, the user owns line-ending normalization and I never commit).
7. **Bundling (low-impact defaults):** the three templates live in a `templates/` subdirectory with **un-dotted** filenames (`templates/gitignore`, `templates/gitattributes`, `templates/editorconfig`) so a literal `.gitignore` inside the skill folder cannot act as a real ignore file in the `dotfiles` repo; the skill copies them to the dotted names in the target repo.
8. **Skill type:** Reference-type skill; verification = retrieval/dry-run at build time.

## Scope

Directory `/home/jo/dotfiles/.claude/skills/git-repo-hygiene/` (auto-exposed via the existing
`~/.claude/skills` directory symlink — no per-skill symlink needed):

- **`SKILL.md`** — new. Frontmatter + body (sections below).
- **`templates/gitignore`** — new. Verbatim copy of `Enigma.Cryptography/.gitignore` (VS template + `.idea/`, ~364 lines).
- **`templates/gitattributes`** — new. Verbatim from the draft.
- **`templates/editorconfig`** — new. Verbatim from the draft.

### SKILL.md content

- Frontmatter:
  - `name: git-repo-hygiene`
  - `description: Use when setting up or normalizing a .NET/Visual Studio git repository's text configuration — creating a .gitignore, .gitattributes, and .editorconfig, or fixing CRLF/LF line-ending churn (mixed endings, whole-file diffs) with git renormalize.`
- Body sections:
  1. `# git-repo-hygiene` + one-line house rule ("Give a .NET repo consistent, cross-platform text: LF-normalized files, correct binary handling, a VS ignore set, and editor defaults.").
  2. **When to use / When not** — new/existing .NET repo config; CRLF remediation. Not for non-.NET `.gitignore` needs.
  3. **The three files** — what each does and where its content comes from.
  4. **Workflow** — create-only-if-missing: copy each `templates/*` to its dotted name only if absent; if present, leave it and report/diff.
  5. **Normalizing line endings** — after `.gitattributes` is in place, present, for the user to run:
     ```
     git add --renormalize .
     git status
     git commit -m "Normalize all line endings to LF"
     ```
     Explicitly note the skill does not run/commit these; the user does (see `dev-workflow`).
  6. **`## Bundled files`** — links to `templates/`.
  7. Cross-reference: branching, commits, and CRLF ownership are governed by `dev-workflow`; this skill does not restate them.
  8. **`## Common mistakes`** — table: naming a template `.gitignore` with a dot; overwriting a customized ignore; committing normalization as part of unrelated work; expecting the skill to fire for non-.NET repos.

## Acceptance criteria

1. `git-repo-hygiene/SKILL.md` exists with valid `name`+`description`-only frontmatter and the sections above; `name` matches the directory.
2. `templates/gitignore`, `templates/gitattributes`, `templates/editorconfig` exist with the exact intended content (gitignore = Enigma's file byte-for-byte incl. `.idea/`).
3. SKILL.md documents create-only-if-missing behavior and the renormalize sequence as user-run (not auto-run).
4. SKILL.md cross-references `dev-workflow` and does not restate its VCS/CRLF rules.
5. Dry-run verification: a fresh agent, given the skill and a sample .NET repo, scaffolds the three files without clobbering pre-existing ones.

## Notes & follow-ups

- Licensing: the VS `.gitignore` derives from `github/gitignore` (CC0 / public domain) — safe to bundle.
- No build/test step (documentation/prompt artifact): DoD criteria 1–2 are met by a well-formed skill verified via the dry-run in criterion 5.
- Standing recommendation (line endings): `Enigma.Cryptography` currently has **no** `.gitattributes`/`.editorconfig` (gap already noted in `docs/done/FEATURE-002-PHASE01.md`); applying this skill there later would close it.
