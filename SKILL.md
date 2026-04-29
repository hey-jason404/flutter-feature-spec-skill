---
name: feature-spec
description: |
  Use whenever the user wants to create or edit a Feature Spec for a
  Flutter project that follows this convention, especially when
  reverse-engineering an Android Activity / Fragment / ViewModel /
  Composable into a Flutter spec.

  Triggers include:
  - editing files under `src/docs/use-cases/`
  - mentioning identifiers like SCREEN-XX, BLOCK-X, UC-X
  - pointing at Android source code and asking for a Flutter spec
  - asking "how should I document this page / feature"

  Produces markdown only. Never generates tests or implementation code.
---

# Feature Spec

Reverse-engineer Android source into Flutter Feature Specs through an
interactive flow: read source → ask 2–3 key questions → produce draft →
mark every uncertainty as an Open Question.

## Three spec types

| Type | What it documents | When to write |
|------|-------------------|---------------|
| **SCREEN-XX** | A page (container) | One per Android Activity/Fragment/Composable screen |
| **BLOCK-XX** | A cross-page UI block | Only when the same block appears on the **3rd** page (Rule of Three). Do not extract earlier. |
| **UC-XX** | A cross-page flow | Multi-step / multi-screen flow (OTP, logout, payment selection) |

IDs use a two-digit zero-padded number per type (`SCREEN-01`,
`BLOCK-01`, `UC-01`). Each type has its own counter — `SCREEN-01` and
`BLOCK-01` can coexist. IDs are permanent: once issued, never rename
or recycle, even if a spec is deprecated. They are referenced from
commits, branches, and other specs.

## File layout

```
src/docs/use-cases/
├── README.md           # index of every spec, created on first run
├── pages/              # SCREEN-XX
├── blocks/             # BLOCK-X
└── flows/              # UC-X
```

File naming: `{TYPE}-{ID}_{slug}.md`, slug in `snake_case`.

## Document format (frontmatter + four sections)

Every spec uses the templates in `templates/{SCREEN,BLOCK,UC}.md`. The
shape is fixed:

1. **Frontmatter** — `id`, `title`, `status` (draft / in-review /
   implemented / deprecated), `last_updated`, `domain`, `android_ref`,
   `flutter_ref`, plus `contains` (SCREEN) or `appears_in` (BLOCK / UC).
   `flutter_ref` may be the anticipated path before implementation
   exists. `domain` is a string for single-domain specs or an array
   for cross-domain SCREENs (e.g. `[auth, push]`); it is the only
   source of truth for README grouping — filenames never encode the
   domain.
2. **📋 What & Why** — purpose, entry points, preconditions. PM/PO read
   this section.
3. **🧪 Scenarios** — Gherkin-style with **Given / When / Then** in
   bold. One scenario per row, max 6 lines each. QA reads this section.
4. **🔧 Implementation** — API table, state, error mapping, Android
   reference notes. Dev reads this section.
5. **❓ Open Questions** — every uncertainty goes here. Never silently
   guess.

Each section starts with the emoji header so readers can scan to their
zone. Do not omit the emoji — it is part of the navigation contract.

## Interactive flow

### Step 0 — Decide: new spec, or update an existing one?

- **New** → continue with Step 1.
- **Update an existing spec** (Android code changed, behavior tweak,
  status change): Read the existing spec first. Then read the
  corresponding Android source. Diff in your head — record changes as
  edits to the existing scenarios, and put any uncertain delta into
  Open Questions. **Do not rewrite from scratch** and do not change
  the spec ID.

### Step 1 — Read the Android source (bounded)

When the user gives an Activity / Fragment / ViewModel / Composable
path, read with a hard scope to avoid runaway exploration:

- The given file itself.
- Its **directly** referenced ViewModel, Repository, and DataSource
  (one hop only — do not follow chains further).
- Retrofit interface: read method **signatures** for endpoints called
  by this screen; do not read the implementation classes.
- DTO / response models: only if needed to understand a scenario
  outcome.

Do not read: unrelated screens, base classes / framework utilities,
DI modules, generated code.

Extract:
- **Entry points** — who navigates here (`startActivity`,
  `NavController.navigate`, deep link).
- **APIs** — Retrofit endpoints actually called in this code path.
- **UI interactions** — buttons, taps, text inputs, list items.
- **Navigation targets** — where this screen jumps to.
- **Error handling** — `catch` blocks, error states emitted.

### Step 2 — Ask 2–3 key questions before writing

After reading, surface only the decisions that block a good draft:

- Type confirmation: is this a single-page **SCREEN**, or a
  multi-screen **UC** flow? (BLOCK extraction is **not** considered at
  this stage — Rule of Three handles that later.)
- Android features that may be dropped or kept in the new app.
- Code that signals intent you cannot infer (feature flags, mysterious
  conditionals, `TODO` / `FIXME` comments).

Do **not** dump 10 questions. Do **not** silently produce a 100%
finished doc — uncertainty is the norm in reverse-engineering.

### Step 3 — Produce the draft

- Ensure `src/docs/use-cases/` exists, plus the `pages/`, `blocks/`,
  and `flows/` subdirectories. Create them if missing.
- Compute next ID by scanning filenames under
  `src/docs/use-cases/{pages,blocks,flows}/` for the relevant type and
  taking max + 1 (zero-padded to two digits). The README index is for
  humans and may lag — never trust it as the source of truth for
  numbering.
