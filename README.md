# Forge

A Claude Code skill that convenes a panel of AI personas to debate, refine, or brainstorm any question. Three modes: stress-test a proposal, sharpen a rough idea, or generate options from scratch. Finance, product, and engineering domain overlays. No external APIs — pure Claude.

## Skills

| Command | Use when |
|---|---|
| `/forge:debate` | You have a proposal and want it stress-tested |
| `/forge:hone` | You have a rough idea and want it sharpened |
| `/forge:brainstorm` | You have a problem and need options generated |

## Persona Presets

| Preset | Personas | Best for |
|---|---|---|
| `default` | First Principles, Risk Analyst, Pragmatist, Contrarian, Synthesizer, Devil's Advocate | Any domain |
| `finance` | Quant Analyst, Risk Manager, Portfolio Manager, Economist | Investing, fund strategy, financial decisions |
| `product` | User Researcher, PM, Growth Strategist, Competitive Intel | Product strategy, monetization, go-to-market |
| `engineering` | Security, Performance, Maintainability, Simplicity, Scalability, DX, Compliance | Technical architecture and code review |

## Usage

```bash
# Stress-test a proposal
/forge:debate "We should store sessions in Redis with a 24h TTL" --roles=engineering

# Sharpen a rough financial idea
/forge:hone "Series A at $8M ARR, 3x growth" --roles=finance

# Generate options from scratch
/forge:brainstorm "How do we build a hedge fund algorithm?" --roles=finance
/forge:brainstorm "How should we monetize our dev tool?" --roles=product

# Default personas work for anything
/forge:debate "Should I join a startup or big tech?"
/forge:brainstorm "How do we reduce customer churn?"
```

## Options

```
--roles=<list>        Role names or preset (default: default)
--rounds=<n>          Rounds 1-5 (default: 2)
--mode=<mode>         parallel or sequential (default: parallel)
--verbosity=<v>       brief | standard | detailed (default: standard)
--file=<path>         Attach file as context
--no-auto-context     Skip automatic file detection
--no-cache            Force fresh session
--quiet               Show only final verdict
--output=<path>       Export session to markdown
```

## Installation

Add to your Claude Code project:

```bash
cd your-project/.claude/skills
git clone https://github.com/manan-buddhadev/forge
```

Or install globally:

```bash
cd ~/.claude/plugins
git clone https://github.com/manan-buddhadev/forge
```

Sessions save to `.claude/forge-history/`. Cache in `.claude/forge-cache/`.

## How It Works

Each `/forge` command spawns one Claude sub-agent per persona using the Agent tool. In `parallel` mode all personas run simultaneously. In `sequential` mode each persona reads prior responses before replying. A Moderator agent synthesizes the full transcript into a structured verdict.
