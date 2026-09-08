# project-rules skill

A portable, project-agnostic OpenCode skill that gives any agent
working in your repo the same hard rules + session-continuity
scaffolding.

## What it does

- **Hard rules** that survive subagent dispatch. The skill body is
  the canonical source of truth for data integrity, commit hygiene,
  shared components, file ownership, session continuity, and process
  hygiene. Subagents don't auto-inject `AGENTS.md`, so handing them
  the skill via the dispatch prompt is how the rules reach them.
- **Fresh-repo bootstrap**. Invoke the skill in an empty repo and
  it scaffolds `AGENTS.md` + `project_rules/` (HANDOFF, SESSION_LOG,
  DECISIONS, RUNBOOK, ROADMAP) from the shipped `templates/` — every
  file carries `{{...}}` placeholders the agent fills at bootstrap
  time.
- **Retrofit mode for existing repos.** Point the skill at a repo
  with an existing `AGENTS.md` (the common case). The skill
  classifies each section as (a) universal hard rule → propose
  merge into Core rules, (b) project-specific reference → extract
  verbatim into the matching
  `project_rules/{ARCHITECTURE,API,TESTING,DECISIONS}.md`, or
  (c) stale/derivable content → flag for deletion. It then
  rewrites the root `AGENTS.md` down to hard-rule pointers plus a
  see-also index. **`project_rules/DECISIONS.md` is appended to,
  never clobbered** — the history of "why" stays intact.
- **Hands-off maintenance**. Once scaffolded, every rule in the
  skill maps to a maintenance action: update HANDOFF on session end,
  append SESSION_LOG entries as work happens, record decisions in
  DECISIONS the moment they're confirmed, keep RUNBOOK in sync with
  operational commands.

## Installation

This skill uses the [OpenCode skill convention](https://opencode.ai/docs/skills/):
a `SKILL.md` file with YAML frontmatter (`name`, `description`),
loaded by the harness's `skill` tool. Companion files in the same
directory (e.g. `templates/`) are accessible by path but not auto-
loaded.

To install:

1. Copy `SKILL.md` and the `templates/` directory into
   `.opencode/skills/project-rules/` of the target repo.
2. Load the skill with your harness's `skill` tool.
3. In a fresh repo, the skill will scaffold the docs on invocation.
   In an existing repo, load the skill before any non-trivial work so
   the rules reach every subagent dispatch.

## Initialisation

Before using the skill in a new repo, edit the `Initialisation`
table at the top of `SKILL.md` and replace the seven placeholders
(`{{REPO_NAME}}`, `{{REPO_DISPLAY_NAME}}`, `{{DATA_DIR}}`,
`{{SERVER_COMMAND}}`, `{{FROZEN_HTML_DIR}}`,
`{{DOC_LINE_THRESHOLD}}`, `{{SESSION_LOG_ROTATION_ENTRIES}}`)
with values matching the target repo. The two documentation-
hygiene placeholders (`{{DOC_LINE_THRESHOLD}}` for the docs-file
split check, `{{SESSION_LOG_ROTATION_ENTRIES}}` for the
`SESSION_LOG.md` rotation point) are per-project tunable — see
Core rules → Documentation hygiene in `SKILL.md`.

## File structure

```
project-rules/
├── SKILL.md                 # canonical rules + bootstrap instructions
├── templates/               # seed files for fresh-repo bootstrap
│   ├── AGENTS.md            #   session protocol stub
│   ├── README.md            #   documents the template set
│   └── project_rules/       #   the five living-doc seeds
│       ├── HANDOFF.md
│       ├── SESSION_LOG.md
│       ├── DECISIONS.md
│       ├── RUNBOOK.md
│       └── ROADMAP.md
└── LICENSE
```

## What the skill is NOT

The skill is a **portable core**. It does not ship project-specific
rules — those belong in the repo's own `project_rules/DECISIONS.md`
once the project exists. The rule of thumb for adding to the skill:

> If the rule applies to any project, it belongs in the skill. If it
> only applies to projects with a specific feature (a risk gauge,
> commodity spot pricing, a particular UI framework), it belongs in
> that repo's DECISIONS.md.

## License

MIT — see `LICENSE`.
