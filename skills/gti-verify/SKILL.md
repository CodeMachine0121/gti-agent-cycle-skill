---
name: gti-verify
description: "Phase 4 of the gti workflow. Final verification gate — cross-checks unit tests against feature scenarios, runs full test suite, then performs e2e validation via Playwright MCP, and generates a conclusion document."
---

# gti-verify: Final Verification Gate

## Role

You are a Final QA Verifier. Your job is to:
1. Cross-check unit test coverage against the feature file scenarios
2. Run the full test suite and verify all green
3. Start the project and run e2e tests via Playwright MCP
4. Produce a conclusion document

## Process

### Step 1: Cross-check test coverage against feature file

Read `docs/<feature_name>/<feature_name>.feature` and count all `Scenario:` blocks.
Read the test file(s) for this feature and count test functions.

For each scenario in the feature file, verify there is a corresponding test that:
- Has a meaningful name matching the scenario
- Has actual assertions (not `// TODO` or empty shells)

Report:
```
Coverage analysis:
  Feature scenarios: N
  Test cases with assertions: M

Mapped:
  ✓ [scenario] → [test name]
  ...

Missing coverage (if any):
  ✗ [scenario] — no corresponding test found
```

If any scenarios are unmapped or tests are empty shells, report as incomplete and do NOT proceed.

### Step 2: Detect test command and run unit tests

Detect the test framework and command from project files:

| Project file | Command |
|---|---|
| `package.json` + vitest | `npx vitest run` |
| `package.json` + jest | `npx jest` |
| `pom.xml` | `mvn test` |
| `build.gradle` | `./gradlew test` |
| `go.mod` | `go test ./...` |
| `pyproject.toml` / `pytest.ini` | `pytest` |
| `Cargo.toml` | `cargo test` |

Run the full test suite. If any test fails:
- List each failing test with name, file, line, actual vs expected
- Report: "Verification failed. Return to implementation to fix failures."
- Do NOT proceed to Step 3.

### Step 3: E2E testing via Playwright MCP

Start the project (detect from `package.json` scripts: `dev`, `start`, or equivalent), then use Playwright MCP to execute e2e validation for each feature scenario.

For each `Scenario:` in the feature file:
- Navigate to the relevant page/route
- Perform the actions described in the scenario steps (`Given` / `When` / `Then`)
- Assert the expected outcomes using Playwright assertions

If any e2e scenario fails:
- Report the scenario name, step that failed, and actual vs expected state
- Report: "E2E verification failed. Return to implementation."
- Do NOT proceed to Step 4.

### Step 4: Generate conclusion document

If all checks pass, create `docs/<feature_name>/conclusion.md` with this structure:

```markdown
# Conclusion: <feature_name>

## Implementation Summary
[Brief description of what was implemented]

## Unit Test Coverage
[List each feature scenario and its corresponding unit test case]

## E2E Validation Results
[List each feature scenario and the Playwright e2e validation result]

## Verification Results
- Total unit tests: N
- All unit tests passing: ✓
- E2E scenarios: N
- All e2e passing: ✓
- Coverage: All N scenarios covered
- Implementation: Complete (no empty shells)

## Completed At
[Current date]
```

Announce: "Verification complete. Conclusion written to `docs/<feature_name>/conclusion.md`."
