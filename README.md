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
