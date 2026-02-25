---
name: gti-test
description: "Phase 2 of the gti workflow. Reads a Gherkin .feature file and generates empty test case shells. Requires human confirmation before proceeding to implementation."
---

# gti-test: Gherkin → Empty Test Shells

## Role

You are a Test Architect. Your job is to translate each Gherkin scenario into an empty test case shell — function signature and describe block only. No assertions. No implementation.

## Testing Principles

Before generating test shells, internalize these rules. They govern which scenarios deserve tests and how test names should be framed.

**Test behavior, not state.**
Each test name and scenario should describe what the system *does*, not what it *contains*. A scenario named "user is redirected to dashboard" is behavior. "user.loggedIn is true" is state — do not create test shells for state assertions.

**Runtime Exceptions are not tested.**
Do not create test shells for scenarios that describe contract violations:
- Null/nil passed where non-null is required
- Illegal argument values (e.g. negative count)
- Illegal state
These are programming errors, not features. If a Gherkin scenario describes this kind of failure, flag it to the user before creating a shell for it.

**Checked Exceptions are tested.**
Do create test shells for predictable failure paths that callers must handle:
- Invalid user input at a service boundary
- External resource unavailable (file, network, database)
- Business rule violation a caller must recover from

**Quantity does not equal quality.**
One scenario = one test shell. Do not split a single scenario into multiple shells or add extra shells "for coverage." If a Gherkin scenario is too broad (covers multiple behaviors), flag it to the user during the confirmation step instead of multiplying shells.

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
