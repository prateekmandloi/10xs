---
name: pr-refresh
description: Portable pull-request refresh workflow with reviewer-feedback triage, safe branch update, validation, review-task state handling, and concise PR summary posting. Use when an agent is asked to check whether a PR is still relevant, address reviewer feedback, reply to review comments, update it from the target branch, rebase or merge latest main, resolve conflicts, regenerate artifacts, run validation, push with lease, post an addressed-comments summary, or close out an old branch without losing local work.
---

# PR Refresh

## Portability Contract

Treat this skill as a workflow template, not a repo-specific playbook.

- Discover the forge, branch model, target branch, update strategy, generated-file workflow, validation commands, and push policy from the current repo.
- Keep branch names, backup naming, remote names, CI names, issue IDs, review-task text, comment style, and release labels as runtime inputs.
- Prefer repo policy over this file when choosing rebase, merge, stack update, branch recreation, or close/drop.
- If moving this skill to a new repo, personalize only the knobs section or agent metadata unless the repo has a materially different branch-refresh model.
- If a workflow has no PR concept, map "PR" to the nearest review unit: merge request, change list, patch set, diff, stack, or branch.

## Core Workflow

1. Confirm the PR is still worth refreshing.
   - Fetch live PR state: title, branch, target branch, head SHA, checks, comments, age, merge conflicts, and linked issue or report.
   - Check whether the target branch already contains the fix or feature.
   - If the PR is obsolete, say why and recommend closing or replacing it instead of forcing an empty refresh.
   - If the refresh is driven by reviewer feedback, fetch comments, review threads, tasks, author replies, CI, and latest commits before editing.

2. Protect local and remote state.
   - Inspect `git status --short --branch` before mutating anything.
   - Do not discard unrelated local changes. Stash or stop with a clear blocker summary when dirty state cannot be safely separated.
   - Create a backup branch before complex rebases or conflict-heavy refreshes.
   - Record the original head SHA and remote tracking branch.
   - Never mix fixes for different PRs in one branch or commit unless the repo explicitly uses stacked PRs.

3. Triage reviewer feedback when present.
   - Build a private checklist of every unresolved comment, task, failed check, review-address task, and reviewer request.
   - Deduplicate repeated feedback.
   - Classify each item as `fix`, `pushback`, `question`, or `defer`.
   - For `fix`, make the smallest correct code, test, or doc change.
   - For `pushback`, reply concisely with evidence when the comment is incorrect, stale, already handled, or conflicts with the intended architecture.
   - For `question`, ask a focused follow-up and do not guess.
   - For `defer`, explain why the item belongs outside this PR and point to the follow-up path when one exists.
   - Identify the feedback's functional area before editing so the fix preserves the right contract: contract surface, runtime boundary, auth/security/permission, data/content mutation, search/discovery/ranking, release/install/packaging/CI, docs-only, or tests-only.

4. Refresh from the live target branch.
   - Fetch the target branch and PR branch.
   - Prefer the repo's expected update method: rebase, merge, stack rebase, or branch recreation.
   - For stacked PRs, refresh from bottom to top and preserve the intended base of each branch.
   - Resolve conflicts by preserving the PR intent and current target-branch contracts.

5. Regenerate and validate.
   - Run repo-specific generation steps before typecheck/tests when generated files are part of the build.
   - Run focused tests for the changed behavior plus cheap required checks.
   - If broad tests fail due to unrelated target-branch drift, distinguish that from refresh failure.
   - Broaden validation when the feedback touches shared contracts, CLI/API grammar, auth/tokens/scopes, output shape, destructive behavior, generated instructions, or agent-mode behavior.
   - For runtime-boundary fixes, include validation for the relevant host assumptions: environment, network/fetch, storage, auth, cancellation/timeouts, filesystem, global state, and concurrency.
   - For release/install/package fixes, check the actual package-manager, workspace, lockfile, generated-artifact, publish, installer, docs/changelog, and CI path that would run after merge.
   - For docs-only or tests-only feedback, verify the artifact now matches the runtime contract; do not add unrelated live-validation burden unless behavior changed.

6. Commit, push, and update review state.
   - Follow repo commit rules. Stage only files for the current PR.
   - Add changelog, changeset, release note, or session log artifacts only when the repo expects them.
   - Use `--force-with-lease` for rebased branches.
   - Verify the PR now points at the pushed head and checks have started or passed when available.
   - Post one concise summary comment when the workflow expects it: addressed items, pushed-back items, questions, validation, commit hash, and remaining blockers.
   - Reuse the existing review-address task instead of creating duplicates.
   - Resolve or leave review-address tasks according to repo policy only after each tracked item is handled by a fix, pushback reply, clarifying question, or explicit deferral.
   - Summarize relevance, refresh method, comments handled, conflicts, validation, pushed state, and remaining risks.

## Stop Conditions

Stop and ask or report a blocker when:

- The working tree has unrelated changes that cannot be safely stashed or isolated.
- The PR branch cannot be identified with confidence.
- The target branch or PR branch fetch fails.
- The PR appears obsolete and the user did not explicitly ask for a replacement hardening diff.
- Conflict resolution would require product judgment not inferable from code or comments.
- Pushing would overwrite remote work not included in the local branch.
- A reviewer request is ambiguous and multiple materially different fixes are plausible.
- Validation fails in a way that affects the reviewed behavior.

## Personalization Knobs

Adapt these to the user or repo when known:

- Default target branch names and protected branch rules.
- Rebase versus merge preference.
- Whether stale PRs should be closed, refreshed, or converted into regression hardening.
- Required generation commands, smoke tests, and CI checks.
- Whether to post a PR comment after refresh.
- Preferred backup branch naming.
- Review item outcomes and task handling: fix, pushback, question, defer, resolve, or leave open.
- Required local artifacts before commit: changelog, changeset, session log, generated docs, or none.
- Whether the workflow allows code changes during refresh or only asks for a plan.

Keep personalization small and local. The safety rules should travel unchanged across repos.

## Practical Command Pattern

These are examples, not requirements. Use the exact forge and repo commands when known. A safe generic shape is:

```bash
git status --short --branch
git remote -v
git fetch --all --prune
git branch --show-current
git rev-parse HEAD
git merge-base origin/<target> HEAD
```

Before a risky rebase:

```bash
git branch backup/<pr-or-branch>-before-refresh
git rebase origin/<target>
```

For headless environments where rebase continuation opens an editor:

```bash
GIT_EDITOR=true git rebase --continue
```

After successful validation:

```bash
git push --force-with-lease
```

Never run destructive cleanup such as hard reset, branch deletion, or stash dropping unless the user explicitly asks or it is obviously limited to temporary artifacts you created.

## Summary Comment

When posting back to the PR, keep it repo-facing and concise:

```markdown
Review comments addressed.

- Addressed: <short list>
- Clarified/pushed back: <short list, if any>
- Questions: <short list, if any>
- Validation: <commands or CI signal>
- Commit: <hash>
```

Do not include local paths, temp paths, hidden artifact names, session IDs, or private investigation mechanics.
