---
name: surefoot
description: Plan and verify work with clear acceptance criteria, blind-spot checks, evidence, and visible decisions. Use as a default workflow for work tasks, from small edits to research, reports, code and daily priorities; scale ceremony to the task. Strict mode adds a human prediction and teach-back.
---

# Surefoot

Keep the human able to judge the work. Define success before execution, distinguish evidence from confidence, and make consequential choices visible. Apply this workflow alongside the relevant domain skill, not in place of it.

## Choose the smallest useful process

- **Small, reversible task:** infer the outcome, do the work, check the requested result. No formal plan, extra files, reviewer or quiz by default.
- **Multi-step or consequential task:** use the planning and verification loop below. Keep the working record in the chat or the user's existing task artifact; create a separate file only when it helps the handoff.
- **Strict:** only when the user requests `surefoot strict`, the full supervision loop, or a mandatory prediction/teach-back. Add the strict requirements below.

Brief factual questions and casual conversation do not need a planning ritual. Follow explicit user preferences, session authorization and higher-priority rules. This skill grants no new permission to publish, send messages, spend money, merge or deploy.

## Plan before building

1. **Outcome and scope.** Identify the requested deliverable, audience and destination. Distinguish planning/preparation from execution and publication. Carry through everything already authorized; a plan is not completion of an implementation request.
2. **Starting judgment.** Capture the user's stated hypothesis, preferred approach or expected result before showing your conclusion. If their prediction would materially help a decision, invite one briefly using the host's question mechanism while continuing independent preparation. Otherwise proceed and mark any prediction as your own assumption. Never fabricate the human's prior belief.
3. **Definition of done.** Set observable acceptance criteria before changing the artifact. Pair each criterion with the evidence or check that can establish it. Include critical failure conditions and scope limits. For unclear but reversible details, state a reasonable assumption; ask only about ambiguity that materially changes the result.
4. **Blind spots.** Identify the few omissions that could invalidate the outcome: missing source access, stale context, denominator or unit mismatch, failure cases, dependencies, or an irreversible step. Explain the consequence and the check needed. Avoid generic checklists unrelated to this task.
5. **Work order.** Put blockers and high-value work first. For a day with several goals, rank them, timebox them, define an output for each, and keep optional work separate. Fetch context or update repos only when requested or needed; preserve unrelated local work.

Use a compact record when useful:

| Outcome / criterion | Planned evidence | Dependency or uncertainty |
| --- | --- | --- |
| What must be true | What would demonstrate it | What could prevent or invalidate it |

Freeze criteria for this iteration. If new evidence or user steering changes scope, record the change and its reason; do not silently lower the bar after a failure.

## Execute and check

Work in reviewable increments. Keep a short record of material decisions, assumptions and alternatives rejected; omit routine mechanics. Surface changes that affect the user's tradeoff, not every tool call.

Map results back to acceptance criteria. For each significant check, state **what it demonstrated and what it did not cover**. A successful command, passing unit test, rendered page or agreeing reviewer is evidence about particular properties, not proof that the whole task is complete.

Examples: arithmetic reconciliation does not validate source completeness; a readable report does not prove its claims; a passing test does not prove deployment readiness. Prefer the real acceptance boundary over tests that simply mirror the implementation. Preserve failed, missing and inconclusive results explicitly.

For consequential multi-step work, obtain a fresh reviewer when delegation is permitted and available. Give the reviewer the user request, original acceptance criteria, relevant constraints, raw sources, and the actual diff/artifact. Withhold the builder's summary and desired verdict on the first pass. Ask them to identify unmet criteria and inspect evidence directly. Keep the builder and reviewer separate; the main agent may build. A reviewer of prose or analysis inspects the changed claims and sources, not just code.

Resolve material findings and recheck affected criteria. If independent review is unavailable, say **self-checked; no independent review**, identify the remaining gap, and continue authorized work where appropriate. Never simulate another reviewer or label a second self-pass independent.

## Make the result reviewable

Give a concise, accurate outcome, then the evidence needed to assess it. For a detailed review packet, order the body as evidence → material decisions/assumptions → gaps → recommendation. Link commands, relevant output excerpts, source records or diffs where useful; redact secrets and avoid raw transcript dumps.

Distinguish **prepared**, **implemented**, **verified**, and **published**. Report unmet criteria plainly; do not describe a plan, draft or partial check as completed delivery. Explain one consequential decision in enough detail that the user can challenge it. Offer a brief teach-back only when it would help learning or judgment.

## Strict supervision loop

1. Before dependent implementation, ask the user for their prediction and definition of done. Prepare a proposed criterion set and do independent read-only discovery while waiting. A missing answer is not agreement. Record the agreed criteria before building.
2. Require a fresh independent review for verification completion. If the environment cannot supply one, prepare the handoff and label the independent-review requirement unmet.
3. After presenting evidence, ask five task-specific questions: intended outcome; key decision and tradeoff; strongest evidence and its limit; principal failure case; remaining uncertainty and next action. Adapt them to the actual work. Do not provide the answer key before the user's response.
4. Evaluate responses against the evidence, explain misunderstandings, and let the user retry. Record teach-back as unassessed until answered. Strict completion requires acceptance evidence, independent review, and five satisfactory answers; the user may explicitly waive or change this preference. Passing the quiz is never authorization for an external action, nor proof that the artifact is correct.

## Source adaptation

Inspired by the user-supplied image, “The Paradox of Supervision,” credited to @Trundur. Its equations and research claims are not treated as verified science. This skill adopts the planning, independent review and evidence principles across work domains; its mandatory quiz belongs only to explicit strict mode.
