---
name: brainstorm
description: Generate ideas and options from scratch with a panel of Claude personas. No idea exists yet — each expert proposes approaches from their lens. Use when you have a problem but no solution yet and want diverse options before committing to a direction.
---

# Forge — Brainstorm

Generate options for the user's problem. Convene a panel of Claude sub-agents, each proposing concrete approaches from their expert lens. Synthesize the best ideas into a recommended starting point.

## Parse Arguments

Extract from user input (everything after `/forge:brainstorm`):

- `QUESTION`: the problem or challenge to generate ideas for (required). If empty, print usage and stop.
- `--roles=<list>`: comma-separated role names or preset. Default: `default`.
- `--rounds=<n>`: rounds. Default: 2. Min: 1. Max: 5.
- `--mode=<parallel|sequential>`: execution mode within each round. Default: `parallel`.
- `--verbosity=<brief|standard|detailed>`: response depth. Default: `standard`.
- `--file=<path>`: file to include as context. Repeatable.
- `--no-auto-context`: disable automatic file context detection.
- `--no-cache`: skip cache, force fresh.
- `--quiet`: suppress individual responses, show only synthesis.
- `--output=<path>`: also write session to this path.

### Usage (print and stop if no question):

```
Usage: /forge:brainstorm "your problem or challenge" [options]

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

Examples:
  /forge:brainstorm "How do we build a hedge fund algorithm?" --roles=finance
  /forge:brainstorm "How should we monetize our developer tool?" --roles=product
  /forge:brainstorm "How do we handle rate limiting?" --roles=engineering
  /forge:brainstorm "Should I join a startup or big tech?"
```

## Resolve Roles

**Persona path resolution:**
Persona files are at `${CLAUDE_PLUGIN_ROOT}/personas/<domain>/<file>.md`

**Role → file mapping (domain/filename):**

Default: `default/first-principles.md`, `default/risk-analyst.md`, `default/pragmatist.md`, `default/contrarian.md`, `default/synthesizer.md`, `default/devils-advocate.md`

Finance: `finance/quant-analyst.md`, `finance/risk-manager.md`, `finance/portfolio-manager.md`, `finance/economist.md`

Product: `product/user-researcher.md`, `product/product-manager.md`, `product/growth.md`, `product/competitive-intel.md`

Engineering: `engineering/security-auditor.md`, `engineering/performance-optimizer.md`, `engineering/maintainability-advocate.md`, `engineering/simplicity-champion.md`, `engineering/scalability-architect.md`, `engineering/developer-experience.md`, `engineering/compliance-officer.md`

Cap at 6 personas. If unrecognized role: print available and stop.

Read each persona file using the Read tool. Store name and system prompt.

## Auto-Context

Unless `--no-auto-context` or clearly non-technical: search for relevant files (max 5, 100 lines each). Include `--file` attachments. Print: `📎 Context: file1, file2`

## Cache Check

Unless `--no-cache`:
```bash
echo -n "brainstorm|<roles>|<question>" | shasum -a 256 | cut -d' ' -f1
```
Check `.claude/forge-cache/<key>.json`. If within TTL: print `⚡ Cached (<N> min ago)` and show output. Stop.

## Session Header

```
╔══════════════════════════════════════════════════════════╗
║  FORGE — BRAINSTORM                                      ║
╚══════════════════════════════════════════════════════════╝

Problem:  <QUESTION>
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

[SESSION: BRAINSTORM]
Your job: generate options. No solution exists yet — you are creating them.
From your expert lens, propose 2-3 concrete approaches or solutions.
For each option:
- Give it a name
- Describe it in 2-3 sentences
- State when it's the right choice and when it isn't

Round 1: generate independently. Do NOT critique other personas' ideas yet.

[PROBLEM]
<QUESTION>

<if round > 1:>
[PRIOR ROUND — BUILD ON THE BEST]
Round <n>. Ideas generated so far:

<prior round transcript>

Now:
1. Identify the 1-2 most promising ideas from any persona (name them by persona + idea name)
2. Develop them further OR combine them into something stronger
3. Add one idea that hasn't been raised yet from your lens

<if context files:>
[CONTEXT]
<file path>:
<file contents>
```

Verbosity: `brief` = compact bullets per option. `standard` = 2-3 sentences per option with tradeoffs. `detailed` = full option analysis with examples, risks, and when-to-use.

## Execute Rounds

For each round:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ROUND <n>
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**parallel**: spawn all simultaneously. Print each result immediately.

**sequential**: spawn one at a time, each sees prior speakers' ideas. Print immediately.

```
─── Round <n> | <Persona Name> ──────────────────────────────

<response>

```

## Moderator Synthesis

Spawn Moderator:

```
You are the Moderator. A council brainstormed solutions to this problem:

PROBLEM: <QUESTION>

TRANSCRIPT:
<transcript>

Synthesize the best ideas:

## Top Options
The 3 most promising approaches surfaced. For each:
- Name and brief description
- Who proposed it (attribution)
- Key strength and key risk
- Best fit: when is this the right choice?

## When to Use Each
Decision framework: given context X, choose option Y because Z.

## Recommended Starting Point
If you had to pick one option to explore first, which and why?
Be direct — if there's a clear winner, say so.

## Open Questions
What needs to be answered before committing to any option?
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

Write `.claude/forge-history/<TIMESTAMP>-brainstorm-<slug>.md`. If `--output=<path>`, write there too. Write cache JSON. Print: `📁 .claude/forge-history/<filename>`
