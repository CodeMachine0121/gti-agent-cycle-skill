# gti Plugin Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build a Claude Code plugin named `gti` that enforces a closed-loop BDD→TDD workflow: spec → test shells (human review) → TDD implementation → verify.

**Architecture:** Pure Skills approach — 5 SKILL.md files + 1 command entry point. Each skill invokes the next via the Skill tool. No runtime state; flow is maintained in-context. The TDD discipline is extracted into a reusable `gti-test-driven-development` skill.

**Tech Stack:** Claude Code Plugin (Markdown SKILL.md files), Gherkin `.feature` files, language-agnostic (detects framework from project files at runtime).

---

## Reference Files

Before starting, read these files to understand SKILL.md format and conventions:

- `/Users/jameshsueh/.claude/plugins/cache/claude-plugins-official/superpowers/4.3.1/skills/brainstorming/SKILL.md` — example well-structured skill
- `/Users/jameshsueh/.claude/plugins/cache/claude-plugins-official/superpowers/4.3.1/commands/brainstorm.md` — example command
- `docs/plans/2026-02-25-gti-plugin-design.md` — approved design doc

---

### Task 1: Scaffold project structure

**Files:**
- Create: `README.md`
- Create: `skills/gti-spec/` (directory)
- Create: `skills/gti-test/` (directory)
- Create: `skills/gti-impl/` (directory)
- Create: `skills/gti-test-driven-development/` (directory)
- Create: `skills/gti-verify/` (directory)
- Create: `commands/` (directory)
- Create: `docs/` (directory, already exists)

**Step 1: Create directory structure**

```bash
mkdir -p skills/gti-spec
mkdir -p skills/gti-test
mkdir -p skills/gti-impl
mkdir -p skills/gti-test-driven-development
mkdir -p skills/gti-verify
mkdir -p commands
```

**Step 2: Create README.md**

```markdown
# gti

A Claude Code plugin that enforces a closed-loop BDD→TDD development workflow.

## Workflow

```
/gti → gti-spec → gti-test → [human review] → gti-impl → gti-test-driven-development ⟺ gti-verify
```

## Skills

| Skill | Role | Invokes |
|---|---|---|
| `gti-spec` | Write Gherkin from user requirement | `gti-test` |
| `gti-test` | Generate empty test shells | `gti-impl` (after human confirms) |
| `gti-impl` | Read context, coordinate TDD | `gti-test-driven-development` |
| `gti-test-driven-development` | Enforce RED→GREEN→REFACTOR | `gti-verify` |
| `gti-verify` | Run tests, report result | — |

## Testing Principles

See `docs/testing-principles.md`.

## Installation

```bash
claude plugin install .
```
```

**Step 3: Commit**

```bash
git init
git add README.md
git commit -m "chore: scaffold gti plugin structure"
```

---

### Task 2: Write `docs/testing-principles.md`

**Files:**
- Create: `docs/testing-principles.md`

**Step 1: Write the file**

```markdown
# Testing Principles

These principles apply to all test generation and TDD cycles in the gti workflow.

## Core Rules

### 1. Test Behavior, Not State

Assert on what the system *does*, not what it *contains*.

Bad: `assert user.name == "Alice"`
Good: `assert response.body contains greeting for "Alice"`

Do not assert on internal fields, intermediate variables, or implementation details that are invisible to the caller.

### 2. Runtime Exceptions Are Not Tested

A Runtime Exception (unchecked exception) represents a programming error — a violated contract. These do not need test coverage:

- Null pointer / nil dereference where non-null is required
- Index out of bounds
- Illegal argument (e.g. negative value for a count)
- Illegal state

Callers are not expected to handle these. Tests for them add noise without value.

### 3. Checked Exceptions Are Tested

A Checked Exception (or equivalent) represents a *predictable failure path* that callers must handle. These require explicit test coverage:

- File not found
- Network timeout
- Invalid user input that reaches a service boundary
- External API error

Write one test per meaningful failure scenario.

### 4. Quantity Does Not Equal Quality

A small, focused test suite is better than a large noisy one.

Do not write tests for:
- Every permutation of valid input
- Internal helper functions with no observable behavior
- Trivial getters/setters
- Things already covered by a broader test

Write tests for:
- Each distinct behavior
- Each Checked Exception path
- Edge cases that change the outcome

**If a test would never catch a real bug, don't write it.**
```

**Step 2: Commit**

```bash
git add docs/testing-principles.md
git commit -m "docs: add testing principles"
```

---

### Task 3: Write `skills/gti-spec/SKILL.md`

**Files:**
- Create: `skills/gti-spec/SKILL.md`

**Step 1: Write the skill**

