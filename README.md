# gti-agent-cycle-skill

**GTI** = Gherkin → Test → Implementation

A Claude Code plugin that enforces a closed-loop BDD→TDD development workflow — from natural language requirement to verified, passing tests — with full documentation output at every stage.

## How It Works

You describe a feature. `gti` guides you through the entire development cycle:

1. **Setup + Spec** — creates an isolated git worktree, brainstorms edge cases and failure paths with you, then writes a Gherkin `.feature` file and a `spec.md` document. **You review and confirm the spec before anything else proceeds.**
2. **Generate test shells** — empty test cases mapped from each scenario, generated automatically after spec approval
3. **Implement** — follows your project's conventions (CLAUDE.md, AGENT.md, coding style), fills all assertions first (every test RED), then implements source code to make them all GREEN — no test case is ever skipped
4. **Final verification** — runs the full test suite, checks no empty shells remain, cross-checks test coverage against all feature scenarios, and produces a `conclusion.md` document

```
/gti → gti-spec → [human review] → gti-test → gti-impl → gti-verify
```

## Installation

Install directly from GitHub in Claude Code:

```
/plugin install https://github.com/CodeMachine0121/gti-agent-cycle-skill.git```

## Usage

In any project, run:

```
/gti
```

Claude will ask for your feature requirement and walk through the full workflow automatically. The only time you need to intervene is to review and approve the spec before implementation begins.

## Skills

| Skill | Role | Invokes |
|---|---|---|
| `gti-spec` | Setup worktree, brainstorm requirements → write `.feature` + `spec.md`, wait for human approval | `gti-test` |
| `gti-test` | Generate empty test shells from feature file | `gti-impl` |
| `gti-impl` | Follow project conventions, fill all assertions → implement source code | `gti-verify` |
| `gti-verify` | Run full suite, verify coverage, generate `conclusion.md` | — |

## Output Structure

Each feature produces a documentation folder at `docs/<feature-name>/`:

```
docs/<feature-name>/
  <feature-name>.feature   # Gherkin scenarios (business language)
  spec.md                  # Requirements summary + happy cases
  conclusion.md            # Generated after successful verification
```

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
