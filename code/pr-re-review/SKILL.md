---
name: pr-re-review
description: Portable pull-request re-review workflow after updates, combining prior-comment verification with a full current-head review, non-redundant follow-up comments, review-task handling, and guarded approval. Use when an agent is asked to check a PR again, verify whether review comments were addressed, inspect the latest delta, decide what still blocks merge, approve an unblocked update, or produce an updated review after new commits on any GitHub, Bitbucket, GitLab, internal forge, or local repository workflow.
---

# PR Re-review

## Review Contract

A re-review is not only a closure checklist. It has two required passes:

1. Verify whether prior review comments were addressed.
2. Review the full current PR head against the target branch with the same critical lens as a fresh review.

Do not approve based on comment closure alone. The current head may still contain previously missed issues, new regressions, bad conflict resolutions, stale generated artifacts, dependency drift, or untouched changed files that deserve blockers or non-blocking findings.

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
   - Read the original review summary, open tasks, and any review-address task before adding new findings.

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
   - Re-check manual or live validation evidence when the updated PR changes a user-facing command, auth/setup flow, deployment path, data mutation, external integration, or other behavior automated tests cannot fully prove. Treat missing proof as blocking only when the latest behavior needs real-environment validation before merge.

5. Review the full current head before approving.
   - Inspect the full changed-file set, not only files touched after the last review.
   - Identify the PR's functional area and weight the full-head review toward the relevant risks: auth, permissions, CLI/API contract, install/release, UI, migration, workflow automation, data model, performance, docs-only, tests-only, or generated assets.
   - Re-check contracts, tests, docs, generated artifacts, lockfiles, build/package metadata, migrations, runtime/shipped paths, and ownership boundaries.
   - Look for previously missed issues as well as new issues introduced by the fix.
   - Pay extra attention when the PR is large, stacked, rebased, conflict-resolved, security-sensitive, dependency-heavy, or the earlier review had few or no findings.
   - Call out unnecessary complexity, redundancy, or over-engineering when it creates a correctness, maintainability, security, or operational risk.
   - For process, pipeline, release, permission, or cross-repo workflow changes, briefly fast-forward the post-merge behavior and check likely failure modes from real users, automation, reviewers, or branch protection. Keep this blast-radius based; do not require heavyweight redesign for small local changes.
   - If a finding is merge-blocking, include a brief severity rationale explaining why it must block merge rather than land as a follow-up. If that rationale is weak, downgrade it or ask a question.
   - Apply the same finding bar, output hygiene, manual-validation expectations, and repo-local approval policy as `pr-review`.

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

Previously missed findings
- [P2] ...

Validation
- ...
```

If posting back to a PR, keep the comment short and non-redundant. Reference the old thread or concern rather than repeating the full original review, but include any still-relevant issue found during the full current-head pass.

If all prior review comments look fixed, say that directly before listing any residual risk. If nothing remains, say there are no remaining blockers and no new non-blockers.

For posted follow-up reviews:

- Use exactly one main review summary unless the repo expects thread-by-thread replies.
- Use human-readable Markdown structure: short headings, concise bullets, and clear remaining/new finding sections.
- Do not repeat fixed issues except as a short status line.
- Keep review-address tasks open or resolve them according to the repo's policy.
- If there are no blockers, no major non-blockers, and the user asked for merge-readiness action, approve only when the local workflow, permissions, and repo-local approval policy make that appropriate.
- If you previously approved but the re-review finds blockers, withdraw or remove that approval when the forge supports it and the user expects active review-state management.
- If approval, task creation, or comment posting fails because of tooling or permissions, report that explicitly.

When posting a follow-up to a PR that already has a review-address task, reuse
that task instead of creating another. If the repo policy expects a task and
none exists, create exactly one equivalent task before approving or leaving the
PR blocked.

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
