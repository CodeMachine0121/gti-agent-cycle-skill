# gti Plugin Design

Date: 2026-02-25

## Overview

`gti` is a Claude Code plugin that implements a closed-loop agent workflow for behavior-driven, test-driven development. The workflow automates the path from user requirement to verified implementation while enforcing testing discipline throughout.

## Goals

- Convert user requirements into Gherkin specs automatically
- Generate test case shells for human review before any implementation begins
- Enforce strict RED→GREEN→REFACTOR TDD discipline during implementation
- Close the loop: verify → fix → verify until all tests pass
- Be language-agnostic by detecting the project's testing framework at runtime

## Architecture

**Approach:** Pure Skills (Option A) — each phase is an independent skill that invokes the next. No external runtime or state file required. Skills can be invoked individually or as a full pipeline via the `/gti` command.

## Project Structure

```
gti-skill/
├── skills/
│   ├── gti-spec/SKILL.md
│   ├── gti-test/SKILL.md
│   ├── gti-impl/SKILL.md
│   ├── gti-test-driven-development/SKILL.md
│   └── gti-verify/SKILL.md
├── commands/
│   └── gti.md
├── docs/
│   └── testing-principles.md
└── README.md
```

## Skills

### `gti-spec`

**Role:** PM / Analyst

- Receives the user's feature request in natural language
- Produces a `.feature` file written in Gherkin (Given/When/Then)
- Scenarios must express business behavior, not implementation details
- After writing the feature file, invokes `gti-test`

### `gti-test`

**Role:** Test Architect

- Scans the current project to detect the testing framework (package.json → Jest/Vitest, pom.xml → JUnit, go.mod → testing, etc.)
- Reads the `.feature` file and maps each scenario to an empty test case shell (function signature + describe block only — no implementation, no assertions)
- **Hard stop:** presents the test shells to the user and waits for explicit confirmation
- After confirmation, invokes `gti-impl`

### `gti-impl`

**Role:** Implementation Coordinator

- Reads the Gherkin file and confirmed test shells for context
- Invokes `gti-test-driven-development` to execute the TDD cycle

### `gti-test-driven-development`

**Role:** TDD Discipline Enforcer

Enforces the RED → GREEN → REFACTOR cycle strictly:

1. **RED** — Run tests first. Confirm they fail. Do NOT write implementation until failure is confirmed.
2. **GREEN** — Write the minimum code required to make the failing test pass. Nothing more.
3. **REFACTOR** — Only after green. Clean up code. Then invoke `gti-verify` to confirm tests still pass.
4. **Loop** — If more failing tests remain, repeat from RED. When all tests pass, complete.

Applies the testing principles documented below throughout the cycle.

### `gti-verify`

**Role:** QA Gate

- Detects and runs the appropriate test command for the project
- All green → declares the workflow complete
- Any failure → reports failing tests back to `gti-test-driven-development` to continue the fix cycle

## Commands

### `/gti`

Entry point for the full workflow. Invokes `gti-spec` to begin the pipeline.

## Testing Principles

These principles are embedded in `gti-test` and `gti-test-driven-development`:

1. **Test behavior, not state** — assert on outcomes and side effects, not on internal state or implementation details
2. **Runtime Exceptions are not tested** — errors thrown due to contract violations (null where non-null expected, illegal arguments, etc.) do not need test coverage
3. **Checked Exceptions are tested** — predictable failure paths that callers must handle require explicit test coverage
4. **Quantity ≠ quality** — do not write tests for every possible permutation; write tests for meaningful, distinct behaviors. A smaller focused test suite is better than a large noisy one.

## Flow Diagram

```
User request
     │
  gti-spec
     │ produces .feature file
  gti-test
     │ produces empty test shells
     │ [PAUSE] user confirms test cases
  gti-impl
     │ invokes
  gti-test-driven-development
     │
     ├─ RED: run tests → confirm failure
     ├─ GREEN: write minimal impl
     ├─ REFACTOR: clean up
     └─ gti-verify
          ├─ all pass → DONE
          └─ failures → back to gti-test-driven-development
```

## Key Decisions

| Decision | Choice | Reason |
|---|---|---|
| Architecture | Pure Skills | Matches existing plugin conventions; each skill independently invocable |
| State management | None (in-context) | Resume scenarios are rare; simplicity wins |
| Language support | Language-agnostic | Detect framework from project files at runtime |
| TDD enforcement | Separate skill | Reusable, independently invocable, clear single responsibility |
| Human checkpoints | After gti-test only | Spec is machine-generated; impl details are Claude's job |
