# Generic PR Description Template

Write a precise, concise, and highly technical Pull Request description. Be direct and focus on technical facts — no conversational filler.

---

## Overview

Write 2-3 sentences explaining the primary goal of this PR. What business or technical requirement does this solve?

## Key Changes

Highlight the core logical changes, service integrations, or architectural updates.

* `[Feature/Module Name]`
  * Explain what the change does in a single sentence.
  * Mention any modified API contracts, data models, or configuration changes.

## Technical Implementation

List the most important new or significantly modified files with a concise 1-line description.

* `path/to/file.ext` — Brief explanation of what this file handles in this PR.

## Breaking Changes

> This section is required. If there are no breaking changes, write "None".

List any changes that:
- Break existing behavior or public APIs
- Require dependency updates or environment changes
- Change existing data schemas or contracts

## Testing Notes

List specific areas requiring attention from QA or reviewers. Detail how to replicate the state needed to test the changes.

Example format:
1. Set up [precondition].
2. Execute [action].
3. Verify [expected outcome].
