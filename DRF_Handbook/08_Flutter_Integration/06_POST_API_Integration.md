# Introduction

POST integration sends new data from Flutter to DRF, handles validation, and presents the created resource or errors.

# Why this concept exists

Users need to create records from mobile forms without directly accessing the database.

# Real-world analogy

Flutter submits a completed order form; DRF checks it, records the order, and returns a receipt.

# Internal Working

Flutter encodes input as JSON and sends POST. DRF parses it, authenticates/authorizes, validates serializer fields, saves through the ORM, and returns a representation—commonly 201.

# Data Flow

Flutter form -> JSON -> HTTP POST -> DRF auth/permission -> serializer validation
-> ORM/database -> 201 JSON -> Flutter model/state/UI.

# Complete Example 1

Flutter request:

```dart
Future<Product> createProduct(http.Client client, String name, String sku) async {
  final response = await client.post(
    Uri.https('api.example.com', '/api/products/'),
    headers: {'Content-Type': 'application/json', 'Accept': 'application/json'},
    body: jsonEncode({'name': name, 'sku': sku, 'price': '10.00', 'active': true}),
  );
  if (response.statusCode != 201) {
    throw ApiException.fromResponse(response);
  }
  return Product.fromJson(jsonDecode(response.body) as Map<String, dynamic>);
}
```

`jsonEncode` produces the request body; content type tells DRF to use JSON parsing; status is checked before parsing success; returned JSON maps to Product. Add bearer credentials through the API client when required; never trust client-provided owner fields.

# Complete Example 2

DRF backend:

```python
class ProductCreateView(CreateAPIView):
    serializer_class = ProductSerializer
    permission_classes = [IsAuthenticated]

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)
```

The view validates with ProductSerializer and assigns owner from authenticated identity. Ensure owner is read-only/not accepted from client. Database constraints should handle uniqueness under concurrency.

# Complete Example 3

Flutter form result handling:

```dart
try {
  final product = await repository.create(name, sku);
  if (!context.mounted) return;
  Navigator.of(context).pop(product);
} on ValidationException catch (error) {
  if (!context.mounted) return;
  setState(() => fieldErrors = error.fields);
}
```

Success returns the created model to the previous screen; validation maps field errors to form UI. `context.mounted` prevents using a disposed widget after an async gap. Keep transport parsing out of the widget where possible.

# Advanced Example

Use idempotency keys for retry-prone creates (payments/orders) and persist them with a unique constraint. A network timeout does not prove the server failed to create the resource.

# Under the Hood

DRF's create mixin constructs a serializer from request data, validates, calls `perform_create`, and returns serializer data with 201. Flutter's Future completes after a response or transport exception.

# Performance Notes

Disable duplicate taps while sending, bound payload size, and avoid large synchronous encoding in UI.

# Security Notes

Use HTTPS, authenticated requests, server-owned field assignment, throttling, validation, and safe error messages.

# Best Practices

- Handle 201 and structured 400 responses.
- Keep server-calculated fields out of input.
- Make non-idempotent operations retry-safe.
- Update UI only while mounted.

# Common Mistakes

- Treating any 2xx as the expected response shape.
- Sending `toString()` instead of JSON encoding.
- Retrying after timeout without idempotency.

# Debugging

Inspect outgoing JSON/content type, auth header, serializer errors, status/body, and persisted database record.

# Interview Questions

1. Why return 201 for creation?
2. Why can timeout create duplicate POSTs?
3. Where should owner be assigned?

# Practice Exercises

1. Build Flutter product creation with field errors.
2. Add auth and owner assignment.
3. Implement idempotency for an order submission.

# Chapter Summary

POST integration spans form validation, JSON encoding, authentication, server validation, persistence, and response mapping. Design retry and error behavior explicitly.
