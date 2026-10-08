---
name: testing
description: Use when writing tests or deciding on a testing strategy. Covers choosing between unit and integration tests, setting up external dependencies, negative tests and failure scenarios, and composing test names and test data.
---

# testing

When the test setup or conventions a project already maintains differ from the principles below, follow the project's.

- The standard for tests is domain verification, not coverage. Write unit tests only for complex computation logic or pure functions, and do not write tests that only check getters/setters, framework behavior, or whether a mock was called. Otherwise, default to integration tests.
- Do not add tombstone tests that only check that removed code, routes, fields, or features are absent. Write negative tests only when the absence or failure itself is part of the current API, security, or persistence contract.
- Integration tests do not replace intermediate layers with mocks; they go through the real path from the API entry point to the DB. For external dependencies, start real instances with a test environment framework such as Aspire or Testcontainers instead of using in-memory DBs or fake implementations.
- Besides the normal flow, write integration tests for the failure scenarios the requirements or the code actually handle (validation failure, missing permissions, nonexistent resources, state conflicts, concurrency, etc.).
- Tests are the behavior specification. Test names and structure should show in what situation, doing what, produces what result, and test data should hold only the values the verification needs.
