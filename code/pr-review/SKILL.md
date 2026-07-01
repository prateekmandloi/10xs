---
name: pr-review
description: Portable first-pass pull-request review workflow with live repository and PR context, current-head analysis, comment dedupe, behavioral risk focus, review-task handling, one-summary-comment posting, and guarded approval. Use when an agent is asked to review a fresh PR or branch diff, inspect a change for blockers or risks, prepare or post review feedback, create review-address tasks, approve an unblocked PR, or produce a non-redundant action-oriented code review for any GitHub, Bitbucket, GitLab, internal forge, or local repository workflow; use pr-re-review instead when prior review comments were addressed and the user asks for a follow-up.
---

# PR Review

## Overview

Use this workflow for fresh, first-pass PR reviews that have not yet gone
through this review workflow. Resolve the current PR head, understand the
changed contract before judging it, focus on actionable behavioral risk, and
keep repo-facing output crisp.

## Portability Contract

Treat this skill as a workflow template, not a repo-specific playbook.

- Discover the forge, default branch, review tool, test commands, ownership model, and comment style from the current repo before acting.
- Prefer repo-local conventions over examples in this file.
- Keep all organization names, handles, teams, labels, branch names, and command aliases as runtime inputs, not skill constants.
- If moving this skill to a new repo, only personalize the knobs section or agent metadata; the review workflow should still apply unchanged.
- If a workflow has no PR concept, map "PR" to the nearest review unit: merge request, change list, patch set, diff, stack, or branch.

## Core Workflow

1. Resolve the PR source of truth before inspecting code.
   - Identify the forge and repo from the URL, branch, local remote, issue text, or available CLI.
   - Fetch live PR metadata: title, description, author, source branch, target branch, current head SHA, checks, approvals, changed files, diff, tasks, linked issues, and existing comments.
   - If live fetching is unavailable, state the limitation and use local commits or provided diff only.

2. Read existing comments before forming findings.
   - Deduplicate against reviewer comments, bot comments, unresolved threads, and author replies.
   - Prefer extending or confirming an existing unresolved concern over restating it as new.
   - If asked to post, default to one concise main-thread comment unless inline comments are explicitly requested.
   - If the review system uses tasks, check for an existing equivalent review-address task before creating another.

3. Establish the baseline.
   - Compare the PR against its target branch, not against a stale local branch.
   - Pull or fetch the target ref when safe.
   - Check whether a suspected problem already exists on the target branch. If it does, frame it as an existing codebase gap unless the PR worsens it.

4. Review for behavioral risk first.
   - Understand public contracts, command or API grammar, constraints, and current architecture before judging changes to user-facing surfaces.
   - Read repo policy files when the PR touches workflow, tests, auth, install/release, docs, generated instructions, or command/API surfaces.
   - Identify the PR's functional area before judging it: auth, permissions, CLI/API contract, install/release, UI, migration, workflow automation, data model, performance, docs-only, tests-only, or generated assets. Weight the review lens toward that area instead of applying every checklist equally.
   - Prioritize correctness, data loss, security, auth, permissions, routing, API contracts, migrations, concurrency, reliability, observability, backward compatibility, generated artifacts, and test coverage.
   - Avoid style, naming, readability, and cosmetic feedback unless it creates a real maintenance or behavior risk.
   - Call out unnecessary complexity, redundancy, or over-engineering when it creates a correctness, maintainability, security, or operational risk.
   - Treat missing tests as important when they protect a changed contract, bug fix, authorization boundary, migration, or user-facing flow.
   - For agent-facing tools, CLIs, APIs, or generated instructions, prefer machine-safe contracts over human-friendly interaction patterns when they conflict.
   - If documentation, prompt, or skill/reference files changed, consider size or token growth against the base and flag unjustified growth when the repo treats prompt budget as a constraint.
   - For process, pipeline, release, permission, or cross-repo workflow changes, briefly fast-forward the post-merge behavior and ask what could fail once real users, reviewers, automation, or branch protection interact with it. Use this only when the blast radius justifies it; do not turn small local changes into over-engineering exercises.
   - Classify each concern as a blocker, non-blocker, or repo/codebase gap. Do not call something a PR blocker when the target branch already has the same unresolved behavior.

