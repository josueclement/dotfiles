# FEATURE-009 — dotnet-release TFM normalization

**Status:** TODO

_Single-phase item. Planned via `/interview` (v7); build with `/build FEATURE-009`. Documentation/prompt change — no build/test step._

## Objective

Give the `dotnet-release` skill a **target-framework check/update** step: preserve any `netstandard*`
target, and drive the modern-.NET portion of the packable library's TFM set to **exactly
`net8.0` + `net10.0`** (the two current LTS releases), replacing any `net*` older than 8. Reconcile
`dotnet-solution-setup`, which currently calls `net10.0` the sole "current" .NET, so the two skills
agree on the supported set.

## Context

The `dotnet-release` skill (delivered by FEATURE-007) drives a NuGet library release end-to-end, but
it has **no** logic to inspect or change a project's `<TargetFramework(s)>`. It only *reacts* to an
already-changed TFM set — it updates a README "supported target frameworks" line "if the TFM set
changed this release" and its runbook builds "across all TFMs" — but nothing detects or performs that
change; it relies on the operator having done it by hand. This item closes that gap.

The house policy being introduced: **`netstandard*` is preserved (max consumer compatibility); the
modern-.NET set is `net8.0` + `net10.0` "for now"** — precisely the two active LTS releases (net8 LTS
through Nov 2026, net10 LTS from Nov 2025); a `net*` target older than 8 is replaced with the pair.

A secondary inconsistency surfaced in the interview: `dotnet-solution-setup` declares `net10.0` as the
**sole** current .NET (no `net8.0`; multi-target example `netstandard2.0;net10.0`), contradicting the
new policy — so this item also **reconciles** that skill.

## Design decisions (validated in the interview)

1. **Skill scope:** add the TFM step to `dotnet-release` **and reconcile `dotnet-solution-setup`** so
   its "current .NET" wording and library multi-target example acknowledge `net8.0` + `net10.0` as the
   supported LTS pair. (App/test defaults stay single-target `net10.0` — apps ship one runtime.)
2. **Project scope:** the normalization applies to the **packable library** `.csproj` only (the one
   with `<PackageId>` being packed). Apps/CLIs keep their own single TFM.
3. **Normalization rule:** the `net*` portion of the library's TFM set becomes **exactly
   `{net8.0, net10.0}`** — ensure both present, replace any net older than 8, collapse any other net
   (e.g. `net9.0`) to the pair. `netstandard*` is always preserved. Target behaviour:

   | before | after |
   |---|---|
   | `net8.0` | `net8.0;net10.0` |
   | `netstandard2.0;net8.0` | `netstandard2.0;net8.0;net10.0` |
   | `net6.0` | `net8.0;net10.0` |
   | `net9.0` | `net8.0;net10.0` |
   | `net10.0` | `net8.0;net10.0` |

4. **netstandard-only** (no `net*` target at all): **left as pure netstandard** — do NOT force-add
   net8/net10 (a netstandard2.0 lib already runs everywhere; adding net targets is maintenance surface
   for no gain). The net-normalization only fires once a plain-net target already exists.
5. **Platform-specific TFMs** (`-windows`, `-android`, … suffix): **left untouched** (rewriting them
   would strip the platform surface and break the library); the skill **prints a warning** if it sees
   one older than net8, for manual review.
6. **Apply mode:** **propose-and-confirm** — the skill shows the `old → new` TFM diff (and any platform
   warning) and **waits for the user's OK before writing**. A deliberate departure from the skill's
   usual "in-repo edits the skill makes" model, because a TFM change alters the compatibility surface
   (and a drop can break consumers); it must be worded explicitly.
7. **Audit trail (on confirm):** record `old → new` in `RELEASENOTES.md` under a **Compatibility**
   sub-section; a **dropped** TFM (replace-older-than-8) is flagged as a **breaking/compat** note.
   Update the README **"supported target frameworks"** line if present. Closes the loop the skill
   already anticipated but never fed.
8. **`<TargetFramework>` → `<TargetFrameworks>`:** when the result has >1 TFM and the csproj used the
   singular element, convert to the plural (`;`-separated) form. TFM order: `netstandard*` first, then
   ascending net, then any preserved platform-specific TFM.
