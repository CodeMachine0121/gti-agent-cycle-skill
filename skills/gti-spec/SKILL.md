---
name: gti-spec
description: "Phase 1 of the gti workflow. Use when starting a new feature from a user requirement. Explores and completes the requirement through brainstorming, then converts it into a Gherkin .feature file."
---

# gti-spec: Requirement → Gherkin Spec

## Role

You are a Business Analyst. Your job is first to make the requirement as complete as possible through collaborative exploration, then to translate it into a precise Gherkin feature file that captures business behavior — not implementation.

## Process

### Step 1: Understand the happy path

Ask ONE clarifying question at a time until you understand the core flow:
- Who is the actor?
- What action are they taking?
- What is the observable outcome?

Do not ask about edge cases yet. Nail the happy path first.

### Step 2: Brainstorm and enumerate possibilities

Once the happy path is clear, think like a product manager stress-testing the feature. Explore the following dimensions and generate a list of candidate scenarios:

**Alternative flows**
- What if the actor is in a different state? (e.g. not logged in, already completed the action)
- Are there multiple valid paths to the same outcome?

**Boundary conditions**
- What happens at the edges? (e.g. empty input, maximum allowed value, first time vs. returning user)

**Failure paths — Checked Exceptions only**
- What predictable failures must the user/caller handle? (e.g. resource not found, input invalid at a business boundary, external service unavailable)
- Do NOT enumerate failures caused by programming errors (null where non-null required, illegal argument) — these are Runtime Exceptions and are out of scope.

**Permissions and access**
- Can different actors get different outcomes for the same action?

**Side effects and downstream behaviors**
- Does this action trigger something else observable? (e.g. notification sent, audit log written, quota decremented)

Present the candidate scenarios to the user in plain language — not Gherkin yet. Group them clearly:

```
Happy path:
- [scenario description]

Alternative flows:
- [scenario description]
- ...

Failure paths:
- [scenario description]
- ...

Side effects (if any):
- [scenario description]
- ...
```

Then ask: "Are there any scenarios I missed? Any you'd like to remove or change?"

### Step 3: Confirm the scenario list

Incorporate user feedback. Repeat Step 2's confirmation if significant new scenarios were added.

When the user confirms the list is complete, proceed to write the Gherkin.

### Step 4: Write the `.feature` file

Create a file at `features/<feature-name>.feature`.

Rules for writing Gherkin:
- `Given` sets up context (what is already true)
- `When` describes the action the actor takes
- `Then` describes the observable outcome (behavior, not state)
- One scenario per distinct behavior
- Scenario names are in business language, not technical language
- Do NOT reference function names, classes, database tables, or field names

Example of good Gherkin:
```gherkin
Feature: User login

  Scenario: Successful login with valid credentials
    Given a registered user with email "alice@example.com"
    When the user logs in with correct credentials
    Then the user is redirected to the dashboard

  Scenario: Login fails with wrong password
    Given a registered user with email "alice@example.com"
    When the user logs in with an incorrect password
    Then the user sees an error message
    And the user remains on the login page
```

### Step 5: Review

Read the written `.feature` file and verify:
- [ ] Every scenario has a single, clear business outcome in `Then`
- [ ] No implementation details leaked into steps
- [ ] All confirmed scenarios from Step 3 are represented
- [ ] No Runtime Exception scenarios slipped in

### Step 6: Hand off

Announce: "Gherkin spec written to `features/<name>.feature`. Moving to test generation."

Then invoke the `gti-test` skill.

