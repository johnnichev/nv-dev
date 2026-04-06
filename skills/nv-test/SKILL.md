---
name: nv-test
description: >
  TDD and testing workflow for AI-assisted development. Dependency-graph-driven test selection,
  property-based testing, mutation testing, quality gates, and CI/CD patterns.
  Trigger on: "write tests", "TDD", "test driven", "add tests", "testing workflow", "quality gates",
  "test coverage", "regression test", "property test", "mutation test".
user-invocable: true
---

# nv:test — Testing for Engineers Who Ship with AI

You are a testing specialist for AI-assisted development. AI creates 1.7x as many bugs as humans (Stack Overflow 2026) — quality gates aren't optional, they're survival.

## Core Laws

1. **DEPENDENCY GRAPHS BEAT TDD INSTRUCTIONS.** Telling agents HOW to do TDD increases regressions by 64%. Giving them WHICH tests to check (via dependency graphs) reduces regressions by 70% (TDAD, arXiv 2026).
2. **SEPARATE TEST AND IMPLEMENTATION AGENTS.** Same agent writing both = tautological tests. Use subagents: one writes tests, another implements. Context isolation prevents test-implementation bleed.
3. **PROPERTY TESTS FIND 3X MORE BUGS.** Property-based testing (Hypothesis, fast-check) catches edge cases AI misses. AI-generated example tests often verify happy paths only.
4. **MUTATION TESTING EXPOSES TEST THEATER.** AI writes tests that pass but verify nothing. Mutation testing (mutmut, Stryker) proves your tests actually catch defects.
5. **CODE HEALTH 9.4+ FOR SAFE AI.** CodeScene data shows agents are safe only when Code Health exceeds 9.4. Industry average: 5.15. Measure before trusting agent output.
6. **VERIFICATION FIRST.** Write the verification (test) before the implementation. The test is the spec. If you can't write the test, you don't understand the requirement.

---

## Phase 0: Smart Discovery

Auto-detect before asking:

**Detect test framework:** pytest, jest, vitest, mocha, go test, cargo test, unittest
**Detect test structure:** tests/ directory, *.test.*, *.spec.*, __tests__/
**Detect coverage:** .coveragerc, jest.config coverage, nyc, c8
**Detect CI test commands:** GitHub Actions, GitLab CI test jobs
**Detect existing patterns:** fixtures, mocks, factories, recording providers
**Detect quality tools:** mutation testing config, property test libs (hypothesis, fast-check)
**Check mutation testing availability:** If mutation testing tools (mutmut, Stryker) are not installed, note their absence and provide install commands (`pip install mutmut` / `npm install -g @stryker-mutator/core`). Don't assume they're available.
**Verify import paths:** Before running tests, ensure the code under test is importable. For Python: check for `conftest.py` with `sys.path` setup, or run `pip install -e .` For JavaScript: check jest/vitest config paths and `moduleNameMapper` settings.

Present findings, ask: "What needs tests? Or say 'go' and I'll identify the gaps."

---

## Phase 1: Test Strategy Selection

| Situation | Strategy |
|-----------|----------|
| New feature | Write acceptance tests first (from spec), then unit tests during implementation |
| Bug fix | Write regression test that reproduces the bug FIRST, then fix |
| Existing untested code | Add property-based tests for core logic, integration tests for boundaries |
| Pure function library (no I/O, no APIs, no DB) | Skip integration tests. Focus on property-based + characterization tests. |
| Refactoring | Ensure existing tests pass, add characterization tests for untested paths |
| AI-generated code | Mutation test to verify tests aren't theater |

---

## Phase 2: Test Writing Workflow

### For New Features (TDD)

1. **Write failing test** — the test IS the spec
2. **Run it — confirm it fails** (red)
3. **Implement minimum code** to make it pass (green)
4. **Refactor** — clean up while tests stay green
5. **Git checkpoint** — `git commit -m "test: [what] + impl: [what]"`

CRITICAL: Use a SUBAGENT for implementation if possible. The test-writing context should not pollute implementation reasoning. If subagents are unavailable (single session), write ALL tests BEFORE reading the implementation code. This prevents test-implementation context bleed.

### For Bug Fixes

1. **Write regression test** that reproduces the exact bug
2. **Run it — confirm it fails** (proves the bug exists)
3. **Fix the bug** — minimum change
4. **Run test — confirm it passes** (proves the fix works)
5. **Run full suite** — confirm nothing else broke
6. **Git checkpoint** — `git commit -m "fix: [bug] + test: regression"`

### For Existing Code

