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
