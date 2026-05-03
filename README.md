# Forge

A Claude Code skill that convenes a panel of AI personas to debate, refine, or brainstorm any question. Three modes: stress-test a proposal, sharpen a rough idea, or generate options from scratch. Finance, product, and engineering domain overlays. No external APIs — pure Claude.

---

## Skills

| Command | Use when |
|---|---|
| `/forge:debate` | You have a proposal and want it stress-tested |
| `/forge:hone` | You have a rough idea and want it sharpened |
| `/forge:brainstorm` | You have a problem and need options generated |

---

## Install as plugin

```
/plugin marketplace add manan-buddhadev/forge
/plugin install forge:debate@forge
/plugin install forge:hone@forge
/plugin install forge:brainstorm@forge
```

---

## Install as standalone skills

```bash
git clone https://github.com/manan-buddhadev/forge.git /tmp/forge
cp -r /tmp/forge/skills/debate ~/.claude/skills/forge-debate
cp -r /tmp/forge/skills/hone ~/.claude/skills/forge-hone
cp -r /tmp/forge/skills/brainstorm ~/.claude/skills/forge-brainstorm
cp -r /tmp/forge/personas ~/.claude/skills/forge-debate/../../personas 2>/dev/null || true
```

---

## Usage

```
/forge:debate "We should use JWTs in localStorage for auth" --roles=engineering
/forge:hone "Series A at $8M ARR, 3x growth" --roles=finance
/forge:brainstorm "How do we build a hedge fund algorithm?" --roles=finance
/forge:brainstorm "How should we monetize our dev tool?" --roles=product
/forge:debate "Should I join a startup or big tech?"
```

---

## Persona Presets

| Preset | Personas | Best for |
|---|---|---|
| `default` | First Principles, Risk Analyst, Pragmatist, Contrarian, Synthesizer, Devil's Advocate | Any domain |
| `finance` | Quant Analyst, Risk Manager, Portfolio Manager, Economist | Investing, fund strategy, financial decisions |
| `product` | User Researcher, PM, Growth Strategist, Competitive Intel | Product strategy, monetization, go-to-market |
| `engineering` | Security, Performance, Maintainability, Simplicity, Scalability, DX, Compliance | Technical architecture and code review |

---

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

Sessions save to `.claude/forge-history/`. Cache in `.claude/forge-cache/`.

---

## How It Works

Each `/forge` command spawns one Claude sub-agent per persona using the Agent tool. In `parallel` mode all personas run simultaneously. In `sequential` mode each persona reads prior responses before replying. A Moderator agent synthesizes the full transcript into a structured verdict.

---

## License

MIT
