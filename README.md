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
  it scaffolds `AGENTS.md` + `project_rules/` from the shipped
  `templates/` — nine seed files (HANDOFF, SESSION_LOG, DECISIONS,
  RUNBOOK, ROADMAP, ARCHITECTURE, ARCHITECTURE_DETAILS, API, TESTING)
  carry `{{...}}` placeholders the agent fills at bootstrap time.
  See "Conditional docs" below for which to skip.
- **Retrofit mode for existing repos.** Point the skill at a repo
  with an existing `AGENTS.md` (the common case). The skill
  classifies each section as (a) universal hard rule → propose
  merge into Core rules, (b) project-specific reference → extract
  verbatim into the matching `project_rules/*.md`, or
  (c) stale/derivable content → flag for deletion. It then
  rewrites the root `AGENTS.md` down to hard-rule pointers plus a
  see-also index. **`project_rules/DECISIONS.md` is appended to,
  never clobbered** — the history of "why" stays intact.
- **Documentation hygiene.** The skill enforces a
  `{{DOC_LINE_THRESHOLD}}`-line self-check on `AGENTS.md` and the
  `project_rules/*.md` files. When a file crosses the threshold *and*
  has grown materially since its last edit, the skill proposes a
  split in that same session (not deferred). The split follows the
  Retrofit classifier — extract project-specific material to
  `project_rules/`, hoist universal rules into Core, flag derivable
  content for deletion. See Core rules → Documentation hygiene in
  `SKILL.md` for the full rule.
- **Pointer-archive patterns.** Two of the live files (`DECISIONS.md`
  and `SESSION_LOG.md`) use a **live-index + archive** layout to keep
  required-reads cheap. DECISIONS.md is a pointer index (each entry
  → `archive/decisions/<slug>.md`); SESSION_LOG.md is a hybrid
  (latest entry in full, older entries → `archive/sessions/<slug>.md`).
  When the live file exceeds `{{SESSION_LOG_ROTATION_ENTRIES}}`
  entries, the oldest pointer drops — the archive file is the
  source of truth.
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
├── README.md                # this file
├── templates/               # seed files for fresh-repo bootstrap
│   ├── AGENTS.md            #   session protocol stub
│   ├── README.md            #   documents the template set
│   └── project_rules/       #   the nine living-doc seeds
│       │                    #   (five core + four conditional)
│       ├── HANDOFF.md       #     session-to-session state        (core)
│       ├── SESSION_LOG.md   #     append-only dated log           (core)
│       ├── DECISIONS.md     #     durable decisions + rationale   (core)
│       ├── RUNBOOK.md       #     operational commands            (core)
│       ├── ROADMAP.md       #     phase-level plan                (core)
│       ├── ARCHITECTURE.md          #  high-level overview     (conditional)
│       ├── ARCHITECTURE_DETAILS.md  #  deep-dive per-module    (conditional)
│       ├── API.md                   #  HTTP routes             (conditional)
│       └── TESTING.md               #  test suite layout       (conditional)
└── LICENSE
```

## Conditional docs

Four of the nine `project_rules/*.md` templates are conditional — only
written if the project needs them. The bootstrap should ask which
apply and skip the rest:

| Template | When to include | When to skip |
|----------|-----------------|--------------|
| `ARCHITECTURE.md` | Almost always — most projects have a module layout worth documenting | Pure data / config repo with no code |
| `ARCHITECTURE_DETAILS.md` | Only after `ARCHITECTURE.md` crosses the `{{DOC_LINE_THRESHOLD}}`-line split threshold (see SKILL.md Core rules → Documentation hygiene) | Most projects — created reactively when needed |
| `API.md` | Project exposes an HTTP API | CLI tool, library, or background service with no HTTP surface |
| `TESTING.md` | Project has a test suite | Toy project / experiment; tests are one-off |

Edit `AGENTS.md` "See also" at bootstrap to drop the conditional files
the project doesn't need.

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