```markdown
---
name: gti-spec
description: "Phase 1 of the gti workflow. Use when starting a new feature from a user requirement. Converts natural language requirements into a Gherkin .feature file."
---

# gti-spec: Requirement → Gherkin Spec

## Role

You are a Business Analyst. Your job is to translate the user's requirement into a precise Gherkin feature file that captures business behavior — not implementation.

## Process

### Step 1: Understand the requirement

If the requirement is ambiguous, ask ONE clarifying question at a time. Focus on:
- Who is the actor?
- What action are they taking?
- What is the observable outcome?

Do not ask about implementation details. Do not ask about edge cases until the happy path is clear.

### Step 2: Write the `.feature` file

Create a file at `features/<feature-name>.feature`.

Rules for writing Gherkin:
- `Given` sets up context (what is already true)
- `When` describes the action the actor takes
- `Then` describes the observable outcome (behavior, not state)
- One scenario per distinct behavior
- Scenario names are in business language, not technical language
- Do NOT reference function names, classes, database tables, or field names

Example of good Gherkin:
```gherkin
Feature: User login

  Scenario: Successful login with valid credentials
    Given a registered user with email "alice@example.com"
    When the user logs in with correct credentials
    Then the user is redirected to the dashboard

  Scenario: Login fails with wrong password
    Given a registered user with email "alice@example.com"
    When the user logs in with an incorrect password
    Then the user sees an error message
    And the user remains on the login page
```

### Step 3: Review

Read the written `.feature` file and verify:
- [ ] Every scenario has a single, clear business outcome in `Then`
- [ ] No implementation details leaked into steps
- [ ] Edge cases from the requirement are covered as separate scenarios

### Step 4: Hand off

Announce: "Gherkin spec written to `features/<name>.feature`. Moving to test generation."

Then invoke the `gti-test` skill.
```

**Step 2: Commit**

```bash
git add skills/gti-spec/SKILL.md
git commit -m "feat: add gti-spec skill"
```

---

### Task 4: Write `skills/gti-test/SKILL.md`

**Files:**
- Create: `skills/gti-test/SKILL.md`

**Step 1: Write the skill**

```markdown
---
name: gti-test
description: "Phase 2 of the gti workflow. Reads a Gherkin .feature file and generates empty test case shells. Requires human confirmation before proceeding to implementation."
---

# gti-test: Gherkin → Empty Test Shells

## Role

You are a Test Architect. Your job is to translate each Gherkin scenario into an empty test case shell — function signature and describe block only. No assertions. No implementation.

## Process

### Step 1: Detect the testing framework

Scan the project root for framework indicators in this order:

| File | Framework | Test command |
|---|---|---|
| `package.json` with `vitest` | Vitest | `npx vitest run` |
| `package.json` with `jest` | Jest | `npx jest` |
| `pom.xml` | JUnit | `mvn test` |
| `build.gradle` | JUnit | `./gradlew test` |
| `go.mod` | Go testing | `go test ./...` |
| `pyproject.toml` or `pytest.ini` | pytest | `pytest` |
| `Cargo.toml` | Rust tests | `cargo test` |

If none found, ask the user: "What test framework does this project use?"

### Step 2: Map Gherkin scenarios to test shells

Read `features/<name>.feature`. For each `Scenario`, create one empty test function.

Rules:
- Function name = scenario name in snake_case or camelCase (match project convention)
- Body = empty or single `// TODO` comment
- Do NOT write any assertions
- Do NOT write any implementation
- Group by `Feature` using describe/module blocks

Example mapping (TypeScript/Vitest):
```typescript
// Given: features/login.feature

describe("User login", () => {
  it("successful login with valid credentials", () => {
    // TODO
  });

  it("login fails with wrong password", () => {
    // TODO
  });
});
```

Example mapping (Go):
```go
func TestUserLogin_SuccessfulLoginWithValidCredentials(t *testing.T) {
  // TODO
}

func TestUserLogin_LoginFailsWithWrongPassword(t *testing.T) {
  // TODO
}
```

### Step 3: Write the test file

Write the test shells to the appropriate location for the detected framework:

| Framework | Location |
|---|---|
| Jest/Vitest | `src/__tests__/<name>.test.ts` or beside source |
| JUnit | `src/test/java/<package>/<Name>Test.java` |
| Go | `<package>/<name>_test.go` |
| pytest | `tests/test_<name>.py` |

### Step 4: HARD STOP — Human confirmation required

Present the generated test file to the user and say:

"Test shells written to `<path>`. Please review:
- Are all scenarios represented?
- Are any scenarios missing or should be split?
- Do the test names clearly describe the expected behavior?

Reply 'ok' or 'approved' to continue, or provide feedback."

Wait for explicit approval. Do NOT proceed until confirmed.

### Step 5: Hand off

After confirmation, announce: "Test shells confirmed. Moving to implementation."

Then invoke the `gti-impl` skill.
```

**Step 2: Commit**

```bash
git add skills/gti-test/SKILL.md
git commit -m "feat: add gti-test skill"
```

---

### Task 5: Write `skills/gti-impl/SKILL.md`

**Files:**
- Create: `skills/gti-impl/SKILL.md`

**Step 1: Write the skill**

```markdown
---
name: gti-impl
description: "Phase 3 of the gti workflow. Reads Gherkin spec and confirmed test shells, then invokes gti-test-driven-development to implement using TDD."
---

