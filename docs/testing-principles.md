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
