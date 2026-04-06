# nv:dev

**The development workflow loop for AI coding agents.**

```bash
npx skills add johnnichev/nv-dev -g -y
```

3 skills that work together for the core development cycle:

| Skill | Command | What it does |
|-------|---------|--------------|
| **nv:plan** | `/nv-plan` | Spec-driven development. INTENT → SPEC → PLAN → IMPLEMENT → VERIFY. Headless mode for automation. |
| **nv:test** | `/nv-test` | TDD with property-based testing, mutation testing, dependency-graph test selection. Separate test/implementation agents prevent tautological tests. |
| **nv:debug** | `/nv-debug` | 4-phase systematic debugging (observe → analyze → hypothesize → fix). Loop detection (Repeater/Wanderer/Looper). ~95% first-time fix rate vs ~40% ad-hoc. |

## Why This Bundle

These skills aren't independent — they form a loop:

```
nv:plan → write spec with acceptance criteria
         ↓
nv:test → tests verify the acceptance criteria
         ↓ (something fails)
nv:debug → systematic root-cause analysis
         ↓ (fix found)
nv:test → regression test prevents recurrence
         ↓
nv:plan → next feature
```

## Research-Backed

| Finding | Source |
|---------|--------|
| Telling agents HOW to do TDD increases regressions by 64% | TDAD paper (arXiv 2026) |
| Dependency-graph test selection reduces regressions by 70% | TDAD paper |
| AI creates 1.7x as many bugs as humans | Stack Overflow 2026 |
| 56% of agent failures are "iterative refinement failures" | Columbia University |
| ~95% first-time fix rate with systematic debugging vs ~40% ad-hoc | Multiple sources |

## Install

```bash
npx skills add johnnichev/nv-dev -g -y
```

Installs all 3 skills globally. Works with Claude Code, Cursor, GitHub Copilot, Windsurf, Aider, Gemini CLI, and 9+ other tools.

## Use

Open any project and trigger by topic:

```
/nv-plan      # When starting a new feature
/nv-test      # When writing tests
/nv-debug     # When something is broken
```

Or just talk naturally — skills auto-trigger on keywords.

## See Also

- **[nv:context](https://github.com/johnnichev/nv-context)** — Set up context engineering for your repo (the foundation)
- **[nv:ops](https://github.com/johnnichev/nv-ops)** — Guardrails, evaluation, multi-agent orchestration
- **[nv:design](https://github.com/johnnichev/nv-design)** — Professional web design with AI

## License

MIT

---

**Built by [johnnichev](https://github.com/johnnichev).** For engineers who ship.
