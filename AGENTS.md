# nv:dev

Development workflow skills for AI coding agents. The core loop: plan → build → test → debug.

## Skills

- `nv-plan` — Spec-driven development. Write specs with acceptance criteria, decompose into plans with dependency graphs and scope boundaries. Headless mode for automation.
- `nv-test` — TDD and testing. Dependency-graph test selection, property-based testing (Hypothesis), mutation testing (mutmut), quality gates. Separate test/implementation agents.
- `nv-debug` — Systematic debugging. 4-phase: observe → analyze → hypothesize → fix. Loop detection (Repeater/Wanderer/Looper), circuit breakers, verification summaries. ~95% first-time fix rate.

## Install

```bash
npx skills add johnnichev/nv-dev -g -y
```

## Boundaries

### Always
- MUST keep each SKILL.md under 500 lines
- MUST use RFC 2119 language (MUST, SHOULD, NEVER)

### Never
- NEVER remove research-backed claims without replacement data
- NEVER merge skills into a single SKILL.md — one file per skill
