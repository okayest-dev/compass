# When to mock

Mock at **system boundaries** only:

- External services (payment providers, email senders, and so on).
- Datastores (sometimes, prefer a real or embedded test datastore).
- Time and randomness.
- The filesystem (sometimes).

Do not mock:

- Your own modules or classes.
- Internal collaborators.
- Anything you control.

## Designing for mockability

At system boundaries, design interfaces that are easy to mock.

**1. Use dependency injection**

Pass external dependencies in rather than creating them internally:

```
// Easy to mock: the dependency is supplied by the caller
function processPayment(order, paymentClient):
    return paymentClient.charge(order.total)

// Hard to mock: the dependency is built inside the function
function processPayment(order):
    client = newLivePaymentClient(readSecretFromEnvironment())
    return client.charge(order.total)
```

**2. Prefer purpose-built operations over one generic call**

Create a specific operation for each external action instead of one generic call with conditional logic buried inside:

```
// GOOD: each operation is independently mockable
api.getUser(id)
api.getOrders(userId)
api.createOrder(data)

// BAD: mocking requires conditional logic inside the mock
api.request(endpoint, options)
```

The purpose-built approach means:

- Each mock returns one specific shape.
- No conditional logic is needed in test setup.
- It is easy to see which operations a test exercises.
- Each operation's inputs and outputs can be checked independently.
