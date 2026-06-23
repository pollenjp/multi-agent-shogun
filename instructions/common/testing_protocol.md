# Testing Protocol

## Test Pyramid Policy

| Layer | Ratio | Owner | Characteristics |
|-------|-------|-------|----------------|
| Unit | 60-70% | Ashigaru | Fast (<100ms/test), mock external deps only |
| Integration | 20-30% | Ashigaru | Real DB/API (test containers) |
| E2E | 5-10% | Karo | Critical user flows only |

## TDD Cycle (when task has `tdd: true`)

1. **Red**: Write one failing test. No implementation code.
2. **Green**: Write minimum code to pass the test.
3. **Refactor**: Improve while keeping tests green.

Repeat per test case. Batch-writing multiple tests then implementing is prohibited.

## Test Design Principles

- **Arrange-Act-Assert** pattern
- **1 test = 1 behavior** (multiple asserts OK if related)
- **Test independence**: No shared state, no execution order dependency
- **Mock minimization**: Mock external deps only. No mocking internal logic.

## Test Naming

| Language | Format | Example |
|----------|--------|---------|
| Python/Go | `test_<target>_<condition>_<expected>` | `test_calc_shipping_empty_cart_returns_zero` |
| TypeScript | `describe/it` nesting | `describe('calcShipping') > it('returns zero for empty cart')` |

## Coverage Thresholds

Default (when not specified in task YAML):
- Line: 80%+
- Branch: 70%+

Task YAML `coverage` field overrides defaults.

## E2E Standards (Karo/Gunshi reference)

- **Page Object Model**: Mandatory. One class per page.
- **Locator priority**: data-testid > getByRole/getByLabel > getByText > CSS/XPath
- **No fixed waits**: sleep(), time.sleep(), fixed-ms waits absolutely prohibited.
- **Test data**: Prepare via API (never through UI).
- **On failure**: Auto-save screenshot + trace.
- **Scope**: Critical user flows only (login, payment, permissions, error fallback).

## Code Quality (applies to test code too)

- Functions: ~20 lines max (split if longer)
- Arguments: 3 max (use object/struct if more)
- Naming: Express intent (no `data`, `tmp`, `ret`)
- No empty catch blocks
- No magic numbers (use constants)