5. Verify findings before presenting them.
   - Open the exact file and line range.
   - Trace the call path or data contract far enough to prove the behavior.
   - Run focused tests or static checks when practical. If full validation is too expensive or blocked, say what was and was not verified.
   - Check for manual or live validation evidence when the PR changes a user-facing command, auth/setup flow, deployment path, data mutation, external integration, or other behavior automated tests cannot fully prove. Missing proof is a blocker only when the changed behavior needs real-environment validation before merge; for test-only, docs-only, or low-risk internal refactors, do not manufacture a live-test task.
   - Before saying there are no findings or approving, scan the full changed-file set for contracts, tests, docs, generated artifacts, lockfiles, build/package metadata, runtime/shipped paths, and ownership boundaries.

## Finding Standard

Only report a finding when it is actionable, grounded in evidence, and likely to matter.

## Functional Area Lenses

Use these lenses selectively after identifying the PR's functional area. They are prompts for deeper review, not a mandatory checklist for every PR.

- **Contract surfaces:** Check CLI/API/UI grammar, schemas, help text, docs, generated catalogs, compatibility aliases, output envelopes, error semantics, and machine-readable fields against actual runtime behavior.
- **Runtime boundaries:** For browser, embedded, worker, serverless, local CLI, or SDK hosts, verify environment, fetch/network, storage, auth, cancellation, timeout, and filesystem assumptions. Watch process-global patches, shared mutable state, and concurrent execution.
- **Auth, security, and permissions:** Treat login/logout/revoke, token refresh, delegated auth, scope or permission manifests, secret storage, redaction, outbound headers, setup/install auth, and destructive-action guards as high-risk.
- **Data and content mutation:** Check canonical versus legacy paths, serialization formats, version/snapshot/ETag handling, optimistic concurrency, idempotency, rollback, partial writes, binary/media handling, and user-visible format preservation.
- **Search, ranking, and discovery:** Verify pagination, truncation, partial failures, connector/provider state, stale caches, confidence or evidence fields, ranking/filter semantics, and warnings that both humans and machines need.
- **Release, install, packaging, and CI:** Check package-manager assumptions, workspace filters, lockfiles, generated artifacts, publish order, installer/uninstaller paths, changelog or docs mirrors, branch protection, and pipeline behavior after merge.
- **Docs-only or tests-only changes:** Confirm they match the actual runtime contract, but do not invent live-validation work unless the PR changes behavior that automated tests cannot prove.

Each finding should include:

- Severity marker such as `[P0]`, `[P1]`, `[P2]`, or `[P3]`.
- File and line reference when available.
- The failing scenario or contract.
- Why the current change creates or exposes the risk.
- The smallest useful fix direction.
- For merge-blocking findings, a brief severity rationale explaining why it should block merge rather than land as a follow-up.

Use this severity guide:

- `P0`: blocks release, causes data loss, broad security failure, or total outage.
- `P1`: likely user-visible breakage, auth/privacy issue, major regression, or migration hazard.
- `P2`: real correctness, reliability, or maintainability issue that should be fixed before merge if feasible.
- `P3`: low-risk follow-up, hardening, or test improvement.

## Output Shape

Lead with findings, ordered by severity. Keep summaries secondary.

When there are findings:

```markdown
Findings
- [P1] Short title - path/to/file.ext:123
  Explain the concrete failure mode and why it matters. Mention the likely fix.

Open questions
- Any unresolved assumptions that affect review confidence.

Validation
- Commands or checks run, plus anything blocked.
```

When there are no findings, say that clearly and mention residual risk or tests not run.

For a posted PR comment, compress to a single review-ready comment:

```markdown
Code review summary

I found one blocker and one non-blocking follow-up.

Blocker:
- ...

Non-blocker:
- ...

Validation: ...
```

If posting a formal review in a repo with review tasks or approval state:

- Use exactly one main review summary unless inline comments are requested.
- Use human-readable Markdown structure: short headings, concise bullets, and clear blocker/non-blocker sections.
- Separate blockers and non-blockers clearly.
- Do not use checkboxes in the posted review unless the repo expects them.
- Do not mention local paths, temp paths, hidden artifacts, investigation mechanics, or private workspace details.
- If there are blockers, do not approve.
- If there are no blockers and the user asked for a merge-readiness action, approve only when the local workflow, permissions, and repo-local approval policy make that appropriate.
- If you already approved and later find blockers, withdraw or remove that approval when the forge supports it and the user expects active review-state management.
- If approval, task creation, or comment posting fails because of tooling or permissions, report that explicitly.

## PR Comment And Task Workflow

When the user asks to post the review or take review action:

- Post exactly one main PR comment. Use the repo's expected title or default to `Code review summary`.
- Include only net-new findings still relevant on the current head.
- Separate blockers and non-blockers clearly.
- Do not post approval-state words such as `ACCEPTED` or `REJECTED` unless the forge requires them.
- Do not use checkboxes unless the repo explicitly expects them.
- Do not mention reviewer names, personas, hidden process, or investigation mechanics.
- Ensure there is exactly one open review-address task when the repo supports tasks and the local policy expects one. Treat the task text as a repo-local runtime input; default to `Address review comments before merge.` only when no better local convention exists.
- If an equivalent open task already exists, do not create another.
- If there are blockers, do not approve and keep the review-address task open.
- If there are no blockers and no major non-blockers, approve only when the repo-local approval policy permits it; keep or resolve the review-address task according to repo policy.
- If approval, task creation, or comment posting fails because of tooling or permissions, report that explicitly in the final user update.

## Output Hygiene

Use repo-relative paths in review text. Never include local paths, absolute
paths, temp paths, sandbox paths, machine-specific paths, session IDs,
stdout/stderr temp file locations, local worktree references, or hidden
internal artifact names.

Before posting, remove anything that is not useful to a repo reviewer.

## Final User Update

After posting or approving, include the PR comment link when available, whether
the PR was approved or not approved, whether the review-address task was
created or already existed, and a brief blocker status.

## Editing During Reviews

Default to review-only behavior. Do not modify the PR unless the user asks for
fixes.

## Personalization Knobs

Adapt these to the user or repo when known:

- Preferred review lens: architecture, security, product behavior, test gaps, migration safety, performance, or release risk.
- Preferred output: local review only, main-thread comment, inline comments, approval summary, or blocker-only list.
- Relevant identities: user handles, team handles, codeowner groups, bot accounts, and reviewers whose comments should be treated as already covered.
- Repo conventions: required checks, generated files, lockfiles, migration workflow, test commands, monorepo ownership boundaries, and branch naming.
- Tool preference: `gh`, `glab`, Bitbucket CLI/API, internal CLI, or local git only.
- Review task behavior: create one review-address task, reuse an existing one, or avoid tasks entirely.
- Approval behavior: never approve, approve when unblocked, or only summarize.

Keep personalization small and local. Do not fork the whole workflow unless the repo has a genuinely different review model.

## Practical Commands

These are examples, not requirements. Discover tools from the repo before assuming one:

```bash
git remote -v
git branch --show-current
git status --short --branch
```

Common live-state commands, when available:

```bash
gh pr view <pr> --json title,body,author,headRefName,baseRefName,headRefOid,reviewDecision,comments,reviews,files,statusCheckRollup
gh pr diff <pr>
glab mr view <mr>
glab mr diff <mr>
```

For Bitbucket or internal workflows, use the repo's documented CLI or API. First ask the tool for help when the command surface is uncertain.