9. **"for now" framing:** the skill states `net8.0`/`net10.0` are the *current* policy values (the two
   supported LTS releases) and are expected to change over time.
10. **Supporting updates (all selected):** RELEASE.md pre-release checkbox · Common-mistakes row ·
    bidirectional cross-reference between `dotnet-release` and `dotnet-solution-setup`.

## Scope

### 1. `/home/jo/dotfiles2/.claude/skills/dotnet-release/SKILL.md` — new TFM check/update step

- Add a new step in the **In-repo edits** section (before/alongside the `<Version>` bump), titled e.g.
  **"Target frameworks"**, specifying the algorithm in decisions 2–9: partition into netstandard
  (preserve) / platform-specific (preserve, warn if <net8) / plain-net (normalize to `net8.0;net10.0`
  when non-empty; leave alone when empty), singular→plural conversion, and the
  **propose→confirm→then-edit** flow.
- Wire the outcome into the existing README "supported target frameworks" line and a `RELEASENOTES.md`
  **Compatibility** bullet (old→new; drop = breaking note).
- Cross-reference `dotnet-solution-setup` as the TFM-choice authority.
- Add a **Common mistakes** row: "TFM set changed but README supported-TFMs line / RELEASENOTES
  Compatibility note not updated → keep the audit trail in sync."

### 2. `/home/jo/dotfiles2/.claude/skills/dotnet-release/templates/RELEASE.md` — pre-release checkbox

- Add a checkbox in **Pre-release checks** (near "build across all TFMs"), e.g. "`<TargetFrameworks>`
  reflect the net8.0+net10.0 policy for X.Y.Z; `netstandard*` preserved."

### 3. `/home/jo/dotfiles2/.claude/skills/dotnet-solution-setup/SKILL.md` — reconcile TFM policy

- Update the **Target frameworks** section so the "current .NET" framing acknowledges **net8.0 and
  net10.0** as the supported LTS pair, and change the library multi-target deviation example from
  `netstandard2.0;net10.0` to `netstandard2.0;net8.0;net10.0`. Keep app/test defaults single-target
  `net10.0` (WPF `net10.0-windows`).
- Adjust the related **Common mistakes** row if it hard-codes net10.0-only.
- Add a cross-reference to `dotnet-release` for the release-time normalization policy.

## Acceptance criteria

1. `dotnet-release/SKILL.md` has a TFM check/update step implementing decisions 2–9, including the
   propose→confirm flow, netstandard-preserve, platform-untouched-with-warning, singular→plural, and
   the RELEASENOTES Compatibility + README supported-TFMs wiring.
2. The step's algorithm, walked against the decision-3 table plus the two edge cases (netstandard-only
   → unchanged; `net6.0-windows` → unchanged + warning), yields the stated outputs.
3. `templates/RELEASE.md` has the matching pre-release checkbox; `dotnet-release` Common mistakes has
   the new row.
4. `dotnet-solution-setup/SKILL.md` no longer contradicts the net8+net10 policy; its multi-target
   example reads `netstandard2.0;net8.0;net10.0`; app/test defaults unchanged.
5. Bidirectional cross-references resolve between the two skills; terse house style preserved;
   frontmatter untouched.
6. Roadmap + plan-file statuses updated and `docs/done/FEATURE-009.md` written on completion, stating
   this is a documentation/prompt change with no build/test step (verified by inspection).

## Notes & follow-ups

- No build/test step (documentation change): DoD criteria 1–2 are satisfied by well-formed artifacts
  verified by inspection (walk the normalization algorithm against the decision-3 table + both edge
  cases) — state this in `docs/done/FEATURE-009.md`.
- The `net8.0`/`net10.0` values are policy "for now" (current LTS pair); revisit when the LTS set moves
  (e.g. net12 lands / net8 exits support).
- Deliberately out of scope: auto-rewriting platform-specific TFMs (warn only); forcing net targets
  onto netstandard-only libraries; applying normalization to non-packable apps/CLIs.
