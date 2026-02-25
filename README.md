# gti-agent-cycle-skill

**GTI** = Gherkin → Test → Implementation

A Claude Code plugin that enforces a closed-loop BDD→TDD development workflow — from natural language requirement to verified, passing tests.

## How It Works

You describe a feature. `gti` guides you through the entire development cycle:

1. **Explore requirements** — Claude brainstorms edge cases and failure paths with you, ensuring nothing is missed before a single line of code is written
2. **Write Gherkin spec** — requirements become a `.feature` file in business language
3. **Generate test shells** — empty test cases mapped from each scenario, **you review and confirm before implementation starts**
4. **TDD implementation** — strict RED→GREEN→REFACTOR cycle until all tests pass
5. **Verify** — automated test execution closes the loop

```
/gti → gti-spec → gti-test → [human review] → gti-impl → gti-test-driven-development ⟺ gti-verify
```

## Installation

Install directly from GitHub in Claude Code:

```
/plugin install github:CodeMachine0121/gti-agent-cycle-skill
```

## Usage

In any project, run:

```
/gti
```

Claude will ask for your feature requirement and walk through the full workflow automatically. The only time you need to intervene is to confirm the generated test cases before implementation begins.

## Skills

| Skill | Role | Invokes |
|---|---|---|
| `gti-spec` | Brainstorm requirements → write Gherkin | `gti-test` |
| `gti-test` | Generate empty test shells (human confirmation required) | `gti-impl` |
| `gti-impl` | Gather context, coordinate TDD | `gti-test-driven-development` |
| `gti-test-driven-development` | Enforce RED→GREEN→REFACTOR cycle | `gti-verify` |
| `gti-verify` | Run tests, report pass/fail | — |

## Testing Principles

Built-in discipline enforced throughout the workflow:

- **Test behavior, not state** — assert on what the system does, not internal fields
- **Runtime Exceptions are not tested** — contract violations (null, illegal argument) are bugs, not features
- **Checked Exceptions are tested** — predictable failure paths that callers must handle
- **Quantity ≠ quality** — a small focused test suite beats a noisy large one

See [`docs/testing-principles.md`](docs/testing-principles.md) for full details.

## Language Support

Language-agnostic. `gti` detects your project's test framework automatically:

| Framework | Detected from |
|---|---|
| Vitest / Jest | `package.json` |
| JUnit | `pom.xml` / `build.gradle` |
| pytest | `pyproject.toml` / `pytest.ini` |
| Go testing | `go.mod` |
| Rust | `Cargo.toml` |

## Author

james.hsueh — [asdfg55887@gmail.com](mailto:asdfg55887@gmail.com)
