---
inclusion: always
---

# Forge — File Structure

```
forge/
├── .claude-plugin/
│   ├── plugin.json          # Claude Code plugin manifest (name, version, author)
│   └── marketplace.json     # Marketplace catalog (name: "forge", plugins array)
├── .kiro/
│   └── steering/            # Kiro context files
├── skills/
│   ├── debate/
│   │   └── SKILL.md         # /forge:debate — stress-test a proposal
│   ├── hone/
│   │   └── SKILL.md         # /forge:hone — refine a rough idea
│   └── brainstorm/
│       └── SKILL.md         # /forge:brainstorm — generate options from scratch
└── personas/
    ├── default/             # Domain-agnostic thinking-style personas (6)
    │   ├── first-principles.md
    │   ├── risk-analyst.md
    │   ├── pragmatist.md
    │   ├── contrarian.md
    │   ├── synthesizer.md
    │   └── devils-advocate.md
    ├── finance/             # Finance domain overlay (4)
    │   ├── quant-analyst.md
    │   ├── risk-manager.md
    │   ├── portfolio-manager.md
    │   └── economist.md
    ├── product/             # Product domain overlay (4)
    │   ├── user-researcher.md
    │   ├── product-manager.md
    │   ├── growth.md
    │   └── competitive-intel.md
    └── engineering/         # Engineering domain overlay (7)
        ├── security-auditor.md
        ├── performance-optimizer.md
        ├── maintainability-advocate.md
        ├── simplicity-champion.md
        ├── scalability-architect.md
        ├── developer-experience.md
        └── compliance-officer.md
```

## Key paths

- `${CLAUDE_PLUGIN_ROOT}` resolves to the forge plugin root at runtime
- Persona files referenced in SKILL.md as `${CLAUDE_PLUGIN_ROOT}/personas/<domain>/<file>.md`
- Skills invoked as `/forge:debate`, `/forge:hone`, `/forge:brainstorm`

## SKILL.md anatomy

Each SKILL.md follows this flow:
1. Parse args (`--roles`, `--rounds`, `--mode`, `--verbosity`, `--file`, `--quiet`, `--output`)
2. Resolve persona files from `${CLAUDE_PLUGIN_ROOT}/personas/`
3. Auto-detect context files (unless `--no-auto-context`)
4. Check cache
5. Print session header
6. Execute rounds (parallel or sequential Agent spawns)
7. Spawn Moderator for synthesis
8. Save history + cache
