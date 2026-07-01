# 10x-skills

This repo is a portable, agent-agnostic skill library. The repo name should be
`10xs`, short for `10x skills`.

## Library Shape

Keep the repo organized by broad work domain:

- `code/` - code, pull-request, review, refresh, and engineering workflow skills.

Each skill lives in its own folder with a required `SKILL.md`:

```text
code/
  pr-review/
    SKILL.md
  pr-re-review/
    SKILL.md
  pr-refresh/
    SKILL.md
  scan-prs/
    SKILL.md
```

Stay flat inside each domain until a skill truly needs references, scripts, or
assets.

## Skill Rules

- Write skills as portable workflow modules, not repo-specific playbooks.
- Keep skills agent-agnostic: do not assume any specific agent runtime.
- Keep tool and forge names as examples unless the skill is explicitly an
  adapter for that tool.
- Put personalization in a small `Personalization Knobs` section instead of
  forking the whole workflow.
- Treat organization names, handles, teams, branch names, task text, command
  aliases, and CI names as runtime inputs.
- Prefer local repo conventions over generic examples whenever a skill is used.
- Keep output guidance repo-facing: no local paths, temp paths, session IDs, or
  hidden workspace details.

## Naming

- Use lowercase, hyphenated folder names.
- Prefer verb-led names: `scan-prs`, `pr-refresh`, `review-api-contracts`.
- Keep names stable once other workflows may reference them.
- Use domain folders for grouping rather than prefixes when the skill is part of
  a clear family.

## Portability

Every skill should answer these questions:

- What source of truth should the agent discover first?
- What assumptions must stay as runtime inputs?
- What should be adapted for a new repo?
- What should never be hard-coded?
- How should the workflow degrade when a tool, permission, or live data source
  is unavailable?

If a workflow has no pull-request concept, map PR language to the closest local
unit: merge request, change list, patch set, diff, stack, branch, review task,
or release candidate.

## Building Comprehensive Repo-Local Review Skills

Use the portable skill as the base and add a small repo-local defaults section
for the target repository. Keep the upstream 10xs skill repo-agnostic; put
organization names, reviewer allowlists, exact task text, command names, CI
names, and product-specific risk lenses only in the consuming repo's local
copy.

A good repo-local review skill should add only the context that changes review
quality:

- Source of truth: the forge, PR command/API, task model, approval model, and
  how to fetch comments, tasks, CI, linked issues, and current head SHA.
- Local contracts: repo policy files, contribution rules, public API/CLI
  contracts, generated artifacts, migrations, release/install behavior, and
  docs that must stay synchronized.
- Risk lenses: security/auth, permissions, command/API compatibility, data
  loss, migrations, runtime portability, concurrency, reliability, generated
  instructions, prompt/token budget, and operational blast radius.
- Validation: cheapest meaningful local checks, broad gates, smoke/e2e/manual
  proof requirements, and which missing evidence should block merge.
- Posting workflow: one summary comment shape, blocker/non-blocker split,
  whether review tasks are required, duplicate-task prevention, and final user
  update requirements.
- Approval policy: who may approve, whether approval is expected when clean,
  what counts as a major non-blocker, and how to report tool or permission
  failures.

Keep these repo-local sections short and explicit. Do not fork the whole skill
unless the repository has a genuinely different review model. The portable
workflow should still answer: resolve current head, read existing discussion,
compare against the correct base, review behavioral risk, verify findings, post
clean repo-facing output, and handle approval conservatively.

When adapting a comprehensive review recipe from one repo to another:

1. Extract the generic workflow into the 10xs skill first.
2. Move every repo name, team, command, path, task string, and reviewer identity
   into the target repo's local defaults.
3. Replace product-specific lenses with the new repo's real risk surfaces.
4. Preserve output hygiene: no local paths, temp paths, session IDs, hidden
   artifacts, or investigation mechanics in PR comments.
5. Validate the skill frontmatter and scan for leaked repo-specific text before
   sharing it back to 10xs.

## Validation

When editing a skill, run the available validator against the skill folder:

```bash
python3 /path/to/quick_validate.py code/<skill-name>
```

Also scan for accidental runtime or repo leakage:

```bash
rg -n "specific-agent-name|specific-repo-name|/Users/|/tmp/" code/<skill-name>
```

The scan should not reveal private repo, machine, or agent-runtime assumptions.
