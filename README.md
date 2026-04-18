# Skill Workshop

Analyze your Claude Code session history to discover patterns that should become reusable skills.

Skill Workshop mines JSONL session files, finds repeating explanations, tool-chain workflows, and error→workaround patterns, then proposes skill candidates ranked by score. You pick the winners — it generates ready-to-use `SKILL.md` drafts.

## What it finds

| Signal type | What it catches | Example |
|---|---|---|
| **Repeated explanation** | Same concept explained to Claude in 3+ sessions | "Always use `--break-system-packages` with pip on this machine" |
| **Tool chain** | Same sequence of tools used across sessions | `Grep → Read → Edit → Bash` for a specific workflow |
| **Workaround** | Errors followed by user corrections | Claude keeps using wrong flags, user keeps fixing it |

## Installation

Copy the contents into your Claude Code configuration:

```
~/.claude/
├── agents/
│   └── session-analyzer.md    # Haiku-based extraction agent
└── skills/
    └── skill-workshop/
        └── SKILL.md            # Orchestrator skill
```

Or symlink from this repo:

```bash
ln -s /path/to/skill-workshop/agents/session-analyzer.md ~/.claude/agents/session-analyzer.md
mkdir -p ~/.claude/skills/skill-workshop
ln -s /path/to/skill-workshop/skills/skill-workshop/SKILL.md ~/.claude/skills/skill-workshop/SKILL.md
```

## Usage

In any Claude Code session, run:

```
/skill-workshop
```

Or with an explicit project path:

```
/skill-workshop /path/to/project
```

The skill will:
1. Find session files for the target project
2. Delegate extraction to the `session-analyzer` agent (runs on Haiku for speed/cost)
3. Present ranked candidates with evidence quotes
4. Ask which candidates to develop into skill drafts
5. Generate `SKILL.md` files in `.claude/skills/`

## How it works

**Phase 1 — Extract & Analyze** (session-analyzer agent):
- Parses JSONL session files using bash + jq/python3
- Extracts user messages, tool sequences, and error→correction pairs
- Groups by semantic similarity, scores candidates (`frequency × type_weight`)
- Writes results to `/tmp/skill-workshop-results.json`

**Phase 2 — Present & Generate** (skill-workshop orchestrator):
- Reads analysis results, presents candidates in a table
- For approved candidates, generates proper SKILL.md files following skill authoring best practices
- Outputs skills categorized as `knowledge`, `workflow`, or `gotcha`

## Prerequisites

- Claude Code with skills and agents support
- Session history for the target project (Claude Code stores sessions in `~/.claude/projects/`)

> **Tip:** Claude Code deletes sessions after 30 days by default. To keep longer history, add `"cleanupPeriodDays": 100000` to `~/.claude/settings.json`.

## Output

Results are written to `/tmp/skill-workshop-results.json` — a structured JSON with:
- Project metadata and date range
- Extraction stats (messages parsed, tool calls, errors found)
- Up to 15 candidates ranked by score (0.0–1.0), each with evidence quotes

## License

MIT
