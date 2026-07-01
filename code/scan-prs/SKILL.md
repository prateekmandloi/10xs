---
name: scan-prs
description: Portable pull-request action scanning workflow that routes PRs into fresh review, re-review, refresh, response, merge-readiness, task follow-up, or escalation. Use when an agent is asked to find PRs needing review, re-review, refresh, response, merge, approval, follow-up, or escalation across a repo, organization, workspace, team, internal forge, or local checkout, and to prioritize the user's next actions.
---

# Scan PRs

## Portability Contract

Treat this skill as a workflow template, not a repo-specific dashboard.

- Discover the forge, scope, identity model, review-request model, team aliases, stale thresholds, and available APIs from the current workspace.
- Keep user handles, teams, repos, labels, milestones, projects, and query syntax as runtime inputs.
- Prefer the repo or organization's native activity model: PRs, merge requests, patch sets, changes, branches, review queues, or internal equivalents.
- If moving this skill to a new repo, personalize only the knobs section or agent metadata unless the workspace has a materially different action model.
- If comments or review-request data are unavailable, degrade gracefully and label the scan source clearly.

## Core Workflow

1. Resolve scope and identity.
   - Determine the repo, workspace, organization, team, or local checkout to scan.
   - Determine the user's relevant handles, team names, codeowner groups, and bot accounts.
   - If identity is unknown, infer from git config, forge auth, repo ownership, recent commits, or ask only when action classification depends on it.

2. Fetch live PR lists.
   - Include PRs authored by the user.
   - Include PRs requesting the user's review or their team's review.
   - Include PRs where the user was mentioned, commented, requested changes, approved, or has unresolved threads.
   - Include stale open PRs owned by the user that may need refresh or closure.
   - Include blocked PRs where the user is likely the next responder.

3. Classify each PR into action buckets.
   - `Review`: user or team is requested, no response yet, and the PR has not yet gone through the fresh `pr-review` workflow.
   - `Re-review`: user already reviewed or commented, author pushed updates, and unresolved risk may remain.
   - `Respond`: someone asked the user a question, requested changes from the user, or CI/reviewer feedback needs owner action.
   - `Refresh`: user's PR is stale, conflicted, failing due to drift, or behind target branch.
   - `Approve/merge-readiness`: PR appears reviewed, green, unblocked, and needs approval or final merge-readiness action.
   - `Merge/land`: user's PR is approved and checks are green, but still open.
   - `Close/drop`: PR appears obsolete, superseded, or already landed elsewhere.
   - `Watch`: relevant but no immediate user action.

4. Prioritize.
   - Rank blockers, requested reviews, failing owner PRs, merge-ready PRs, and stale high-value PRs above passive watch items.
   - Highlight age, last activity, failing checks, unresolved comments, review state, and target branch drift.
   - Avoid noisy lists. Prefer the smallest set of PRs with a concrete next action.

5. Optionally hand off to action skills.
   - Use `pr-review` for a selected fresh review, including posting one summary comment, task creation, and guarded approval when requested.
   - Use `pr-re-review` for an updated PR after prior feedback.
   - Use `pr-refresh` for a stale or conflicted user-owned PR.

## Output Shape

Use an action table by default:

```markdown
| Priority | PR | Action | Why now | Suggested next step |
| --- | --- | --- | --- | --- |
| P1 | #123 Title | Re-review | New commits after requested changes | Check auth path fix |
```

Then include a short bucket summary:

```markdown
Review: 3
Re-review: 2
Respond: 1
Refresh: 2
Merge/land: 1
Watch: 4
```

When the scan source is incomplete, include a clear caveat such as "local git only", "comments unavailable", or "team review requests unavailable".

## Personalization Knobs

Adapt these to the user or repo when known:

- User handles across forges.
- Team review groups and CODEOWNERS aliases.
- Repos, projects, labels, milestones, or components that matter most.
- Definition of stale, such as 7 days without activity or behind target branch.
- Preferred priority ordering: unblock others first, land owned PRs first, review requests first, or release blockers first.
- Whether to include watch-only PRs.

Keep personalization small and local. The action buckets should travel unchanged across repos.

## Practical Commands

These are examples, not requirements. Discover the forge first:

```bash
git remote -v
git config user.email
git config user.name
```

GitHub examples:

```bash
gh pr list --search "review-requested:@me state:open" --json number,title,url,author,updatedAt,reviewDecision,statusCheckRollup
gh pr list --author "@me" --state open --json number,title,url,updatedAt,reviewDecision,statusCheckRollup
gh search prs --review-requested=@me --state=open
```

GitLab examples:

```bash
glab mr list --reviewer @me --state opened
glab mr list --assignee @me --state opened
```

For Bitbucket or internal workflows, use the repo's documented CLI/API and fetch comments or activities when possible. If the command surface is uncertain, inspect help before scanning broadly.
