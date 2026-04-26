---
name: flutter-view-refactor
description: Refactors long Flutter/Dart view/widget files to improve readability and maintainability without changing UI/UX. Applies Clean Architecture organization, SOLID, and DRY; extracts widgets into their own classes, removes unused code, reduces duplication, adds concise flow comments, and reuses existing widgets first. Use when the user mentions one or more Flutter/Dart file paths to refactor or asks to improve a screen/view without affecting how it looks or behaves.
---

# Flutter View Refactor

Turn repeated “refactor this view” prompts into a consistent, safe workflow for Flutter/Dart presentation code.

## Non-negotiables

- **Preserve UI/UX**: Do not change layout, spacing, styling, copy, gestures, focus behavior, scroll position behavior, transitions/animations, loading/error/empty states, accessibility semantics, or responsive/adaptive behavior unless the user explicitly requests a UI change.
- **Preserve behavior**: Do not change business logic outcomes, navigation, analytics, localization, theming, feature flags, or state flow.
- **Reuse-first**: Before creating new widgets/utilities, search the nearby feature/module for existing reusable widgets/helpers and prefer reusing them.
- **Plan first; implement only after approval**: Always produce a refactor plan first. Only edit code after the user explicitly approves implementation.
- **Scope control**: Refactor only what’s needed for readability/maintainability. Avoid speculative abstractions.

## When to use

Use this skill when the user:

- Mentions one or more Flutter/Dart file paths and asks to refactor/improve readability/maintainability.
- Mentions Clean Architecture, SOLID, DRY, “too long class”, “extract widgets”, or “don’t change how the view works/looks”.
- Wants the same refactor guidance repeated across multiple files.

## Inputs and file handling

- Treat **every file path the user mentions** as a target. If multiple are mentioned, process them in order and keep outputs clearly separated per file.
- For each target file, also inspect nearby likely reuse candidates (same feature folder) before proposing new components:
  - `widgets/`, `components/`, `common/`, `shared/`, `ui/`, `presentation/`
  - theme/style helpers, spacing constants, typography, buttons, tiles, cards, dialogs, sheets

## Workflow

### Phase 1 — Plan (always)

For each mentioned file:

1. **Read & understand**
   - Identify responsibilities mixed in the file (layout, orchestration, state wiring, widget composition, formatting, helpers).
   - Identify repetition and dead/unused code.
2. **Reuse scan**
   - List existing widgets/utilities that could be reused instead of creating new ones.
3. **UI/UX impact evaluation (mandatory)**
   - For every proposed change, explicitly state why it is **UI/UX-neutral**.
   - Watch-outs: `const` placement, keys, widget identity, `BuildContext` usage, `MediaQuery`/`LayoutBuilder`, `Animated*`, `Hero`, `Semantics`, `Focus`, `ScrollController`, slivers.
4. **Plan the refactor**
   - Extract widgets into their own `Widget` classes when it improves clarity **without harming performance/rendering**.
   - Keep “method widgets” only when extraction would increase rebuild cost, complicate keying, or capture too much context.
   - Remove unused code/imports.
   - Reduce duplication with small, local helpers/widgets (prefer existing ones).
   - Add **small, concise** flow comments only where the intent is not obvious.

### Phase 2 — Implement (only after explicit approval)

After user approval:

- Apply the plan with minimal diffs.
- Keep public APIs stable unless the plan explicitly calls out a safe rename and user approved it.
- Re-run available checks (examples, depending on repo):
  - `flutter analyze` (or `fvm flutter analyze`)
  - targeted tests for the feature if they exist

## Output template (use exactly)

For each file, produce this structure:

### File
- Path: `<path>`

### UI/UX impact check
- **Baseline UI/UX invariants**: (bullet list of what must not change)
- **Planned changes that could affect UI/UX**: (list + why neutral)
- **High-risk areas to double-check**: (keys, animations, semantics, scroll, responsive)

### Current issues
- (bullets)

### Existing reuse candidates (reuse-first)
- (bullets with paths/symbol names)

### Proposed refactor plan
- **Widget extraction**: (what becomes its own class; suggested names; file placement)
- **Remove unused code**: (imports, fields, methods)
- **DRY opportunities**: (duplicated layouts/styles; propose reuse of existing widgets first)
- **Clean Architecture/SOLID notes**: (only what applies to the file; keep UI thin)
- **Comments to add**: (short, intent-focused)

### Verification plan
- (how to confirm no UI/UX changes; what to run/check)

### Risk notes
- (what could accidentally change behavior and how you’ll avoid it)

## Implementation checklist (after approval)

- [ ] Extract widgets into dedicated classes where it improves clarity and doesn’t change rebuild behavior.
- [ ] Keep key/identity behavior stable (`Key`, `ValueKey`, `PageStorageKey`, `Hero` tags).
- [ ] Preserve semantics/focus/keyboard navigation behavior (`Semantics`, `FocusNode`, `Actions/Shortcuts`).
- [ ] Preserve animations and transitions (`Animated*`, `TweenAnimationBuilder`, routes).
- [ ] Remove unused imports/methods/fields safely.
- [ ] Replace duplication with reuse-first approach (prefer existing widgets/utilities).
- [ ] Add only minimal flow comments (no narration).
- [ ] Run analysis/tests if available and fix any introduced issues.

