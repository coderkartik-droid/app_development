# Introduction

These exercises turn CRUD concepts into implementation and reasoning practice. Build each endpoint in a Django app with DRF installed and a test database.

# Why this concept exists

Reading about request flow is not enough; implementing failure cases reveals hidden assumptions about permissions, serializers, and database behavior.

# Real-world analogy

Practice is a flight simulator: safely rehearse routine requests and failures before operating a production service.

# Internal Working

For each task, trace the URL/action, permission, serializer, queryset, database operation, response status, and tests. Keep implementation decisions explicit.

# Request Lifecycle

```text
client -> URL/router -> policy -> input serializer -> ORM/transaction
       -> output serializer -> status/body -> assertions
```

# Syntax

```python
response = self.client.patch(url, {"active": False}, format="json")
self.assertEqual(response.status_code, 200)
```

# Complete Example 1

Implement a read-only public catalogue list:

```python
from rest_framework.generics import ListAPIView
from rest_framework.permissions import AllowAny

class PublicProductList(ListAPIView):
    serializer_class = ProductSerializer
    permission_classes = [AllowAny]

    def get_queryset(self):
        return Product.objects.filter(active=True).order_by("name", "pk")
```

`AllowAny` intentionally permits public reads; active filtering defines visibility; stable order supports pagination. Add a URL and pagination class. Test that inactive rows never appear and ordering is deterministic.

# Complete Example 2

Implement owner-only creation:

```python
from rest_framework.generics import CreateAPIView
from rest_framework.permissions import IsAuthenticated

class OwnedProductCreate(CreateAPIView):
    serializer_class = ProductSerializer
    permission_classes = [IsAuthenticated]

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)
```

The authenticated user supplies the owner through trusted request context. Ensure `owner` is not writable from request input. Test anonymous rejection, successful owner assignment, and attempts to spoof another owner.

# Complete Example 3

Test invalid update:

```python
response = self.client.patch(
    f"/api/products/{product.pk}/",
    {"price": "-5.00"},
    format="json",
)
self.assertEqual(response.status_code, 400)
self.assertIn("price", response.data)
```

The test documents the field-level error contract. It assumes a nonnegative price validator exists and the test client is authenticated as an authorized user. Also assert the database value remains unchanged after rejection.

# Advanced Example

Capstone exercise: build product CRUD with owner isolation, category filtering, search, allowlisted ordering, pagination, schema documentation, and tests for every permission and validation boundary. Add atomic bulk import only after specifying all-or-nothing behavior.

# Under the Hood

DRF tests can use `APITestCase`/`APIClient` to exercise routing and middleware-adjacent behavior. Queryset evaluation and transactions occur against the test database; test setup should isolate data.

# Performance Notes

Add a query-count assertion for nested list serialization and test with enough rows to reveal unbounded or N+1 behavior.

# Best Practices

- Write success and failure tests.
- Use explicit fixtures and authenticated identities.
- Assert persisted effects as well as response status.
- Keep each exercise focused and repeatable.

# Common Mistakes

- Testing only the happy path.
- Skipping authentication setup and drawing wrong permission conclusions.
- Asserting implementation details instead of API contract.

# Debugging

Read response data before changing code; inspect test database setup, URL reverse, authentication, serializer errors, and persisted state.

# Interview Questions

1. Which tests prove tenant isolation?
2. How can you test a 204 response?
3. Why assert DB state after invalid input?

# Practice Exercises

1. Add GET list/detail and assert response fields.
2. Add POST/PUT/PATCH/DELETE tests.
3. Add filtering, ordering, and pagination tests.
4. Add duplicate SKU and cross-owner access tests.

# Chapter Summary

Practice complete contracts, especially forbidden and invalid requests. A reliable endpoint is demonstrated by tests for response shape, persisted state, access boundaries, and bounded list behavior.
