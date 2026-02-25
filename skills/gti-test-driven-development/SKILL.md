---
name: gti-test-driven-development
description: "TDD discipline enforcer. Implements the RED→GREEN→REFACTOR cycle for a given test file. Called by gti-impl. Can also be invoked directly for TDD on any existing test file."
---

# gti-test-driven-development: RED → GREEN → REFACTOR

## Role

You are a TDD practitioner. You enforce strict discipline: never write implementation before a failing test. Never refactor on red. One cycle at a time.

## Testing Principles

Before writing any test or code, internalize these rules:

**Test behavior, not state.**
Assert on what the system does, not what it contains internally. Do not assert on field values, intermediate variables, or implementation details invisible to the caller.

**Runtime Exceptions are not tested.**
Do not write tests for:
- Contract violations (null passed where non-null required, illegal argument)
- Programming errors that only appear when the code is wrong
These are Runtime Exceptions (unchecked). They indicate bugs, not features.

**Checked Exceptions are tested.**
Write tests for predictable failure paths that callers must handle:
- Invalid user input at a service boundary
- External resource unavailable (file, network, database)
- Business rule violation that a caller must recover from

**Quantity does not equal quality.**
Write one test per distinct behavior. Do not write tests for:
- Every valid input permutation
- Trivial wrappers or delegations
- Things already proven by another test

If a test would never catch a real bug, do not write it.

## The TDD Cycle

Repeat this cycle for each failing test:

---

### Phase 1: RED

**Goal:** Confirm the test fails for the right reason.

1. Identify the next unimplemented test in the test file
2. Run ONLY that test:
   - Jest/Vitest: `npx vitest run --reporter=verbose -t "test name"`
   - Go: `go test -run TestName ./...`
   - JUnit: `mvn test -Dtest=ClassName#methodName`
   - pytest: `pytest tests/test_file.py::test_name -v`
3. Verify the output shows FAIL with a meaningful error (not a syntax error or import error)
4. If it passes already: the test shell was not empty — investigate before continuing
5. If it errors on import/compile: fix the import/compile error first (this is infrastructure, not TDD)

Do NOT proceed to GREEN until you see a genuine test failure.

---

### Phase 2: GREEN

**Goal:** Make this one test pass with the minimum possible code.

Rules:
- Write only what is needed to make THIS test pass
- Do not implement logic for other tests yet
- Do not add error handling not required by this test
- Do not refactor
- Hardcoding the return value is acceptable if it makes the test pass

After writing the implementation, invoke `gti-verify` for this test only.

If `gti-verify` reports PASS: move to REFACTOR.
If `gti-verify` reports FAIL: re-read the test, adjust implementation, invoke `gti-verify` again.

---

### Phase 3: REFACTOR

**Goal:** Improve the code without changing behavior.

Only refactor when the test is green. Rules:
- Extract duplication only if it appears 3+ times
- Rename for clarity
- Remove dead code
- Do NOT add new behavior during refactor

After refactoring, invoke `gti-verify` for ALL tests written so far.

If all pass: proceed to the next failing test (back to RED).
If any fail: the refactor broke something — revert and try a smaller refactor.

---

## Completion

When all tests in the file pass, announce:

"All tests passing. TDD cycle complete. Invoking gti-verify for final confirmation."

Invoke `gti-verify` for the full test suite.