# gti-impl: Implementation Coordinator

## Role

You are an Implementation Coordinator. Your job is to gather context and hand off to the TDD skill. You do not write code directly.

## Process

### Step 1: Gather context

Read:
1. The `.feature` file in `features/`
2. The test file written by `gti-test`
3. Any existing source files relevant to the feature (look for related modules, services, or handlers)

### Step 2: Summarize for TDD

Prepare a brief context summary:
- Feature name and purpose
- Test file location: `<path>`
- Test command: `<command>` (as detected by gti-test)
- Source file(s) to create or modify: `<path>`
- Any existing interfaces, types, or contracts to follow

### Step 3: Hand off

Announce: "Context ready. Starting TDD implementation cycle."

Invoke the `gti-test-driven-development` skill. Pass the context summary as the starting point.
```

**Step 2: Commit**

```bash
git add skills/gti-impl/SKILL.md
git commit -m "feat: add gti-impl skill"
```

---

### Task 6: Write `skills/gti-test-driven-development/SKILL.md`

This is the most important skill. Read `docs/testing-principles.md` before writing it.

**Files:**
- Create: `skills/gti-test-driven-development/SKILL.md`

**Step 1: Write the skill**

```markdown
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
```

**Step 2: Commit**

```bash
git add skills/gti-test-driven-development/SKILL.md
git commit -m "feat: add gti-test-driven-development skill"
```

---

### Task 7: Write `skills/gti-verify/SKILL.md`

**Files:**
- Create: `skills/gti-verify/SKILL.md`

**Step 1: Write the skill**

```markdown
---
name: gti-verify
description: "Phase 4 of the gti workflow. Runs the project's test suite and reports results. Called by gti-test-driven-development after each GREEN and REFACTOR phase."
---

# gti-verify: Test Execution Gate

## Role

You are a QA Gate. You run tests and report results truthfully. You do not interpret or fix failures — you report them and hand back to the caller.

## Process

### Step 1: Determine scope

You will be called with one of two scopes:

- **Single test:** Run only the specified test (passed by gti-test-driven-development during GREEN phase)
- **Full suite:** Run all tests (called after REFACTOR or at workflow completion)

### Step 2: Run the tests

Use the test command detected during `gti-test`. If not available in context, re-detect:

| Project file | Command |
|---|---|
| `package.json` + vitest | `npx vitest run` |
| `package.json` + jest | `npx jest` |
| `pom.xml` | `mvn test` |
| `build.gradle` | `./gradlew test` |
| `go.mod` | `go test ./...` |
| `pyproject.toml` / `pytest.ini` | `pytest` |
| `Cargo.toml` | `cargo test` |

For single test scope, append the test filter flag appropriate for the framework.

### Step 3: Report results

**All tests pass:**

```
✓ All tests passing ([N] tests)
```

If called from the final verification step, announce:
"Workflow complete. All tests green."

**Failures detected:**

List each failing test with:
- Test name
- File and line number
- Actual vs expected output (from test runner output)

Then return to `gti-test-driven-development` with this failure report.

Do NOT suggest fixes. Do NOT modify code. Report only.
```

**Step 2: Commit**

```bash
git add skills/gti-verify/SKILL.md
git commit -m "feat: add gti-verify skill"
```

---

### Task 8: Write `commands/gti.md`

**Files:**
- Create: `commands/gti.md`

**Step 1: Write the command**

```markdown
---
description: "Start the gti BDD→TDD workflow. Converts a user requirement into a Gherkin spec, generates test shells for review, then implements using TDD until all tests pass."
---

Invoke the `gti-spec` skill and follow it exactly.
```

**Step 2: Commit**

```bash
git add commands/gti.md
git commit -m "feat: add /gti command entry point"
```

---

### Task 9: Verify plugin structure is complete

**Step 1: Check all files exist**

```bash
find . -name "SKILL.md" -o -name "*.md" | grep -v node_modules | sort
```

Expected output should include:
```
./README.md
./commands/gti.md
./docs/plans/2026-02-25-gti-plugin-design.md
./docs/plans/2026-02-25-gti-plugin-impl.md
./docs/testing-principles.md
./skills/gti-impl/SKILL.md
./skills/gti-spec/SKILL.md
./skills/gti-test-driven-development/SKILL.md
./skills/gti-test/SKILL.md
./skills/gti-verify/SKILL.md
```

**Step 2: Verify SKILL.md frontmatter**

Each `SKILL.md` must have `name:` and `description:` in frontmatter. Run:

```bash
grep -l "^---" skills/*/SKILL.md
```

All 5 skill files should be listed.

**Step 3: Final commit**

```bash
git add -A
git commit -m "chore: verify complete plugin structure"
```
