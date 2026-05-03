---
name: debate
description: Stress-test a proposal or decision with a panel of Claude personas. Each expert challenges the idea from their lens, debates across rounds, and a Moderator delivers a verdict. Use when you have a concrete proposal and want it pressure-tested before committing.
---

# Forge — Debate

Stress-test the user's proposal. Convene a panel of Claude sub-agents, each embodying a distinct expert persona, to challenge it from every angle. Produce a synthesis verdict.

## Parse Arguments

Extract from user input (everything after `/forge:debate`):

- `QUESTION`: the proposal or decision to stress-test (required). If empty, print usage and stop.
- `--roles=<list>`: comma-separated role names or preset. Default: `default`.
- `--rounds=<n>`: debate rounds. Default: 2. Min: 1. Max: 5.
- `--mode=<parallel|sequential>`: execution mode within each round. Default: `parallel`.
- `--verbosity=<brief|standard|detailed>`: response depth. Default: `standard`.
- `--file=<path>`: file to include as context. Repeatable.
- `--no-auto-context`: disable automatic file context detection.
- `--no-cache`: skip cache, force fresh.
- `--quiet`: suppress individual responses, show only synthesis.
- `--output=<path>`: also write session to this path.

### Usage (print and stop if no question):

```
Usage: /forge:debate "your proposal" [options]

Options:
  --roles=<list>        Role names or preset (default: default)
  --rounds=<n>          Debate rounds, 1-5 (default: 2)
  --mode=<mode>         parallel or sequential (default: parallel)
  --verbosity=<v>       brief | standard | detailed (default: standard)
  --file=<path>         Include file as context (repeatable)
  --no-auto-context     Skip automatic file detection
  --no-cache            Force fresh session
  --quiet               Show only final verdict
  --output=<path>       Export session to custom path

Presets:
  default       First Principles, Risk Analyst, Pragmatist, Contrarian,
                Synthesizer, Devil's Advocate
  finance       Quant Analyst, Risk Manager, Portfolio Manager, Economist
  product       User Researcher, Product Manager, Growth Strategist,
                Competitive Intel
  engineering   Security Auditor, Performance Optimizer, Maintainability
                Advocate, Devil's Advocate, Simplicity Champion,
                Scalability Architect, Developer Experience, Compliance

Individual roles (default):
  first-principles | risk-analyst | pragmatist | contrarian | synthesizer | devils-advocate

Individual roles (finance):
  quant | risk-manager | portfolio-manager | economist

Individual roles (product):
  user-researcher | product-manager | growth | competitive-intel

Individual roles (engineering):
  security | performance | maintainability | simplicity | scalability |
  developer-experience | compliance

Examples:
  /forge:debate "We should use JWTs in localStorage for auth"
  /forge:debate "Should we raise a Series A now or in 12 months?" --roles=finance
  /forge:debate "Review this auth middleware" --file=src/auth.ts --roles=engineering --rounds=3
```

## Resolve Roles

**Preset → role list:**
- `default` → `first-principles, risk-analyst, pragmatist, contrarian, synthesizer, devils-advocate`
- `finance` → `quant, risk-manager, portfolio-manager, economist`
- `product` → `user-researcher, product-manager, growth, competitive-intel`
- `engineering` → `security, performance, maintainability, simplicity, scalability, developer-experience, compliance`

**Persona path resolution:**
Persona files are at `${CLAUDE_PLUGIN_ROOT}/personas/<domain>/<file>.md`

**Role → file mapping (domain/filename):**

Default: `default/first-principles.md`, `default/risk-analyst.md`, `default/pragmatist.md`, `default/contrarian.md`, `default/synthesizer.md`, `default/devils-advocate.md`

Finance: `finance/quant-analyst.md`, `finance/risk-manager.md`, `finance/portfolio-manager.md`, `finance/economist.md`

Product: `product/user-researcher.md`, `product/product-manager.md`, `product/growth.md`, `product/competitive-intel.md`

Engineering: `engineering/security-auditor.md`, `engineering/performance-optimizer.md`, `engineering/maintainability-advocate.md`, `engineering/simplicity-champion.md`, `engineering/scalability-architect.md`, `engineering/developer-experience.md`, `engineering/compliance-officer.md`

If unrecognized role: print available roles and stop. Cap at 6 personas.

Read each persona file now using the Read tool. Store each persona's name and system prompt.

## Auto-Context

Unless `--no-auto-context` or question is clearly non-technical (finance, strategy, career):

```bash
find . -type f \( -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.py" -o -name "*.go" -o -name "*.md" \) \
  -not -path "*/node_modules/*" -not -path "*/.git/*" | head -20
```

Select up to 5 relevant files. Read them (max 100 lines each). Include `--file` attachments.

If files found: print `📎 Context: file1, file2`

## Cache Check

Unless `--no-cache`:
```bash
echo -n "debate|<roles>|<question>" | shasum -a 256 | cut -d' ' -f1
```
Check `.claude/forge-cache/<key>.json`. If within TTL (default 3600s): print `⚡ Cached (<N> min ago)` and show cached output. Stop.

## Session Header

```
╔══════════════════════════════════════════════════════════╗
║  FORGE — DEBATE                                          ║
╚══════════════════════════════════════════════════════════╝

Proposal: <QUESTION>
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

[SESSION: DEBATE]
Your job: stress-test the proposal below. Critique it from your expert lens.
Find flaws, risks, blind spots, and hidden assumptions. Do not validate — challenge.
Be specific and direct. Rate severity where applicable.

[PROPOSAL]
<QUESTION>

<if round > 1:>
[PRIOR ROUND — READ CAREFULLY]
You are in Round <n>. Below are responses from Round <n-1>.
1. Name what you agree with and why
2. Disagree specifically — cite the persona by name
3. Surface anything critical that wasn't raised
4. Update your position only if the logic warrants it — do not capitulate for consensus

<prior round transcript>

<if context files:>
[CONTEXT]
<file path>:
<file contents>
```

Verbosity: `brief` = 1-2 paragraphs, bullets. `standard` = 3-5 paragraphs. `detailed` = full analysis, all tradeoffs.

## Execute Rounds

For each round 1..N, print:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ROUND <n>
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**parallel**: spawn all persona agents simultaneously (multiple Agent tool calls). Print each result immediately as received.

**sequential**: spawn one at a time. Each receives prior speakers' responses from current round. Print immediately before spawning next.

Persona header:
```
─── Round <n> | <Persona Name> ──────────────────────────────

<response>

```

If `--quiet`: accumulate transcript silently, skip printing responses.

## Moderator Synthesis

Spawn Moderator with full transcript:

```
You are the Moderator. The following proposal was debated:

PROPOSAL: <QUESTION>

TRANSCRIPT:
<transcript>

Produce a structured verdict:

## Consensus
What did most personas agree on? Attribute each point: "(Persona A, Persona B)".

## Divergence
Genuine tensions that remain unresolved. Which personas held which positions and why.

## Recommendation
Concrete path forward. If no clear answer, state what information would resolve it.

## Confidence
HIGH / MEDIUM / LOW — one sentence explaining why.
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

Write `.claude/forge-history/<TIMESTAMP>-debate-<slug>.md` with full session. If `--output=<path>`, write there too. Write cache JSON. Print: `📁 .claude/forge-history/<filename>`
