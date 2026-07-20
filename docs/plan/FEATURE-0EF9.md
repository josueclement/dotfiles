# FEATURE-0EF9 — Random-hex work-item IDs

**Status:** DONE
**Type:** FEATURE (single-phase)
**Branch:** `feature/feature-0ef9-random-hex-ids`

## Objective

Replace the sequential per-type counter in the work-item ID scheme
(`FEATURE-NNN` / `BUG-NNN` / `CODE-REVIEW-NNN`, 3-digit zero-padded) with a
**random 4-digit uppercase hex value** (`FEATURE-HHHH`, e.g. `FEATURE-3A7F`).
Sequential numbers collide when multiple devs allocate IDs concurrently on the
same repo (each takes "the next number" and clashes on merge). Random IDs remove
the shared counter, so concurrent allocation is collision-safe. Existing IDs are
**not migrated** — old `NNN` IDs and new `HHHH` IDs coexist.

## Scope

### Files changed
1. **`.claude/skills/dev-workflow/SKILL.md`** — the scheme's home: rewrite the
   "Work-item identifiers" allocation rules, update the frontmatter description,
   update every illustrative example / branch-name / commit-scope example, and
   bump the version.
2. **`.claude/commands/interview.md`** — replace the `CODE-REVIEW-NNN` prose and
   bump the announce line.
3. **`.claude/commands/build.md`** — update the `FEATURE-001`-style illustrative
   examples and bump the announce line.
4. **`.claude/skills/dotnet-release/SKILL.md`** — replace the two `FEATURE-NNN`
   placeholder references so the docs stay consistent.

### Explicitly out of scope (no migration)
- `docs/roadmap.md` existing rows (`FEATURE-001` … `FEATURE-010`) stay as-is.
- All `docs/plan/*` and `docs/done/*` historical files stay as-is.

## Design — the new scheme

- **Format:** `FEATURE-HHHH` / `BUG-HHHH` / `CODE-REVIEW-HHHH`, where `HHHH` is a
  random 4-digit **uppercase** hex value (`0-9`, `A-F`) — 65 536 values per type.
- **Generation:** via a shell entropy source, uppercase — primary
  `printf '%04X\n' $((RANDOM % 65536))` (already uppercase), alternative
  `openssl rand -hex 2 | tr '[:lower:]' '[:upper:]'`. The skill must not invent a
  value itself (LLM-picked hex is non-uniform and defeats the purpose).
- **Uniqueness:** **per type**. Generate → check the hex isn't already used by an
  item of the *same type* in the roadmap's ID column or as a stray file in
  `docs/plan/` (including `ABANDONED` rows) → regenerate on any collision.
  `FEATURE-3A7F` and `BUG-3A7F` may coexist (the type prefix disambiguates).
- **Ordering:** the roadmap's row order is the canonical order (random IDs carry
  no sequence). **New items append at the bottom.** `/build` keeps recommending
  the topmost `TODO`/`IN PROGRESS` (unchanged → oldest-planned first). The old
  "take the highest number + 1 / never reuse a number" allocation text is deleted.
- **Phases unchanged:** phase suffixes stay sequential 2-digit (`-PHASE01`,
  `-PHASE02`); only the **base** ID becomes random (phases live inside one item
  owned by one dev, so they never collide across devs).
- **Placeholder token in prose:** `HHHH` (direct replacement for `NNN`).

## Versioning (per user decision — bump all three)
- `dev-workflow` skill: **v2 → v3** (`**Version: dev-workflow v3.**`).
- `/interview`: announce line **v7 → v8** (`Using interview v8 by Josué Clément`).
- `/build`: announce line **v4 → v5** (`Using build v5 by Josué Clément`).

## Acceptance criteria
1. `grep -rn "NNN" .claude/skills .claude/commands` returns nothing.
2. The `dev-workflow` allocation section describes: random 4-digit uppercase hex,
   shell-generated, per-type uniqueness + regenerate-on-collision, phases still
   sequential, row-order-is-canonical + append-at-bottom; and the old
   "highest + 1 / never reuse" counter text is gone.
3. Every illustrative example, branch name, and commit scope across all four
   files uses hex IDs (no `FEATURE-001`-style sequential examples remain).
4. Announce/version lines updated: dev-workflow v3, `/interview` v8, `/build` v5.
5. No existing committed roadmap/plan/done data changed (no migration).
6. All four files remain well-formed markdown (valid tables, intact frontmatter).

## Verification (no build/test — prompt/doc changes)
There is nothing to compile or unit-test. Criteria are verified by: the `grep`
checks in AC1–AC3, and visual inspection of the rendered markdown / frontmatter.
