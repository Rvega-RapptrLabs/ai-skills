# AI Skills, Agents & Rules

A central repository of **AI skills**, **agents**, **Cursor rules**, and related assets for the company. Use this repo to:

- **Reference** shared behaviors (e.g. how PRs are created, how Jira is used).
- **Install** skills, agents, and rules into your Cursor environment.
- **Improve** existing items via pull requests so everyone benefits.

Everything here is intended to work with **Cursor** (and compatible setups that use Cursor’s skills, agents, and rules).

---

## Repository structure

```
ai-skills/
├── README.md                 # This file
├── skills/                   # Cursor Agent Skills (SKILL.md + assets)
│   ├── pr-creator-agent/
│   │   └── templates/
│   │       ├── flutter.md
│   │       └── generic.md
│   ├── flutter-pr-review/    # Flutter PR review via gh / Jira MCP
│   └── release-preparer/     # Jira + GitHub release candidate branch
│       ├── SKILL.md
│       └── feedback.md
├── agents/                   # Cursor Agents (subagent / workflow definitions)
│   └── flutter-dev.md
└── rules/                    # Cursor Rules (safety & project conventions)
    ├── github-delete-protection.mdc
    └── jira-mcp-safety.mdc
```

---

## Quick setup (install into Cursor)

1. **Clone this repo** (or fork and clone):
   ```bash
   git clone <repo-url> && cd ai-skills
   ```

2. **Copy items into your Cursor config** (paths assume macOS/Linux; adjust for Windows):
   - **Skills** → `cp -r skills/pr-creator-agent ~/.cursor/skills/` and/or `cp -r skills/release-preparer ~/.cursor/skills/`
   - **Agents** → `cp agents/flutter-dev.md ~/.cursor/agents/`
   - **Rules** → `cp rules/*.mdc ~/.cursor/rules/` (or your project’s `.cursor/rules/`)

3. **Dependencies for flutter-dev**: The Flutter agent expects:
   - **pr-creator-agent** at `~/.cursor/skills/pr-creator-agent/` (for PR creation; branch naming rules are defined in the agent itself).
   - Jira MCP and GitHub CLI (`gh`) configured and authenticated.

---

## Skills

### pr-creator-agent

**Location:** `skills/pr-creator-agent/`

**What it does**

- Reads your **current branch** and **git diff** against a target branch (default `main`).
- Fetches the **Jira ticket** associated with the branch (e.g. `feature/RT-123` → `RT-123`) via Atlassian/Jira MCP.
- Picks a **PR template** (Flutter vs generic) based on the repo (e.g. `pubspec.yaml` → Flutter).
- Builds a **PR title** (`<TICKET-KEY> <purpose>`) and **description** from the template, then shows a **preview**.
- After you approve, runs `git push -u origin HEAD` and `gh pr create ... --assignee @me` in the **local terminal** (uses your `gh` auth; never creates PRs via GitHub MCP).

**When to use it**

- You say: “Create a PR”, “Open a pull request”, “Submit for review”, or similar.
- You want a consistent, Jira-linked PR with a structured description.

**How to use**

1. Install: copy `skills/pr-creator-agent/` to `~/.cursor/skills/pr-creator-agent/`.
2. Ensure Jira MCP is configured and `gh` is logged in (`gh auth login`).
3. Work on a branch whose name contains a Jira key (e.g. `fix/RT-456` or `feature/AUTH-405-oauth`).
4. In Cursor, ask to create a PR; the agent will gather diff + Jira, pick a template, show a preview, then create the PR after you confirm.

**Templates**

- `templates/flutter.md` – Flutter-focused (screens, state management, breaking changes, testing).
- `templates/generic.md` – Generic (overview, key changes, technical implementation, breaking changes, testing).

To add a new stack, add a new template in `templates/` and extend the “Select the template” logic in `SKILL.md`.

### release-preparer

**Location:** `skills/release-preparer/`

**What it does**

- Validates a **three-segment build number** (e.g. `2.0.17`) from the user prompt.
- Queries **Jira** (REFFER TEAM / `RT`) for issues where `"Target Release[Labels]"` equals that build number; lists ticket key and title.
- Auto-detects the **GitHub repo** from the workspace (`gh repo view`).
- Finds **merged PRs** on `development` that reference those ticket keys (PR search + optional commit scan).
- Maps PRs to tickets; flags gaps.
- Creates **`release-candidate/<build_number>`** from `main`, **cherry-picks** merge (or squash) commits, then **pushes** the branch.
- **Waits for confirmation after each phase.** Appends user ratings to `feedback.md` in the skill folder.

**When to use it**

- You say: “Create the 2.0.17 release”, “Prepare release 2.0.17”, or similar with an explicit `x.y.z` version.

**How to use**

1. Install: copy `skills/release-preparer/` to `~/.cursor/skills/release-preparer/`.
2. Open the **application repo** in Cursor (not only ai-skills), with `gh` and Git configured.
3. Ensure **Jira MCP** and **GitHub MCP** (or `gh` + API access) can read issues and PRs.
4. Invoke with a clear version string; confirm each step before the agent proceeds.

**Safety**

- No delete commands on GitHub or Jira; no force-push or destructive git. See `SKILL.md` for full rules.

### Other skills in `skills/`

