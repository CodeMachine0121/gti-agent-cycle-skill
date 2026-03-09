---
name: gti-test
description: "Phase 2 of the gti workflow. Reads a Gherkin .feature file from docs/<feature_name>/ and generates empty test case shells. Human confirmation is handled in gti-spec; this phase proceeds directly to implementation."
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
One scenario = one test shell. Do not split a single scenario into multiple shells or add extra shells "for coverage." If a Gherkin scenario is too broad (covers multiple behaviors), flag it to the user instead of multiplying shells.

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

### Step 1.5: Read context

Read the following files to understand the feature's requirements and scenarios:

- `docs/<feature_name>/spec.md` — requirements context for the feature
- `docs/<feature_name>/<feature_name>.feature` — Gherkin scenarios to translate into test shells

Note: feature files live under `docs/<feature_name>/`, not `features/`.

### Step 2: Map Gherkin scenarios to test shells

Read `docs/<feature_name>/<feature_name>.feature`. For each `Scenario`, create one empty test function.

Rules:
- Function name = scenario name in snake_case or camelCase (match project convention)
- Body = empty or single `// TODO` comment
- Do NOT write any assertions
- Do NOT write any implementation
- Group by `Feature` using describe/module blocks

Example mapping (TypeScript/Vitest):
```typescript
// Given: docs/login/login.feature

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

Use the language-appropriate naming convention and placement. Follow the project's existing structure if test files already exist — match their pattern.

**TypeScript / JavaScript (Vitest or Jest)**
- Name: `<source-file-name>.test.ts` or `<source-file-name>.spec.ts`
- Vitest (preferred): place beside the source file — `src/feature/login.test.ts`
- Jest: place beside source or in `src/__tests__/login.test.ts`
- Use `kebab-case` filenames matching the source file name
- Group with `describe` for Feature, `it` for Scenario

**Go**
- Name: `<source-file>_test.go` — always beside the source file in the same directory
- Black-box (integration-style): `package <name>_test` — preferred when testing public API
- White-box (access to unexported symbols): `package <name>`
- Function names: `Test<FeatureName>_<ScenarioName>` in PascalCase

**Python (pytest)**
- Name: `test_<module>.py`
- Place in `tests/` directory mirroring source structure: `tests/unit/test_login.py`
- Function prefix: `test_`, class prefix: `Test` (no suffix needed)
- Use `class Test<Feature>:` to group related scenarios

**Java (JUnit 5)**
- Name: `<ClassName>Test.java`
- Mirror source structure under `src/test/java/`: `src/test/java/com/example/LoginTest.java`
- Use exact same package as the source class
- Use `@Nested` classes to group scenarios by Feature

**Rust**
- Unit tests (testing internal module behavior): inline in source file using `#[cfg(test)]` module at the bottom
- Integration tests (testing public API): `tests/<name>.rs`
- Function prefix: `test_` with `#[test]` attribute

**Ruby (RSpec)**
- Name: `<name>_spec.rb`
- Mirror source structure under `spec/`: `spec/services/login_spec.rb`
- Use `describe` for the class/module, `context` for scenarios

### Step 4: Hand off

After writing the test file, announce: "Test shells generated. Moving to implementation."

Then invoke the `gti-impl` skill directly.
