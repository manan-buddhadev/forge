---
inclusion: always
---

# Forge — Product Overview

Forge is a multi-agent idea refinement tool. It convenes a panel of AI expert personas to debate, refine, or brainstorm any question. Each expert argues from their own lens. A Moderator synthesizes a structured verdict.

Use it when you want a second opinion that actually pushes back.

## Three modes

- `debate` — stress-test a proposal. Personas actively critique and challenge it.
- `hone` — refine a rough idea. Personas build it up constructively, no tearing down.
- `brainstorm` — generate options from scratch when no solution exists yet.

## How it works

Forge spawns one expert persona per round. In `parallel` mode all personas respond independently. In `sequential` mode each reads prior responses before replying. A Moderator synthesizes the full transcript into a verdict: consensus, divergence, recommendation, confidence.

## Persona presets

- `default` (6 personas): First Principles, Risk Analyst, Pragmatist, Contrarian, Synthesizer, Devil's Advocate — works for any question
- `finance` (4 personas): Quant Analyst, Risk Manager, Portfolio Manager, Economist
- `product` (4 personas): User Researcher, PM, Growth Strategist, Competitive Intel
- `engineering` (7 personas): Security, Performance, Maintainability, Simplicity, Scalability, DX, Compliance

## Usage by environment

- **Claude Code**: `/plugin install forge@forge` → `/forge:debate "..."`
- **Kiro / any AI IDE**: `forge debate: <question>` in chat — steering file teaches the AI to run the full session
- **Terminal CLI**: `npx forge debate "..."` (coming soon)