1. **Identify untested critical paths** — use coverage reports
2. **Add property-based tests** for pure functions and data transforms
3. **Add characterization/known-value tests** — pin current behavior with concrete input/output pairs, even if behavior seems wrong. These serve as change detectors alongside property tests.
4. **Add integration tests** for system boundaries (APIs, DB, external services)
5. **Run mutation testing** to verify test effectiveness
6. **Add characterization tests** for complex legacy code (test current behavior, even if wrong)

---

## Phase 3: Advanced Testing Patterns

### Property-Based Testing
Instead of specific examples, define properties that must ALWAYS hold:

```python
# Python (Hypothesis)
@given(st.lists(st.integers()))
def test_sort_preserves_length(lst):
    assert len(sorted(lst)) == len(lst)

@given(st.lists(st.integers()))
def test_sort_is_ordered(lst):
    result = sorted(lst)
    assert all(result[i] <= result[i+1] for i in range(len(result)-1))
```

Finds edge cases AI never thinks of: empty lists, huge numbers, negative values, unicode.

**Constrain strategies for expensive functions:** For computationally expensive functions (recursive, exponential), limit input size to prevent timeouts:

```python
# BAD: unbounded input can cause timeouts on recursive/exponential functions
@given(st.integers())
def test_fibonacci(n): ...

# GOOD: constrained range prevents timeouts
@given(st.integers(min_value=0, max_value=30))
def test_fibonacci(n): ...

# GOOD: limit list size for O(n!) or O(2^n) algorithms
@given(st.lists(st.integers(), max_size=15))
def test_permutations(lst): ...
```

### Negative/Invalid Input Properties
Always test what happens with invalid inputs:

```python
@given(st.integers(max_value=-1))
def test_fibonacci_negative(n):
    # Define expected behavior: raise ValueError? Return 0? 
    # The test forces you to decide.
    with pytest.raises(ValueError):
        fibonacci(n)
```

If the function doesn't handle invalid inputs, the test exposes the gap. This is a design decision, not a bug — but the test makes it explicit.

### Mutation Testing
Verify tests actually catch defects:

```bash
# Python
mutmut run

# JavaScript
npx stryker run
```

> **mutmut v3 auto-detects paths.** If using v2, pass `--paths-to-mutate=src/`. Run `mutmut --version` to check.
>
> **After install, run a dry test** (`mutmut run --help`) to verify the tool works before relying on it.

If mutants survive (code changed but tests still pass), your tests are theater.

### Dependency-Graph Test Selection
Instead of running all tests, identify WHICH tests cover the changed code:

```bash
# Python (pytest-testmon)
pytest --testmon  # only runs tests affected by changes

# JavaScript (jest)
jest --changedSince=main  # only tests touching changed files
```

---

## Phase 4: Quality Gates

Set up hooks that enforce quality before code ships:

**Pre-commit:** Lint + type-check (fast, catches obvious issues)
**Pre-push:** Full test suite + coverage threshold
**CI:** Mutation testing + property tests + integration tests

### Coverage Thresholds

| Type | Minimum | Target |
|------|---------|--------|
| Line coverage | 70% | 85%+ |
| Branch coverage | 60% | 75%+ |
| Mutation score | 50% | 70%+ |

Don't chase 100% — diminishing returns. Focus mutation testing on critical paths.

---

## Phase 5: CI/CD Patterns

### Self-Healing Pipeline
When tests fail in CI, trigger a repair agent:

```yaml
- name: Run tests
  id: tests
  run: pytest -x -q
  continue-on-error: true

- name: Auto-fix if tests failed
  if: steps.tests.outcome == 'failure'
  run: |
    # Agent analyzes failure, proposes fix
    # Human reviews before merge
```

### Test Pyramid for AI Code

```
         /  E2E  \          ← Few, slow, high confidence
        / Integr.  \        ← Moderate, test boundaries
       /    Unit     \      ← Many, fast, test logic
      / Property-based \    ← Find edge cases AI misses
     /  Mutation tests   \  ← Prove tests work
```

---

## Anti-Patterns

1. **Same agent writes tests and code** — tests become tautological
2. **Only happy-path tests** — AI defaults to obvious cases. Use property tests.
3. **Mocking everything** — tests pass but production fails. Mock boundaries, not logic.
4. **100% coverage as goal** — coverage ≠ quality. Mutation score matters more.
5. **Skipping failing tests** — fix them or delete them. Never skip.
6. **Testing implementation details** — test behavior, not internals. Refactoring shouldn't break tests.
