# Repro Guidelines

Standards for writing failing tests that reproduce bugs. Follow `compass-test` TDD principles: red before green, test observable behaviour, vertical slices.

---

## Scope inference

Infer the test scope from the bug report, then **confirm with the user** before writing.

| Bug report signals | Inferred scope | Test type |
|--------------------|----------------|-----------|
| Single function, class, or module mentioned | Unit | Unit test |
| "Service A calls Service B", "module X uses module Y" | Integration | Integration test |
| "API returns X", "frontend shows Y", cross-app boundary | Contract | Contract test (both apps in harness) |
| Unclear / spans multiple layers | Ask user | — |

**Confirmation prompt:** "This looks like a [unit/integration/contract] bug. Should I write a failing test at that level?"

---

## Unit repro

- Test the smallest unit that exhibits the bug (function, method, class)
- Use the project's existing unit test framework and conventions
- Mock only external dependencies (I/O, time, randomness) — see `compass-test` MOCKING.md
- Test through the public interface of the unit
- One test case per distinct bug symptom

```python
# Example: unit test for a discount calculation bug
def test_discount_applies_to_eligible_items_only():
    cart = Cart(items=[eligible_item, ineligible_item])
    discount = calculate_discount(cart)
    assert discount.amount == eligible_item.price * 0.1
```

---

## Integration repro

- Test the interaction between two or more modules within the same application
- Use real implementations where possible; mock only external boundaries (DB, HTTP, message queue)
- Follow the project's integration test conventions (test DB, test containers, in-memory adapters)
- Test through the highest seam that covers the interaction (see `compass-design`)

```python
# Example: integration test for order → payment interaction
def test_order_creation_triggers_payment_authorization():
    order = create_order(cart, customer)
    payment = payment_gateway.authorize(order.total)
    assert payment.status == "authorized"
    assert order.payment_id == payment.id
```

---

## Contract repro

- Test the contract between two applications/services
- Both applications must be in the test harness (test instances, not mocks)
- Use the project's contract testing framework (Pact, Spring Cloud Contract, custom)
- Verify both provider and consumer sides if possible
- Focus on the specific contract violation (schema, status code, event shape)

```yaml
# Example: contract test fragment (Pact-style)
provider:
  path: /api/v1/orders
  method: POST
  request:
    body:
      items: [{"sku": "ABC", "qty": 1}]
  response:
    status: 201
    body:
      order_id: "matching(regex, '^ord_')"
      status: "created"
```

---

## Test standards

- **Red first:** The test must fail before any fix. Capture the failing output.
- **Observable behaviour:** Test what the user sees, not internal state.
- **Single command:** `pytest tests/unit/test_discount.py::test_discount_applies_to_eligible_items_only` or equivalent.
- **No implementation code:** Only the failing test. The fix comes later via `compass-implement`.
- **Follow project conventions:** Naming, structure, fixtures, helpers — mirror existing tests.

---

## When repro cannot be written

If the inferred scope lacks test infrastructure (no test DB, no contract harness, no integration test setup):
1. Document what's missing
2. Escalate to user: "Cannot write [integration/contract] test because [missing infrastructure]. Options: [unit test at smaller scope / set up infrastructure / defer]"
3. Do not write a test at the wrong scope just to have something.

---

## Cross-app bug handling

For bugs spanning multiple applications:
1. Identify the contract boundary (API, message queue, shared DB, events)
2. Determine which app owns the contract test
3. Write the failing contract test in the owning app's test suite
4. Ensure the other app is available in the harness (test instance, not mock)
5. If no contract testing exists, recommend `compass-navigate` to decide on approach