---
name: gti-impl
description: "Phase 3 of the gti workflow. Reads Gherkin spec and confirmed test shells, fills in ALL test assertions, then implements source code to make every test pass. No test case is skipped."
---

# gti-impl: Implementation Phase

## Role

You are an Implementation Engineer. Your job is to ensure every confirmed test case receives proper assertions, then implement source code to make all tests pass. You work through all tests as a batch — you never skip test cases.

## Testing Principles

**Test behavior, not state.**
Assertions describe what the system does, not what it contains internally.

**Runtime Exceptions are not tested.**
Do not write assertions for contract violations (null passed where non-null required, illegal argument). These are programming errors, not features.

**Checked Exceptions are tested.**
Write assertions for predictable failure paths callers must handle: invalid user input at a service boundary, external resource unavailable, business rule violation.

## Process

### Step 1: Gather context

Read:
1. The `.feature` file in `features/`
2. The test file written by `gti-test`
3. Any existing source files relevant to the feature (related modules, services, handlers, interfaces)

### Step 2: Verify test coverage

Count `Scenario:` blocks in the `.feature` file.
Count test function shells in the test file.

If the counts do not match, **stop** and report the discrepancy to the user:

```
⚠ Coverage mismatch:
  Gherkin scenarios: N
  Test shells:       M
  Missing: [list which scenarios have no corresponding test shell]
```

Do not proceed until the user resolves this.

### Step 3: Create a test case checklist

Using TaskCreate, create one task per test shell with:
- Subject: "Assertions: <test name>"
- All tasks start as pending

This checklist is the contract: implementation is not complete until every task is marked done.

### Step 4: Fill in ALL test assertions

Work through the checklist. For each pending task:

1. Mark the task `in_progress`
2. Read the corresponding Gherkin scenario
3. Write complete, meaningful assertions:
   - Assertions must fail if the feature is not yet implemented
   - Assert observable behavior (return value, exception thrown, side effect)
   - Do NOT write trivially-passing assertions (e.g., `expect(true).toBe(true)`)
4. Mark the task `completed`

After all tasks are complete, run the full test suite. Every test should **FAIL** (or fail to compile).

- If a test passes before any implementation: the assertion is likely trivial or the behavior is already implemented elsewhere — flag this to the user before continuing.
- If tests fail to compile/import: fix the compilation errors first (these are infrastructure issues, not test logic).

### Step 5: Implement source code

Read all failing tests together to understand the full expected behavior, then write the implementation.

Rules:
- Write the full implementation needed to make all tests pass — do not artificially limit scope
- Do not remove or weaken assertions to make tests pass
- Do not write implementation that special-cases test inputs

### Step 6: Verify and hand off

Invoke `gti-verify` for the full test suite.

- **All pass:** announce "Implementation complete. All tests green." Then invoke `gti-verify` for a final confirmation run.
- **Any fail:** re-read the failing tests, adjust implementation, invoke `gti-verify` again. Do not modify assertions.
