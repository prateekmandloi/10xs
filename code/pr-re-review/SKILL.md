---
name: pr-re-review
description: Portable pull-request re-review workflow after updates. Use when an agent is asked to check a PR again, verify whether review comments were addressed, inspect only the latest delta, decide what still blocks merge, or produce an updated review after new commits on any GitHub, Bitbucket, GitLab, internal forge, or local repository workflow.
---

# PR Re-review

## Portability Contract

Treat this skill as a workflow template, not a repo-specific playbook.

- Discover the forge, previous review source, thread model, default branch, and validation commands from the current repo or PR.
- Keep old head SHAs, reviewer handles, team names, labels, and check-suite names as runtime inputs.
- Prefer the repo's native review-thread semantics: GitHub review threads, GitLab discussions, Bitbucket comments, Gerrit patch sets, or internal equivalents.
- If moving this skill to a new repo, personalize only the knobs section or agent metadata unless the repo has a materially different review model.
- If a workflow has no PR concept, map "PR" to the nearest review unit: merge request, change list, patch set, diff, stack, or branch.

## Core Workflow

1. Anchor the re-review to the previous state.
   - Fetch live PR metadata, current head SHA, target branch, checks, approvals, existing review threads, and the previous review point when available.
   - Identify the old head SHA from prior comments, review events, merge-base history, local notes, or the user's provided context.
   - If no prior head is recoverable, say so and review the full PR with extra attention to already-discussed threads.

2. Build a delta-first view.
   - Compare old head to new head for changed commits and files.
   - Separately inspect the full PR diff against target branch for regressions that the delta may hide.
   - Read author replies and resolved/unresolved threads before deciding whether something remains open.
   - Read the original review summary and any review-address task before adding new findings.

3. Classify each prior concern.
   - `Resolved`: the code or tests now address the issue.
   - `Partially resolved`: the main path is fixed but an edge case, contract, or test remains.
   - `Still blocking`: the original failure mode still exists.
   - `Superseded`: the code changed enough that the old concern no longer applies, but a new risk may have appeared.
   - `Cannot verify`: required environment, artifact, permission, or command is unavailable.

4. Review new risk introduced by the update.
   - Focus on the new commits, conflict resolutions, generated files, dependency updates, migrations, and test changes.
   - Watch for fixes that only satisfy the visible comment while leaving the underlying contract broken.
   - Re-run focused validation that maps to the changed behavior.
   - Do not call something a blocker if the target branch already has the same unresolved behavior; classify it as a repo/codebase gap unless the PR worsens it.
   - If documentation, prompt, or skill/reference files changed, consider size or token growth against the base when the repo treats prompt budget as a constraint.

## Output Shape

Prefer a concise delta report:

```markdown
Re-review result: still blocked / unblocked with follow-ups / no remaining issues.

Resolved
- ...

Still blocking
- [P1] ...

New findings
- [P2] ...

Validation
- ...
```

If posting back to a PR, keep the comment short and non-redundant. Reference the old thread or concern rather than repeating the full original review.

If all prior review comments look fixed, say that directly before listing any residual risk. If nothing remains, say there are no remaining blockers and no new non-blockers.

For posted follow-up reviews:

- Use exactly one main review summary unless the repo expects thread-by-thread replies.
- Do not repeat fixed issues except as a short status line.
- Keep review-address tasks open or resolve them according to the repo's policy.
- If there are no blockers and the user asked for merge-readiness action, approve only when the local workflow and permissions make that appropriate.
- If approval, task creation, or comment posting fails because of tooling or permissions, report that explicitly.

## Personalization Knobs

Adapt these to the user or repo when known:

- Whether to verify every old comment or only blocker-level comments.
- Whether to post a final "looks addressed" comment, leave local notes only, or update specific threads.
- Which reviewers, bots, or check suites are authoritative.
- How much validation is expected before saying a PR is unblocked.
- Preferred phrasing for remaining blockers versus non-blocking follow-ups.
- Review task behavior after re-review: keep open, resolve, or avoid tasks.
- Approval behavior after fixes: never approve, approve when unblocked, or only summarize.

Keep personalization small and local. The re-review loop should remain delta-first across repos.

## Useful Checks

These are examples, not requirements. Start with safe local and live state:

```bash
git status --short --branch
git fetch --all --prune
git merge-base <target> <head>
git diff <old-head>..<new-head>
git diff <target>...<head>
```

Use forge tools when available:

```bash
gh pr view <pr> --json headRefOid,baseRefName,comments,reviews,reviewThreads,statusCheckRollup
gh pr diff <pr>
glab mr view <mr>
glab mr diff <mr>
```

If the repo uses Bitbucket or an internal CLI, inspect that CLI's help and fetch comments plus full PR details before re-reviewing.