| Folder | Purpose |
|--------|---------|
| `flutter-pr-review` | Review Flutter PRs (Clean Architecture, BLoC, SOLID, bugs, performance); optional Jira MCP; GitHub comments with required AI attribution line. |
| `release-preparer` | Prepare `release-candidate/x.y.z` from Jira target release + cherry-picks from `development`; confirmation gates; feedback log. |

---

## Agents

### flutter-dev

**Location:** `agents/flutter-dev.md`

**What it does**

- **Jira-first workflow**: Parses a Jira key from your message (e.g. `RT-1730`), fetches the ticket via **jira-mcp** (REFFER TEAM), then creates a branch (e.g. `fix/RT-1730` or `feature/RT-1730` from issue type).
- **Context before code**: Asks you which areas of the codebase to focus on before reviewing code (to reduce noise and hallucinations).
- **Expected result**: Asks for or confirms the expected outcome/acceptance criteria before planning.
- **Plan in a file**: Writes an implementation plan to `.cursor/plans/<TICKET>-implementation.plan.md` and **waits for your confirmation** before writing any app code.
- **Implementation**: Uses **FVM** (`fvm flutter` for run/test/pub get), follows Clean Architecture, BLoC, DioClient.
- **Testing**: Prompts you to test and/or runs `fvm flutter test` for affected code.
- **PR**: When you say you’re ready, follows the **pr-creator-agent** skill (preview, then `gh pr create --push` with assignee and default reviewers per that skill).

**When to use it**

- You say: “Review ticket X and work on a fix”, “Work on RT-XXXX”, or similar.
- You want a full flow: Jira → branch → context → plan → confirm → implement → test → PR.

**How to use**

1. Install: copy `agents/flutter-dev.md` to `~/.cursor/agents/flutter-dev.md`.
2. Install **pr-creator-agent** (see Quick setup).
3. Ensure Jira MCP (REFFER TEAM) and `gh` are set up.
4. In Cursor, invoke the agent with a ticket key (e.g. “Work on RT-1730”); follow the steps (context, expected result, plan approval, then implementation and PR).

**Safety (built-in)**

- No destructive Git (no force-push, branch delete, etc.).
- No Jira delete; only read (and optionally comment/worklog).
- All Flutter commands via `fvm flutter`.

---

## Rules

### github-delete-protection

**Location:** `rules/github-delete-protection.mdc`

**What it does**

- Prevents the AI from running **any** command or MCP action that **deletes** GitHub resources (branches, repos, tags, etc.).
- Disallows e.g. `git branch -d`, `git push --delete`, `gh repo delete`, and any MCP delete operations.
- If you ask to delete something, the AI will **refuse** and only give you the exact steps or commands to run yourself.

**When it applies**

- Marked `alwaysApply: true`, so it applies in every conversation where these rules are active (e.g. in `.cursor/rules/` or your user rules).

**How to use**

- Copy `rules/github-delete-protection.mdc` to `~/.cursor/rules/` (or your project’s `.cursor/rules/`).

---

### jira-mcp-safety

**Location:** `rules/jira-mcp-safety.mdc`

**What it does**

- Sets **REFFER TEAM** as the default Jira project when you say “my tickets”, “the board”, etc.
- **Delete protection**: The AI must never run a Jira MCP tool that deletes an issue.
- If you ask to delete a Jira issue, the AI will **not** do it until you **explicitly confirm** in a **new message** (e.g. “yes” or “confirm”).

**When it applies**

- Marked `alwaysApply: true` where this rule set is loaded.

**How to use**

- Copy `rules/jira-mcp-safety.mdc` to `~/.cursor/rules/` (or your project’s `.cursor/rules/`).

---

## Contributing and improving items

- **Suggest changes**: Open an issue or a pull request against this repo.
- **Edits**: Prefer updating the source files here (e.g. `SKILL.md`, `flutter-dev.md`, `.mdc` rules) so that improvements are versioned and shared.
- **New skills/agents/rules**: Add them under `skills/`, `agents/`, or `rules/` with a short description in this README and, if needed, in the “Quick setup” section.

---

## Dependencies overview

| Item              | Depends on                          |
|-------------------|-------------------------------------|
| pr-creator-agent  | Jira MCP, `gh` CLI, Git             |
| flutter-pr-review | `gh`, GitHub repo access; optional Jira MCP |
| release-preparer  | Jira MCP, GitHub MCP (or `gh`), Git; app repo workspace |
| flutter-dev       | Jira MCP, `gh`, branch-creator skill, pr-creator-agent skill, FVM |
| Rules             | None (optional: Jira MCP for Jira rule to be useful) |

---

## Summary

| Item                      | Purpose |
|---------------------------|--------|
| **pr-creator-agent**      | Create consistent, Jira-linked PRs via `gh pr create` and templates. |
| **flutter-pr-review**     | Review Flutter PRs with team standards and structured GitHub comments. |
| **release-preparer**      | Build release candidate branch from Jira target release + cherry-picks from `development`. |
| **flutter-dev**           | End-to-end Jira ticket flow: fetch ticket → branch → context → plan → implement → test → PR. |
| **github-delete-protection** | Block all GitHub/Git delete actions; only give instructions. |
| **jira-mcp-safety**       | REFFER TEAM default; no Jira issue delete without explicit confirmation. |

Clone, copy into Cursor, and extend this repo so the whole company can reference and improve each piece.
