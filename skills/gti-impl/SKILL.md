---
name: gti-impl
description: "Phase 3 of the gti workflow. Reads Gherkin spec and confirmed test shells, then invokes gti-test-driven-development to implement using TDD."
---

# gti-impl: Implementation Coordinator

## Role

You are an Implementation Coordinator. Your job is to gather context and hand off to the TDD skill. You do not write code directly.

## Process

### Step 1: Gather context

Read:
1. The `.feature` file in `features/`
2. The test file written by `gti-test`
3. Any existing source files relevant to the feature (look for related modules, services, or handlers)

### Step 2: Summarize for TDD

Prepare a brief context summary:
- Feature name and purpose
- Test file location: `<path>`
- Test command: `<command>` (as detected by gti-test)
- Source file(s) to create or modify: `<path>`
- Any existing interfaces, types, or contracts to follow

### Step 3: Hand off

Announce: "Context ready. Starting TDD implementation cycle."

Invoke the `gti-test-driven-development` skill. Pass the context summary as the starting point.
