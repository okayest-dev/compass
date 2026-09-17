# Good and bad tests

## Good tests

**Integration-style**: test through real interfaces, not mocks of internal parts.

```
// GOOD: tests observable behaviour
test "user can complete checkout with a valid cart":
    cart = createCart()
    cart.add(product)
    result = checkout(cart, paymentMethod)
    assert result.status == "confirmed"
```

Characteristics:

- Tests behaviour that users or callers care about.
- Uses the public interface only.
- Survives internal refactors.
- Describes WHAT, not HOW.
- One logical assertion per test.

## Bad tests

**Implementation-detail tests**: coupled to internal structure.

```
// BAD: tests implementation details
test "checkout calls the payment service":
    mockPayment = mockOf(paymentService)
    checkout(cart, payment)
    assert mockPayment.process was called with cart.total
```

Red flags:

- Mocking internal collaborators.
- Testing private methods.
- Asserting on call counts or call order.
- The test breaks on refactor without a behaviour change.
- The test name describes HOW, not WHAT.
- Verifying through some external means instead of the interface.

```
// BAD: bypasses the interface to verify
test "createUser saves a row to storage":
    createUser(name: "Alice")
    row = storage.query("select * from users where name = 'Alice'")
    assert row exists

// GOOD: verifies through the interface
test "createUser makes the user retrievable":
    user = createUser(name: "Alice")
    retrieved = getUser(user.id)
    assert retrieved.name == "Alice"
```

**Tautological tests**: the expected value restates the implementation, so the test passes by construction.

```
// BAD: expected value is recomputed the same way the code computes it
test "calculateTotal sums line items":
    items = [{price: 10}, {price: 5}]
    expected = sum(item.price for item in items)
    assert calculateTotal(items) == expected

// GOOD: expected value is an independent, known literal
test "calculateTotal sums line items":
    assert calculateTotal([{price: 10}, {price: 5}]) == 15
```
