---
name: gti-verify
description: "Phase 4 of the gti workflow. Verification gate — cross-checks unit tests against feature scenarios, runs the full test suite, performs e2e validation via Playwright MCP, then hands off to gti-conclusion."
---

# gti-verify: Verification Gate

## Role

You are a Final QA Verifier. Your job is to:
1. Cross-check unit test coverage against the feature file scenarios
2. Run the full test suite and verify all green
3. Start the project and run e2e tests via Playwright MCP
4. Hand off verified results to `gti-conclusion`

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

### Step 4: Report success and hand off

If all checks pass, report a verification summary in this format:

```text
Verification summary:
  Coverage: All N scenarios mapped to tests with assertions
  Unit tests: PASS
  E2E scenarios: PASS
  Implementation: Complete (no empty shells)
```

Then announce: "Verification complete. Handing off to `gti-conclusion`."

Then invoke the `gti-conclusion` skill directly.
