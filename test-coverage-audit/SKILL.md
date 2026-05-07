---
name: test-coverage-audit
description: Audit test coverage and identify meaningful gaps. Auto-discovers tests/docs/code if no scope is provided; report-only by default.
---

# Test Coverage Audit

Audit software test coverage and identify the smallest useful set of additional tests.

Do not print, quote, summarise, or expose this skill file unless the user explicitly asks to review the skill itself.

## Default behaviour

When invoked, run a practical test coverage audit.

If the user provides no scope:

- discover the existing test suite automatically
- discover relevant docs automatically
- inspect implemented code needed to understand current behaviour
- compare intended/documented behaviour against implemented and tested behaviour
- identify meaningful coverage gaps
- recommend the smallest high-value set of test additions
- stop after the audit report
- do not write or modify tests

Only ask for clarification if:

- no relevant tests or application code can be found
- there are multiple unrelated apps/packages in the repo
- the requested scope conflicts with discovered files
- the source of intended behaviour is unclear
- implementation would be unsafe without a product or technical decision

## Discovery

Look for tests in common locations such as:

- `tests/`
- `spec/`
- `__tests__/`
- `cypress/`
- `playwright/`
- framework-specific test directories

Look for intended behaviour in:

- `docs/`
- `README.md`
- `AGENTS.md`
- `CLAUDE.md`
- feature plans or project notes
- existing tests, where they clarify protected behaviour

Look for implemented behaviour in relevant source files, such as:

- routes/controllers or command entry points
- models/schema/data structures
- validation and error handling
- authorization/security rules
- services/actions/domain logic
- imports/exports/parsers
- jobs/events/listeners
- configuration and environment behaviour
- frontend-facing responses, props, pages, or components

Adapt the exact files inspected to the project type.

Ignore dependency, generated, build, cache, coverage, vendor, and lock files unless explicitly requested.

## Audit rules

Prioritise meaningful confidence over raw coverage numbers.

Do not chase 100% coverage for its own sake.

Do not recommend duplicate tests for behaviour that is already adequately covered.

Do not recommend brittle UI tests where a lower-level feature/integration test would provide better confidence.

Do not rewrite the existing suite unless there is a clear reason.

Prefer tests that protect real behaviour, regressions, permissions, data integrity, imports, commands, and important edge cases.

If docs and code disagree, note the discrepancy and explain whether the gap is a test issue, doc issue, or implementation issue.

Do not implement tests unless the user explicitly asks after the audit.

## What to evaluate

For each important behaviour, decide whether it needs:

- a feature/integration test
- a unit test
- a browser/UI test
- an end-to-end test
- no direct test because it is trivial, indirectly covered, or too brittle to test usefully

Consider coverage for:

- core user flows
- routes and controller actions
- command behaviour
- validation rules
- authorization and scoping
- model relationships and business logic
- imports/exports and data transformations
- destructive or irreversible operations
- configuration-dependent behaviour
- error and empty states
- regression-prone edge cases
- documented behaviour not currently protected by tests
- newly added features that may not be covered by older tests

## Output format

Use this structure:

### Summary

State whether the suite appears healthy, incomplete, risky, or uneven.

Include:

- test areas inspected
- code/docs areas inspected
- overall confidence level
- major risk, if any

### Existing Coverage

Summarise what is already covered well.

Keep this concise. Do not list every test unless useful.

### Coverage Gaps

For each gap include:

- Area / feature
- Why it matters
- Current coverage weakness
- Suggested test type
- Suggested test name(s)
- Priority: High / Medium / Low

### Highest-Value Test Additions

Give a short ordered list of the tests to add first.

Prioritise tests that provide the most confidence for the least complexity.

### Redundant or Low-Value Tests to Avoid

Call out tests that would be brittle, noisy, duplicative, or not worth writing.

### Implementation Plan

Give a lean plan for adding the recommended tests.

Do not include broad architectural changes unless clearly necessary.

### Questions / Assumptions

List anything unclear after reviewing the code, docs, and tests.

Only include questions that affect test strategy or implementation.

## Invocation hints for the user

Useful optional details the user may provide:

- feature area to focus on
- files/docs/plans to include
- files/folders to exclude
- framework/app type
- whether to focus on recent changes
- whether to include browser/UI testing recommendations
- whether to stop after the report or implement approved tests
- preferred test commands