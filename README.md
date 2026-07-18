# skill-reviewer

*[中文文档](./README_zh.md)*

An Agent Skill that reviews the quality of other Agent Skills. It's the quality gate every skill wishes it had before shipping: it reads the target skill exactly the way the agent that will actually use it does, and turns "feels off" into a graded, actionable P0/P1/P2 report — so broken triggers, dangling references, and silent logic gaps get caught before they cost you a bad run in production.

> Non-goal: this skill does not create or rewrite skills from scratch (that's the job of skill-creator / darwin-skill and similar authoring tools) — it only reviews and gives feedback.

## Features

- **23-item review checklist**, in three layers:
  - **Mechanical checks (1.1–1.4)**: frontmatter compliance (field whitelist, `name` rules), description trigger quality (keyword-stuffing thresholds, exclusion clauses), Markdown structural integrity, security scan (destructive commands, hardcoded credentials, over-privileged `allowed-tools`)
  - **General checks (2.1–2.16)**: engineering-detail leakage, workflow/tool completeness (including decision/completion criteria and script-example dry-runs), internal contradictions, git version management & asset migration, redundancy & size thresholds, orphaned docs & reference depth, common-sense violations, missing examples, undocumented edge cases, positive-example bias & counter-example pollution, implicit-logic gaps, runtime neutrality, goal/non-goal clarity, reference-loading conditions, structural consistency (terminology drift / near-duplication / structural breaks)
  - **Dependency & install checks (3.1–3.4)**: standalone install instructions, environment isolation, state/credential JSON separation (credentials must be gitignored), failure-driven install guidance
- **User's-eye view**: always reviews as if seeing the target skill for the first time and having to rely on it completely to finish a task
- **Graded report**: every issue is pinned to a location (file:line) + problem + suggested fix, plus a list of passed items, N/A items, and an optional 8-dimension maturity scorecard
- **User-adjudication gate**: ambiguous findings (orphaned docs, suspected intentional deviations, requests to add hard "never do X" rules, narrowing counter-examples) are queued and presented to the user together at the end, instead of interrupting item by item
- **Large-skill splitting**: for skills with more than 20 files, review work is fanned out to parallel sub-agents by branch, while cross-file checks (conflicts, orphaned docs, version management, structural consistency) stay in the main session
- **Two optional deep modes** (off by default, offered after the base review):
  - **Enhanced review**: cross-checks against current domain practice online, flagging staleness / inconsistency / gaps / over-restriction
  - **Behavioral testing**: trigger-rate sampling on positive/negative cases, RED/GREEN/REFACTOR behavioral stress tests, meta feedback

## Install

Option 1: symlink (recommended, keeps you in sync with the repo)

```bash
git clone <this repo> ~/projects/skills/skill-reviewer
ln -s ~/projects/skills/skill-reviewer/skill ~/.claude/skills/skill-reviewer
```

Option 2: use the packaged `.skill` file under `dist/` (a zip archive) — unzip it into your runtime's skills directory as `skill-reviewer/`.

> `~/.claude/skills/` above is Claude Code's directory convention, shown only as an example; for any other runtime that supports Agent Skills, substitute its own skills directory.

## Usage

Just tell an Agent that supports Agent Skills:

- "Review ~/projects/skills/foo for me"
- "Check the quality of this skill"
- "Give me feedback on this SKILL.md"

Workflow: confirm the target → inventory files and the reference graph → review against the checklist → output a graded report and collect adjudication questions →  (optionally) run a deep mode. It only edits the target skill if the user explicitly asks for changes.

## Directory structure

```
skill-reviewer/
├── skill/                          # runtime skill package (symlink target)
│   ├── SKILL.md                    # main workflow
│   └── references/
│       ├── review-checklist.md     # 23-item review checklist
│       ├── report-template.md      # graded report template + scorecard
│       ├── enhanced-review.md      # enhanced review mode
│       └── behavior-testing.md     # behavioral testing mode
├── dist/                           # packaged build artifacts (.skill)
└── README.md
```

## Version

Versioning is managed via git tags; current version is `v0.4.0`.

- `v0.1.0` — Initial release: three-layer checklist, split review, adjudication gate, enhanced review
- `v0.2.0` — Ran a full self-review of the skill on itself (including a real sub-agent split run), fixed 3 P1 issues around split-responsibility alignment; added mechanical-layer checks; rewrote wording to be tool-neutral
- `v0.3.0` — Drew on darwin-skill, obra/superpowers, philschmid, skill-validator, and verdict/skillscore; added behavioral testing mode, security scanning, runtime-neutrality/non-goal/reference-loading-condition checks, scorecard
- `v0.4.0` — Logical-consistency hardening: for skills repeatedly edited by multiple LLMs/agents, added 2.16 structural consistency (terminology drift / near-duplication / structural breaks); 2.2 now also checks decision and completion criteria; 2.12's implicit-logic walkthrough now runs on all skills (conversational-constraint capture remains limited to authoring conversations); scorecard gained a "logical consistency" dimension (8 total)

## Design references

- darwin-skill (sibling skill on this machine) — review dimensions, runtime hygiene, scorecard
- [obra/superpowers](https://github.com/obra/superpowers) — RED/GREEN/REFACTOR sub-agent behavioral testing method
- [Testing Claude Skills](https://www.philschmid.de/testing-skills) — trigger-rate positive/negative sample evaluation
- [skill-validator](https://github.com/agent-ecosystem/skill-validator) — graded token thresholds, keyword-stuffing detection
- [Anthropic Agent Skills Best Practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) — progressive disclosure, avoiding time-sensitive information
