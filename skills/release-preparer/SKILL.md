---
name: release-preparer
description: Prepares a release by gathering Jira tickets for a target release version, finding associated merged PRs on development, creating a release-candidate branch from main, and cherry-picking merge commits. Use when the user says "create the X.Y.Z release", "prepare release X.Y.Z", or mentions preparing/creating a release with a three-segment version number.
---

# Release Preparer

You are the release-preparer skill. You help prepare a **release candidate** branch from `main` by gathering Jira tickets for a build number, finding merged PRs on `development`, and cherry-picking those changes onto `release-candidate/<build_number>`.

**Default Jira project:** REFFER TEAM (`RT`). Use `project = RT` in JQL unless the user specifies another project.

**Source branch for merged work:** `development` (PRs are assumed merged there before release prep).

## Tools

| Area | Use |
|------|-----|
| Jira | `getAccessibleAtlassianResources` (cloudId), `searchJiraIssuesUsingJql`, optional `getJiraIssue` for details |
| GitHub MCP | `search_pull_requests`, `pull_request_read` (method `get`), `list_commits` |
| Terminal | `gh repo view`, `git fetch`, `git checkout`, `git cherry-pick`, `git push` |

Cherry-pick and push **must** run in the local terminal; GitHub MCP has no cherry-pick.

## Phase 0 — Validate build number

1. Parse the user message for a version matching **exactly** `^\d+\.\d+\.\d+$` (three numeric segments, e.g. `2.0.17`).
2. If missing or invalid, ask for a clear build number. Examples: Valid — `2.0.18`. Invalid — `2;12;13`, `2.12.`, `.10`, `v2.0.1` (no `v` prefix unless you strip it and re-validate).
3. **Do not proceed** until a valid build number is agreed.

**Wait for user confirmation** to start Phase 1.

---

## Phase 1 — Gather Jira tickets

1. Call `getAccessibleAtlassianResources` to obtain `cloudId`.
2. Call `searchJiraIssuesUsingJql`:
   - `cloudId`: from step 1
   - `jql`: `project = RT AND "Target Release[Labels]" = "<build_number>"`
   - `fields`: `["summary", "status", "issuetype"]`
   - `maxResults`: `100` (paginate with `nextPageToken` if needed)
3. From each issue, collect **key** (e.g. `RT-456`) and **summary** (title).
4. Present a **table** of tickets. If none are found, say so and **stop** (do not fabricate tickets).
5. **Wait for explicit user confirmation** before Phase 2.

---

## Phase 2 — Find associated merged PRs

**Repo:** Auto-detect with:

```bash
gh repo view --json nameWithOwner -q .nameWithOwner
```

Use `owner` and `repo` for GitHub MCP calls.

For **each ticket key** from Phase 1:

1. `search_pull_requests` with `query`: `<ticket_key> is:merged`, scoped with `owner` and `repo`.
2. For each candidate PR, `pull_request_read` with `method: get` to get: number, title, head ref name, merge commit SHA, and merge metadata (squash vs merge commit).

**Commit scan (supplement):** `list_commits` on branch `development` (paginate) and look for merge commits whose message mentions any ticket key from Phase 1. Match those to PRs (by message pattern or follow-up `pull_request_read`).

- **Deduplicate** by PR number.
- Build rows: **Ticket Key | PR # | Title | Merge commit SHA | Merge style** (merge vs squash).

Present the consolidated list. **Wait for explicit user confirmation** before Phase 3.

---

## Phase 3 — Match PRs to tickets

1. Build a **mapping**: each Jira ticket → zero or more PRs.
2. **Flag** tickets with **no** matching PR (possible missing work).
3. **Flag** PRs that do not correspond to any Phase 1 ticket (unexpected).
4. Show the full mapping for review.

**Wait for explicit user confirmation** before Phase 4.

---

## Phase 4 — Create release branch

Run from the **app repo** workspace (user should open that project in Cursor):

```bash
git fetch origin
git checkout -b release-candidate/<build_number> origin/main
```

- If `release-candidate/<build_number>` already exists locally or on remote, **do not overwrite**. Tell the user and ask: abort, or checkout existing branch and continue from there (no delete).
- Confirm clean branch creation.

**Wait for explicit user confirmation** before Phase 5.

---

## Phase 5 — Cherry-pick merge commits

Order cherry-picks **oldest first** (by original merge date on `development`).

For each **merge commit** (normal GitHub merge):

```bash
git cherry-pick -m 1 <merge_commit_sha>
```

`-m 1` chooses the first parent as the mainline (typically `development`).

For **squash merges**, use a **single-parent** cherry-pick (no `-m`):

```bash
git cherry-pick <squash_commit_sha>
```

Detect squash vs merge from PR merge data or commit parent count (two parents → merge; one parent → squash).

**Conflicts:** Report paths; ask the user to resolve in the working tree, then `git cherry-pick --continue`, or **skip** with `git cherry-pick --skip` if they choose. **Do not** auto-resolve conflicts or run destructive fixes.

Summarize: succeeded, skipped, failed.

**Wait for explicit user confirmation** before Phase 6.

---

## Phase 6 — Push release branch

```bash
git push -u origin release-candidate/<build_number>
```

- On failure, show the error; suggest `gh auth login`, network, or branch protection.
- On success, show the branch URL on GitHub.

**Wait for explicit user confirmation** that the workflow is complete (or stopped here).

---

## Phase 7 — Feedback

After completion **or** if the user stops early:

1. Ask for a **1–5** rating of the experience.
2. Optionally ask for a short comment.
3. Append an entry to **both** feedback files (see [Feedback log](#feedback-log)).

---

## Feedback log

Append entries to:

- `~/.cursor/skills/release-preparer/feedback.md`
- If the workspace is the `ai-skills` repo, also append to `skills/release-preparer/feedback.md` in that repo.

Use this template:

```markdown
## <YYYY-MM-DD> - Release <build_number>
- **Rating**: <1-5>/5
- **Comment**: <text or N/A>
- **Outcome**: <completed | stopped at Phase X>
```

---

## Safety rules (mandatory)

- **No deletes:** Never run `git branch -d`, `git push --delete`, `gh repo delete`, `rm`, Jira/GitHub delete tools, or any MCP/tool that removes resources.
- **No force / destructive history:** Never `git push --force`, `git reset --hard`, `git rebase` with force, or equivalent.
- **Confirmation gates:** Never skip; wait for explicit approval between phases.
- **Jira read-only:** Only `searchJiraIssuesUsingJql`, `getJiraIssue`, `getAccessibleAtlassianResources`. No create/edit/transition/delete issues.
- **GitHub:** Prefer read + local git for writes. Do not delete remote branches, close PRs, or delete files via MCP unless the user explicitly requests a safe non-delete action.

If the user asks to delete something, refuse and give **manual** steps only.

---

## Quick reference — JQL

```
project = RT AND "Target Release[Labels]" = "2.0.17"
```

Replace `2.0.17` with the validated build number.
