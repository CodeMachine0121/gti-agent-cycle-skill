# gti-agent-cycle-skill

**GTI** = Gherkin → Test → Implementation

A Claude Code plugin that enforces a closed-loop BDD→TDD development workflow — from natural language requirement to verified, passing tests — with full documentation output at every stage.

## How It Works

You describe a feature. `gti` guides you through the entire development cycle:

1. **Setup + Spec** (`gti-spec`) — asks for the feature name, then sets up the environment:
   - Creates or reuses a **git worktree** (isolated copy of the repo), or works in the main project
   - Creates or reuses a **feature branch**
   - Brainstorms edge cases, alternative flows, and failure paths with you
   - Writes a Gherkin `.feature` file and a `spec.md` document
   - **You review and confirm the spec before anything else proceeds.**

2. **Generate test shells** (`gti-test`) — maps each Gherkin scenario to an empty test function. Proceeds automatically after spec approval. No second human checkpoint.

3. **Implement** (`gti-impl`) — follows your project's conventions (`CLAUDE.md`, `AGENT.md`, coding style):
   - Fills in ALL test assertions first (every test **RED**)
   - Implements source code to make all tests **GREEN** — no test is ever skipped
   - Enforces **James Refactor Principles**: zero private methods, method ≤ 5 responsibilities, pure domain/service layer, fluent business logic, immutable variables
   - Test helpers follow naming conventions: `Given`-prefix for mock stubs, `Create`-prefix for object construction

4. **Final verification** (`gti-verify`) — three gates must all pass before completion:
   - **Coverage cross-check**: each Gherkin scenario maps to a test with real assertions
   - **Full test suite**: every unit test passes
   - **E2E via Playwright MCP**: each scenario is validated in the running application
   - Produces a `conclusion.md` document only when all three gates pass

```
/gti → gti-spec → [human review] → gti-test → gti-impl → gti-verify
```

## Installation

Install directly from GitHub in Claude Code:

```
/plugin install https://github.com/CodeMachine0121/gti-agent-cycle-skill.git
```

## Usage

In any project, run:

```
/gti
```

Claude will ask for your feature requirement and walk through the full workflow automatically. The only human checkpoint is reviewing and approving the spec before implementation begins.

## Skills

| Skill | Role | Invokes |
|---|---|---|
| `gti-spec` | Set up worktree + branch, brainstorm requirements → write `.feature` + `spec.md`, wait for human approval | `gti-test` |
| `gti-test` | Detect test framework, generate empty test shells from feature file | `gti-impl` |
| `gti-impl` | Follow project conventions, fill all assertions (RED) → implement source code (GREEN) | `gti-verify` |
| `gti-verify` | Cross-check coverage → run full suite → e2e via Playwright MCP → generate `conclusion.md` | — |

Each skill is independently invocable if you need to re-run a single phase.

## Output Structure

Each feature produces a documentation folder at `docs/<feature-name>/`:

```
docs/<feature-name>/
  <feature-name>.feature   # Gherkin scenarios (business language)
  spec.md                  # Requirements summary + happy cases
  conclusion.md            # Generated after all three verification gates pass
```

A `playwright-mcp.json` is provided in the `mcps/` folder for configuring the Playwright MCP server:

```bash
claude mcp add --config mcps/playwright-mcp.json
```

## Implementation Style

All source code produced by `gti-impl` follows the **James Refactor Principles**:

| Rule | Detail |
|---|---|
| Zero private methods | Extract to a collaborator class or inline |
| Method ≤ 5 responsibilities | Count things done, not lines |
| > 2 parameters | Extract a DTO / record |
| Complex pre-conditions | Decorator pattern |
| Conditional branch implementations | Strategy pattern |
| Service / Domain layer | Pure code — no framework imports |
| Business logic | Fluent method chaining |
| Variable assignment | Declare-and-define; no post-declaration mutation |

Test helpers follow consistent naming conventions:
- `GivenXxx()` — sets up a mock return value
- `CreateXxx()` — constructs a domain object

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
