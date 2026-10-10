# Introduction

PUT, PATCH, and DELETE let Flutter replace, partially change, or remove a DRF resource.

# Why this concept exists

Clients need to maintain records after creation with clear semantics for complete replacement, targeted edits, and removal.

# Real-world analogy

PUT replaces a whole specification, PATCH marks one correction, and DELETE withdraws the record.

# Internal Working

DRF retrieves an authorized object, checks permissions, validates updates, persists changes, and returns a representation or no-content response. Flutter must send the correct payload and understand the resulting status.

# Data Flow

Flutter action -> HTTP verb -> DRF route/action -> auth/policy -> serializer/object
-> database -> status/body -> Flutter state update.

# Complete Example 1

Flutter client methods:

```dart
Future<http.Response> patchProduct(http.Client client, int id, String name) {
  return client.patch(
    Uri.https('api.example.com', '/api/products/$id/'),
    headers: {'Content-Type': 'application/json', 'Authorization': 'Bearer $access'},
    body: jsonEncode({'name': name}),
  );
}

Future<http.Response> deleteProduct(http.Client client, int id) {
  return client.delete(
    Uri.https('api.example.com', '/api/products/$id/'),
    headers: {'Authorization': 'Bearer $access'},
  );
}
```

PATCH sends only changed fields; DELETE sends no body here. Both use bearer credentials and resource IDs in the path. Check status and map errors in a shared client; never interpolate untrusted IDs into query fragments.

# Complete Example 2

DRF backend:

```python
class ProductViewSet(ModelViewSet):
    serializer_class = ProductSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return Product.objects.filter(owner=self.request.user)
```

ModelViewSet provides PUT/PATCH/DELETE; queryset scope restricts target objects. Serializer must keep immutable fields read-only. For soft deletion, override `perform_destroy` and define how reads hide archived rows.

# Complete Example 3

Flutter status handling:

```dart
if (response.statusCode == 200) {
  final updated = Product.fromJson(jsonDecode(response.body));
} else if (response.statusCode == 204) {
  // Delete succeeded; remove the item from local state.
} else {
  throw ApiException.fromResponse(response);
}
```

200 commonly includes updated representation; 204 has no body, so do not JSON-decode it. A successful DELETE should update local UI state. Also handle 404 if the record is missing or inaccessible.

# Advanced Example

For optimistic UI, preserve the old state and roll it back on failure. Use idempotency/version fields for conflict-sensitive updates; avoid automatic retry of a delete or update unless semantics are safe.

# Under the Hood

Generic update dispatch sets `partial=True` for PATCH but not PUT. Destroy usually deletes/archives and returns 204. Permissions run before mutation; database constraints may reject updates.

# Performance Notes

Send only required PATCH fields, avoid full-resource refetch when response already contains updates, and handle cancellation/navigation during requests.

# Security Notes

Scope detail querysets, reject client changes to owner/status/internal fields, and protect unsafe session-auth requests with CSRF.

# Best Practices

- Use PUT for complete representation and PATCH for partial changes.
- Treat 204 as empty.
- Reflect server result in local state.
- Test conflicts and inaccessible IDs.

# Common Mistakes

- Parsing a DELETE 204 body.
- Sending partial fields with PUT.
- Updating UI before server success without rollback.

# Debugging

Check method, path ID, auth, CSRF, serializer errors, object scope, status, and database state.

# Interview Questions

1. Difference between PUT and PATCH?
2. What should Flutter do with 204?
3. Why scope the backend queryset?

# Practice Exercises

1. Add edit form with PATCH.
2. Add delete confirmation and rollback behavior.
3. Test a full PUT missing required values.

# Chapter Summary

Update and deletion integration depends on method semantics, scoped backend access, correct status parsing, and resilient UI state updates.
