# Testing Guidelines

> General testing principles for Rails apps. Copy to `docs/TESTING.md` for new projects.
> This doc grows with the project — add project-specific testing patterns as they emerge.

---

## Testing Layers

| Layer | Tool | Purpose | Volume |
|-------|------|---------|--------|
| **Services + Models** | Minitest | Business logic, validations, scopes | Heavy |
| **Controllers** | Minitest + Inertia helpers | Authorization, correct component/props | Heavy |
| **E2E (System)** | Capybara + Playwright | Critical happy paths in a real browser | Light |

**Test at the lowest possible layer.** If you can verify it in a service test, don't write a controller test. If you can verify it in a controller test, don't write a browser test. Higher layers are slower, flakier, and harder to debug.

---

## Principles

### Test the contract, not the implementation

Assert what happens (output, side effects, state changes), not how it happens internally. Tests that verify method calls or internal ordering break on every refactor and provide zero confidence.

### Name tests like sentences

`test "deactivated user cannot log in"` not `test_auth_edge_case_4`. Someone should be able to read test names alone and understand every behavior the system supports.

### Every bug gets a regression test

Before you fix a bug, write a test that reproduces it. Watch it fail. Fix the code. Watch it pass. This guarantees the same bug never ships twice.

### Don't test the framework

Rails already tests that `validates :name, presence: true` works. Test your business rules — the things that are unique to your domain and would break if someone changed them.

### Cover the sad paths

Happy paths are obvious and rarely where bugs live. The real value is in: invalid inputs, unauthorized access, state transitions that shouldn't be allowed, empty collections, nil values, and race conditions.

### Tests are documentation

A new developer should be able to read your test file and understand what the feature does, what the edge cases are, and what's not allowed — without reading the implementation.

### Arrange-Act-Assert, always

Setup the world, do the thing, check the result. One concept per test. If a test fails, you should know exactly what broke from the test name alone.

### Flaky tests are worse than no tests

A test that passes 95% of the time erodes trust in the entire suite. Fix flaky tests immediately or delete them. Never skip them.

---

## Mocking & Stubbing

### Mock your interfaces, not third-party code

Stub your service interfaces, not external client libraries. If you mock what you don't own, your tests pass while production breaks.

### Prefer real objects when possible

Only stub external services (notification providers, email delivery, payment processors) and slow dependencies. Let models, services, and database interactions run for real — that's the whole point of the test.

---

## E2E / System Tests

### Playwright over Selenium

Use `capybara-playwright-driver` — same Capybara DSL, but Playwright under the hood. More reliable with modern JS frameworks.

### Keep E2E tests focused

Each system test should cover one complete user flow. Don't chain multiple features into mega-tests — when they fail, you can't tell what broke.

### E2E is for JavaScript-dependent flows

If a behavior can be verified without rendering JS components, test it at the controller layer instead. Reserve browser tests for multi-step interactions that require real JS execution.
