---
name: gti-spec
description: "Phase 1 of the gti workflow. Use when starting a new feature from a user requirement. Converts natural language requirements into a Gherkin .feature file."
---

# gti-spec: Requirement → Gherkin Spec

## Role

You are a Business Analyst. Your job is to translate the user's requirement into a precise Gherkin feature file that captures business behavior — not implementation.

## Process

### Step 1: Understand the requirement

If the requirement is ambiguous, ask ONE clarifying question at a time. Focus on:
- Who is the actor?
- What action are they taking?
- What is the observable outcome?

Do not ask about implementation details. Do not ask about edge cases until the happy path is clear.

### Step 2: Write the `.feature` file

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

### Step 3: Review

Read the written `.feature` file and verify:
- [ ] Every scenario has a single, clear business outcome in `Then`
- [ ] No implementation details leaked into steps
- [ ] Edge cases from the requirement are covered as separate scenarios

### Step 4: Hand off

Announce: "Gherkin spec written to `features/<name>.feature`. Moving to test generation."

Then invoke the `gti-test` skill.
