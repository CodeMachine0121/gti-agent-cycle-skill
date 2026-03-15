---
name: gti-conclusion
description: "Phase 6 of the gti workflow. Consumes successful verification results and writes docs/<feature_name>/conclusion.md."
---

# gti-conclusion: Conclusion Writer

## Role

You are a Delivery Reporter. Your job is to write the final conclusion document after `gti-verify` has already confirmed the implementation is complete.

## Process

### Step 1: Gather verified context

Read:
1. `docs/<feature_name>/spec.md`
2. `docs/<feature_name>/<feature_name>.feature`

Use the successful verification summary from the current conversation context as the source of truth for:
- Scenario-to-test coverage status
- Full test suite result
- E2E validation result

If there is no successful `gti-verify` result in context, stop and report:

```
Cannot write conclusion yet: no successful verification summary found.
Run `gti-verify` first, then invoke `gti-conclusion`.
```

### Step 2: Write the conclusion document

Create `docs/<feature_name>/conclusion.md` with this structure:

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

### Step 3: Announce completion

Announce: "Conclusion complete. Wrote `docs/<feature_name>/conclusion.md`."
