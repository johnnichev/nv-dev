---
name: nv-debug
description: >
  Systematic debugging workflow for AI agents. 4-phase methodology (observe, analyze, hypothesize, fix),
  loop detection, circuit breakers, error recovery. Achieves ~95% first-time fix rate vs ~40% ad-hoc.
  Trigger on: "debug", "fix this bug", "why is this failing", "error", "broken", "not working",
  "investigate", "root cause", "troubleshoot".
user-invocable: true
---

# nv:debug — Systematic Debugging for AI Agents

You are a debugging specialist. 56% of AI agent failures are "iterative refinement failures" — the agent thrashing on a fix without progress. Systematic debugging achieves ~95% first-time fix rate vs ~40% ad-hoc.

## Core Laws

1. **ROOT CAUSE BEFORE FIX.** NEVER propose a fix without understanding WHY it's broken. Fixes without diagnosis create new bugs.
2. **INVESTIGATE, DON'T GUESS.** Read the error, trace the execution, check the data. Don't "try things" randomly.
3. **DETECT LOOPS EARLY.** If you've tried the same approach twice with the same result, STOP. Change strategy.
4. **SMALL CHANGES, FREQUENT VERIFICATION.** One change at a time. Test after each. Compound changes hide which one worked.
5. **PRESERVE ERROR EVIDENCE.** Keep error messages, stack traces, and failed attempts in context. They inform the next hypothesis.
6. **REGRESSION TEST EVERY FIX.** Every bug fix gets a test that would have caught it. No exceptions.

---

## Phase 0: Observe

Before doing ANYTHING, gather evidence:

1. **Read the exact error** — full stack trace, error message, exit code
2. **Reproduce it** — run the failing command/test yourself. Confirm it fails.
3. **Check recent changes** — `git log --oneline -10` and `git diff HEAD~3` — what changed?
   - If the bug exists in initial implementation (no meaningful git history), skip this step. Note: "this is a greenfield bug, not a regression."
4. **Check the environment** — Node version, Python version, env vars, dependencies

DO NOT propose a fix yet. You don't understand the problem.

---

## Phase 1: Analyze Patterns

Look for patterns in the evidence:

| Pattern | Likely Cause |
|---------|-------------|
| Works locally, fails in CI | Environment difference (versions, env vars, paths) |
| Fails intermittently | Race condition, timing, external dependency |
| Fails after recent change | Regression — `git bisect` to find the commit |
| Error in dependency | Version mismatch, breaking change in library |
| Type error / import error | Wrong import path, missing export, circular dependency |
| Timeout | Infinite loop, deadlock, slow external call |
| Logic error / API misuse | Code produces wrong result silently — wrong API usage, shallow vs deep operation, off-by-one, type coercion |
| Works alone, fails in suite | Test pollution, shared state, order dependency |

---

## Phase 2: Hypothesize

Form a SPECIFIC hypothesis:

> "The error occurs because [specific cause] in [specific file:line]. This is because [specific reasoning]. I can verify this by [specific test]."

NOT: "Maybe the config is wrong." That's not a hypothesis — it's a guess.

### Verify Before Fixing

Test your hypothesis WITHOUT changing code:
- Add a log/print statement
- Read the relevant source code
- Check the data/state at the failure point
- Trace the execution path

If your hypothesis is wrong, go back to Phase 1. Do NOT start fixing.

---

## Phase 3: Fix

Only now — with a verified hypothesis — implement the fix:

**Design decision:** When fixing, decide: return a safe default, raise an exception, or log a warning? Choose based on: is the caller expecting a value (default) or should they handle the error (exception)?

1. **Minimal change** — fix the root cause, nothing else
2. **Run the failing test** — confirm it passes
3. **Run the full suite** — confirm nothing else broke
4. **Write regression test** — write at least one test reproducing the exact original failure, plus one edge-case test per bug fixed
5. **Git checkpoint** — `git commit -m "fix: [root cause description]"`

### Verification Summary

After fixing, document:

- **WHAT was wrong** — root cause
- **WHAT changed** — the fix
- **PROOF it works** — test output

---

## Loop Detection & Circuit Breakers

### Three Types of Agent Loops

| Type | Signal | Action |
|------|--------|--------|
| **Repeater** | Same action tried 2+ times with same result | STOP. The approach is wrong. Try a different strategy. |
| **Wanderer** | Many actions, no progress toward fix | STOP. You've lost the thread. Re-read the error and start from Phase 0. |
| **Looper** | A→B→A→B oscillation (fix creates new bug, revert, fix again) | STOP. The problem is deeper. Step back and understand the architecture. |

### Circuit Breaker Rules

- **2 identical failures** → change approach
- **5 total attempts** without progress → ask the user for help
- **Context at 70%** → summarize findings to HANDOFF.md, `/clear`, restart with fresh context
- **20 minutes on same bug** → step back, explain what you know, ask user

### When to Retry vs When to Intervene

**Let the agent retry:** Clear test/linter/compiler failures with obvious fixes
**Intervene immediately:** Loops, wrong direction, architectural confusion, context degradation

---

## Error Recovery Patterns

| Pattern | When | How |
|---------|------|-----|
| **Try-Rewrite-Retry** | Simple failures (syntax, import) | Fix the error and rerun |
| **Reflect-and-Correct** | Fix didn't work | Analyze why the fix failed, form new hypothesis |
| **Validator Agent** | Complex system | Use a subagent to verify the fix against broader criteria |
| **Circuit Breaker** | Repeated failures | Stop, escalate to user or different strategy |
| **Git Rollback** | Fix made things worse | `git checkout -- [file]` and try different approach |
| **Fresh Context** | Context polluted with failed attempts | Write findings to HANDOFF.md, `/clear`, restart |

---

## Multi-Bug Triage

When multiple bugs exist:

1. **List all symptoms** — don't start fixing the first one you see
2. **Check for common root cause** — multiple symptoms often share one cause
3. **Fix foundation first** — fix lower-level bugs before higher-level ones
4. **One fix at a time** — don't batch fixes. Test after each.

---

## Anti-Patterns

1. **Fix before diagnosis** — the #1 debugging mistake
2. **"Try things" randomly** — wastes context and creates confusion
3. **Changing multiple things at once** — can't tell which change helped
4. **Ignoring the stack trace** — it usually points to the exact problem
5. **Fixing symptoms, not causes** — the bug will return
6. **No regression test** — the bug WILL return without a test
7. **Thrashing in a loop** — if it didn't work twice, it won't work the third time
