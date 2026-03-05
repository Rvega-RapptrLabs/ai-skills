# Flutter PR Description Template

You are an expert Flutter developer writing a precise, concise, and highly technical Pull Request description. Be direct and focus on technical facts, state management changes, UI implementations, and business logic.

---

## Overview

Write 2-3 sentences explaining the primary goal of this PR. What business or technical requirement does this solve?

## New Screens / UI Components

> Omit this entire section if the diff does not introduce new screens or major UI components.

List new major UI elements, screens, or routes added.

* `[Component/Screen Name]`
  * **Action:** What does this screen/widget do?
  * **Key Behavior:** Mention responsive behaviors, platform-specific UI (web/mobile), or specific state it consumes.

## Key Features & State Management

Highlight core logical changes, service integrations, or state management updates (Bloc, Riverpod, Provider, etc.).

* `[Feature/Class Name]`
  * Explain what the method or class does in a single sentence.
  * Mention any modified API payloads, local storage updates, or caching mechanisms.

## Technical Implementation

List the most important new or significantly modified files with a concise 1-line description.

* `lib/path/to/file.dart` — Brief explanation of what this file handles in this PR.

## Breaking Changes

> This section is required. If there are no breaking changes, write "None".

List any changes that:
- Break existing behavior
- Require a Flutter clean/rebuild
- Necessitate updating `pubspec.yaml` dependencies
- Change existing API contracts

## Testing Notes

List specific areas requiring attention from QA or reviewers. Detail how to replicate the state needed to test the new code.

Example format:
1. Login as [role], navigate to [screen].
2. Trigger [action/error state].
3. Verify [expected outcome].
