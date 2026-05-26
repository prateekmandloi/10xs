---
name: pr-review
description: Portable pull-request review workflow with live repository and PR context. Use when an agent is asked to review a PR, inspect a change for blockers or risks, prepare review feedback, post a review comment, or produce a non-redundant action-oriented code review for any GitHub, Bitbucket, GitLab, internal forge, or local repository workflow.
---

# PR Review

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
   - Fetch live PR metadata: title, description, author, source branch, target branch, current head SHA, checks, approvals, changed files, diff, and existing comments.
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
   - Prioritize correctness, data loss, security, auth, permissions, routing, API contracts, migrations, concurrency, reliability, observability, backward compatibility, generated artifacts, and test coverage.
   - Avoid style, naming, readability, and cosmetic feedback unless it creates a real maintenance or behavior risk.
   - Call out unnecessary complexity, redundancy, or over-engineering when it creates a correctness, maintainability, security, or operational risk.
   - Treat missing tests as important when they protect a changed contract, bug fix, authorization boundary, migration, or user-facing flow.
   - For agent-facing tools, CLIs, APIs, or generated instructions, prefer machine-safe contracts over human-friendly interaction patterns when they conflict.
   - If documentation, prompt, or skill/reference files changed, consider size or token growth against the base and flag unjustified growth when the repo treats prompt budget as a constraint.

5. Verify findings before presenting them.
   - Open the exact file and line range.
   - Trace the call path or data contract far enough to prove the behavior.
   - Run focused tests or static checks when practical. If full validation is too expensive or blocked, say what was and was not verified.
   - Before saying there are no findings or approving, scan the full changed-file set for contracts, tests, docs, generated artifacts, lockfiles, build/package metadata, runtime/shipped paths, and ownership boundaries.

## Finding Standard

Only report a finding when it is actionable, grounded in evidence, and likely to matter.

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
I found one blocker and one non-blocking follow-up.

Blocker:
- ...

Non-blocker:
- ...

Validation: ...
```

If posting a formal review in a repo with review tasks or approval state:

- Use exactly one main review summary unless inline comments are requested.
- Separate blockers and non-blockers clearly.
- Do not use checkboxes in the posted review unless the repo expects them.
- Do not mention local paths, temp paths, hidden artifacts, investigation mechanics, or private workspace details.
- If there are blockers, do not approve.
- If there are no blockers and the user asked for a merge-readiness action, approve only when the local workflow and permissions make that appropriate.
- If approval, task creation, or comment posting fails because of tooling or permissions, report that explicitly.

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
