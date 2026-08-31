---
inclusion: auto
description: Write a failing test first, then implement. No exceptions.
---

# TDD Mandatory

- RED: Write a failing test that describes the desired behavior. Run it. Confirm it fails.
- GREEN: Write the minimal code to make the test pass. Nothing more.
- REFACTOR: Improve the code while keeping tests green.
- Every new behavior gets a failing test first.
- Every bug fix starts with a test that reproduces the bug.
- Every refactor keeps existing tests green.
- If you need a helper, write tests for it first (recursive TDD).
- Writing code without a failing test is a violation — no exceptions.
- Test behavior, not implementation details.
- Property tests for pure functions; behavioral tests for orchestration.
- Mock at boundaries (HTTP, DB, filesystem), not internal components.
