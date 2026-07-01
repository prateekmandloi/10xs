# 10xs

Portable, agent-agnostic skills for repeatable work.

This is a personal skill library that can be copied into different agents,
repos, and workflows. Skills are written as workflow modules rather than
tool-specific prompts, so they should adapt to the local repo, forge, command
line, and review conventions at runtime.

## Current Skills

| Skill | Purpose |
| --- | --- |
| [`code/pr-review`](code/pr-review/SKILL.md) | First-pass pull-request review with live PR context, comment dedupe, findings, validation, and optional review posting. |
| [`code/pr-re-review`](code/pr-re-review/SKILL.md) | Follow-up review after changes, combining prior-comment verification with a full current-head review for remaining, new, or previously missed risk. |
| [`code/pr-refresh`](code/pr-refresh/SKILL.md) | Safely address review feedback, refresh branches, validate, push, and summarize what changed. |
| [`code/scan-prs`](code/scan-prs/SKILL.md) | Scan pull requests for review, re-review, response, refresh, merge, close, or watch actions. |

## How To Use

Copy the skill folder you want into your agent or repo's skill location, or
point your agent at the `SKILL.md` directly if it supports file-based skills.

Each skill is designed to discover local context before acting:

- forge and repository
- default branch and PR target
- review and comment APIs
- test and validation commands
- team, owner, and task conventions

Personalize only the small `Personalization Knobs` section unless the workflow
itself needs to change.

For comprehensive repo-local review behavior, keep 10xs generic and add local
defaults in the consuming repo. See [`skills.md`](skills.md#building-comprehensive-repo-local-review-skills)
for the pattern.

## Repo Structure

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
skills.md
```

See [`skills.md`](skills.md) for authoring and portability guidance.

## Sharing Status

This is an evolving personal library. The skills are intended to be useful as
portable starting points, not a universal framework. Adapt them to the workflow
and risk profile of the repo where you use them.

## License

MIT. See [`LICENSE`](LICENSE).
