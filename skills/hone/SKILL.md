---
name: hone
description: Refine and sharpen a rough idea with a panel of Claude personas. Each expert improves it from their lens — filling gaps, adjusting the approach, strengthening weak points. Use when you have a direction but it's not fully formed yet.
---

# Forge — Hone

Refine the user's rough idea. Convene a panel of Claude sub-agents to improve, fill gaps, and sharpen the thinking. No tearing down — building up.

## Parse Arguments

Extract from user input (everything after `/forge:hone`):

- `QUESTION`: the rough idea to refine (required). If empty, print usage and stop.
- `--roles=<list>`: comma-separated role names or preset. Default: `default`.
- `--rounds=<n>`: refinement rounds. Default: 2. Min: 1. Max: 5.
- `--mode=<parallel|sequential>`: execution mode within each round. Default: `parallel`.
- `--verbosity=<brief|standard|detailed>`: response depth. Default: `standard`.
- `--file=<path>`: file to include as context. Repeatable.
- `--no-auto-context`: disable automatic file context detection.
- `--no-cache`: skip cache, force fresh.
- `--quiet`: suppress individual responses, show only synthesis.
- `--output=<path>`: also write session to this path.

### Usage (print and stop if no question):

```
Usage: /forge:hone "your rough idea" [options]

Options:
  --roles=<list>        Role names or preset (default: default)
  --rounds=<n>          Rounds, 1-5 (default: 2)
  --mode=<mode>         parallel or sequential (default: parallel)
  --verbosity=<v>       brief | standard | detailed (default: standard)
  --file=<path>         Include file as context (repeatable)
  --no-auto-context     Skip automatic file detection
  --no-cache            Force fresh session
  --quiet               Show only final synthesis
  --output=<path>       Export session to custom path

Presets:  default | finance | product | engineering
(same role/preset system as /forge:debate)

Examples:
  /forge:hone "Thinking of using Redis with a 24h TTL for sessions"
  /forge:hone "We want to charge usage-based pricing" --roles=product
  /forge:hone "Series A at $8M ARR, 3x growth" --roles=finance
```

## Resolve Roles

Same as forge:debate. Read each persona file using the Read tool.

**Persona path resolution:**
Your base directory is provided in context as `Base directory for this skill: <path>`.
Persona files live two levels up: `<base-dir>/../../personas/<domain>/<file>.md`

**Role → file mapping (domain/filename):**

Default: `default/first-principles.md`, `default/risk-analyst.md`, `default/pragmatist.md`, `default/contrarian.md`, `default/synthesizer.md`, `default/devils-advocate.md`

Finance: `finance/quant-analyst.md`, `finance/risk-manager.md`, `finance/portfolio-manager.md`, `finance/economist.md`

Product: `product/user-researcher.md`, `product/product-manager.md`, `product/growth.md`, `product/competitive-intel.md`

Engineering: `engineering/security-auditor.md`, `engineering/performance-optimizer.md`, `engineering/maintainability-advocate.md`, `engineering/simplicity-champion.md`, `engineering/scalability-architect.md`, `engineering/developer-experience.md`, `engineering/compliance-officer.md`

Cap at 6 personas. If unrecognized role: print available and stop.

## Auto-Context

Unless `--no-auto-context` or question is clearly non-technical: search working directory for relevant files (max 5, 100 lines each). Include `--file` attachments. Print: `📎 Context: file1, file2`

## Cache Check

Unless `--no-cache`:
```bash
echo -n "hone|<roles>|<question>" | shasum -a 256 | cut -d' ' -f1
```
Check `.claude/forge-cache/<key>.json`. If within TTL: print `⚡ Cached (<N> min ago)` and show output. Stop.

## Session Header

```
╔══════════════════════════════════════════════════════════╗
║  FORGE — HONE                                            ║
╚══════════════════════════════════════════════════════════╝

Idea:     <QUESTION>
Personas: <list>
Rounds:   <N> | Mode: <parallel|sequential>
```

## Build Persona Prompt

For each persona:

```
[VERBOSITY: <level>]
<verbosity directive>

[ROLE: <Persona Name>]
<persona system prompt>

[SESSION: HONE]
Your job: improve and refine the idea below. Do NOT tear it down — build it up.
From your expert lens:
- What would you strengthen or sharpen?
- What gap or blind spot should be filled?
- What would you adjust, and why?
Be constructive and specific. Suggest concrete improvements, not abstract critiques.

[ROUGH IDEA]
<QUESTION>

<if round > 1:>
[PRIOR ROUND — BUILD ON IT]
Round <n>. Below are refinements from Round <n-1>.
Build on the strongest improvements. Add what's still missing.
Synthesize where multiple personas converge.

<prior round transcript>

<if context files:>
[CONTEXT]
<file path>:
<file contents>
```

Verbosity: `brief` = 1-2 paragraphs, bullets. `standard` = 3-5 paragraphs. `detailed` = full analysis.

## Execute Rounds

For each round, print:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ROUND <n>
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**parallel**: spawn all simultaneously. Print each result immediately.

**sequential**: spawn one at a time, each sees prior speakers' output. Print immediately.

```
─── Round <n> | <Persona Name> ──────────────────────────────

<response>

```

## Moderator Synthesis

Spawn Moderator:

```
You are the Moderator. The following rough idea was refined by a council:

IDEA: <QUESTION>

TRANSCRIPT:
<transcript>

Synthesize the refinements:

## Strongest Improvements
Top 3 refinements raised, with attribution to the persona who surfaced them.

## Remaining Gaps
What still needs to be resolved before this idea is solid?

## Sharpened Version
Restate the original idea incorporating the best refinements. Be concrete and specific.

## Confidence
HIGH / MEDIUM / LOW — is this idea ready to execute, or does it need more work?
One sentence explaining why.
```

Print:
```
══════════════════════════════════════════════════════════════
  FORGE VERDICT
══════════════════════════════════════════════════════════════

<synthesis>

══════════════════════════════════════════════════════════════
```

## Save

```bash
mkdir -p .claude/forge-history .claude/forge-cache
TIMESTAMP=$(date +%Y%m%d-%H%M%S)
```

Write `.claude/forge-history/<TIMESTAMP>-hone-<slug>.md`. If `--output=<path>`, write there too. Write cache JSON. Print: `📁 .claude/forge-history/<filename>`
