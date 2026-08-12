# Testing Rules

## TDD is Mandatory for New Features

Always follow RED → GREEN → REFACTOR:
1. Write a failing test that describes the desired behavior
2. Write minimum code to make it pass
3. Refactor - clean up without breaking the test

Never write implementation before the failing test.

## Coverage Threshold

- Business logic: ≥ 80% line coverage
- Utilities and helpers: ≥ 70%
- Check before finishing any feature:
  ```bash
  vitest --coverage
  ```

## Test Rules

- Each test covers ONE behavior
- Test names describe what they verify: `it("returns null when user not found")`
- Tests must be independent - no shared mutable state between tests
- No skipped tests (`it.skip`, `describe.skip`) without a comment explaining why
- Happy path + at least one failure path per public function

## Test File Location

- Co-locate tests with source: `user.ts` → `user.test.ts`
- Or use a `tests/` mirror of `src/`
- Consistent - pick one and stick to it