- Copy the matching template (`templates/SCREEN.md` etc.).
- Fill frontmatter: `id`, `title`, `status: draft`, `last_updated`
  (today), `android_ref`, `flutter_ref` (anticipated Flutter location,
  e.g. `lib/features/profile/`).
- One scenario per observable UI interaction.
- Every uncertainty → an Open Question line. Do not tag individual
  scenarios as draft — the file-level `status` already conveys that.
- Write to `src/docs/use-cases/{pages|blocks|flows}/`.
- Update `src/docs/use-cases/README.md` index. Create the README on
  first run if it does not exist (see "Index format" below).

### Step 4 — Report back

Tell the user:

- File path and ID issued.
- Number of Open Questions remaining (so they know what to follow up).
- **Cross-domain warning** — if APIs come from ≥ 2 distinct Android
  packages, flag that this screen may need BLOCK extraction later. Do
  **not** auto-extract; just note it.
- **Duplicate-extraction heuristic** — only run this check when the
  total SCREEN count under `src/docs/use-cases/pages/` is **≥ 3**
  (otherwise Rule of Three cannot fire). Grep existing SCREEN specs
  for each API endpoint in the new spec. For any endpoint that
  appears in **≥ 2 existing specs** (i.e. with this new one it would
  hit ≥ 3), list it as a BLOCK extraction candidate with the SCREEN
  IDs it shows up in, and ask the user whether to extract. Skip the
  prompt entirely if no candidate hits the threshold — do **not**
  produce a "no duplicates found" line, that is just noise. Still
  **never** auto-extract.

## Extracting BLOCK / UC (Rule of Three)

Only when the user explicitly asks ("找重複", "should this be a
BLOCK?", "extract this"):

1. Grep all SCREEN specs for repeated API endpoints, UI element names,
   navigation targets.
2. List candidates with the SCREENs they appear in. Require **≥ 3
   occurrences** before recommending extraction.
3. After user confirms a candidate:
   - Create the BLOCK / UC spec from its template.
   - In each parent SCREEN, replace the duplicated section with a
     one-line link: `→ see BLOCK-01 (錢包餘額卡片)`.
   - Update `contains` / `appears_in` frontmatter on both sides.
   - Update README.md.

Never auto-extract. Never extract on the 2nd occurrence.

## Validation (run on demand)

When asked to validate, check:

- Every `contains: [BLOCK-XX]` references a file that exists.
- Every `appears_in: [SCREEN-XX]` references a file that exists.
- Bidirectional symmetry: `SCREEN-02.contains` includes `BLOCK-01` ⇔
  `BLOCK-01.appears_in` includes `SCREEN-02`.
- README index covers every spec file.
- No duplicate IDs.
- All required frontmatter fields present (`id`, `title`, `status`,
  `domain`).
- README grouping matches each spec's `domain` field. A spec with
  `domain: [auth, push]` appears under both groups in the index.

## Index format (`src/docs/use-cases/README.md`)

When creating or updating, use this shape:

```markdown
# Feature Specs

## Numbering rules
- `SCREEN-XX` — page-level container
- `BLOCK-XX` — cross-page UI block (extract on 3rd occurrence)
- `UC-XX` — cross-page flow

Two-digit zero-padded numbers per type. Each type has its own counter.
IDs are permanent. Moving a file between folders does not renumber.

## Status
- `draft` — being written
- `in-review` — waiting on PM/QA confirmation
- `implemented` — Flutter implementation merged
- `deprecated` — feature removed

## Specs
<!-- Grouped by frontmatter `domain`. Cross-domain specs (domain is an
array) appear under each group. -->

### Auth
- [SCREEN-01 登入](./pages/SCREEN-01_login.md)

### Profile
- [SCREEN-03 個人資料](./pages/SCREEN-03_profile.md)

### Push
- [SCREEN-01 登入](./pages/SCREEN-01_login.md)  <!-- cross-domain -->

### Shared blocks
- [BLOCK-01 錢包餘額卡片](./blocks/BLOCK-01_wallet_balance.md)

### Shared flows
- [UC-01 OTP 驗證](./flows/UC-01_otp_verify.md)
```

## Hard rules

- Never write tests.
- Never invent API endpoints — only record what the Android code
  actually calls.
- Never pretend to understand business intent — uncertainties go to
  Open Questions.
- Never auto-extract BLOCK / UC — wait for the 3rd occurrence and an
  explicit user request.
- Never rename or recycle existing IDs — they are referenced
  externally.
- Never restate scenario behavior inside the Implementation section
  (drift risk).
- Never write Flutter implementation suggestions — specs describe
  behavior, not code.
- Status defaults to `draft`. The user changes it after review.
- Always update `src/docs/use-cases/README.md` when adding, renaming,
  or moving a spec.
- Always keep `contains` / `appears_in` symmetric.

## Anti-patterns

- Stuffing multiple independent features into one SCREEN — when a hub
  page grows past ~10 scenarios, split into BLOCK / UC instead.
- Copying Android implementation details (which `ViewModel` calls
  which `Repository`) into the spec body — those go only in the
  short Android reference note inside Implementation.
- Writing the Implementation section as "how to build it in Flutter" —
  that belongs in a future plan, not the spec.
- Writing scenarios that cannot be verified by observing the app from
  the outside (e.g. "internal cache is invalidated") — describe the
  observable behavior instead.
