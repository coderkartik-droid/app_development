# Introduction

Flutter/DRF integration bugs often arise from mismatched contracts, incorrect networking assumptions, unsafe token handling, or backend policy gaps.

# Why this concept exists

Each side may appear correct independently while their HTTP boundary is incompatible or insecure.

# Real-world analogy

Two teams can build good equipment that still fails if their connectors use different shapes and voltages.

# Internal Working

Trace the boundary: URL, method, headers, encoding, parser, auth, permissions, serializer, database, status, response schema, Flutter mapping, UI state.

# Data Flow

Flutter request -> transport -> DRF -> database -> response -> Flutter decoder/state.

# Complete Example 1

```dart
final uri = Uri.https('api.example.com', '/api/products/', {'page': '2'});
final response = await client.get(uri);
if (response.statusCode != 200) throw ApiException.fromResponse(response);
```

This avoids string-concatenated query parameters and checks status before decoding. A common mistake is decoding every response as a success page; error bodies may be different or non-JSON.

# Complete Example 2

```python
class PrivateResourceViewSet(ModelViewSet):
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return Resource.objects.filter(owner=self.request.user)
```

Authentication without this scope would still allow cross-user access. Keep client-side hiding separate; the server enforces visibility for every request.

# Complete Example 3

```dart
if (response.statusCode == 204) {
  removeFromUiState(id);
} else if (response.statusCode >= 200 && response.statusCode < 300) {
  updateFromJson(response.body);
}
```

This avoids parsing a 204 no-content response. A stronger implementation handles expected statuses per operation and decodes only when the contract promises a body.

# Advanced Example

Build a compatibility matrix for app version/API version and test schema changes, pagination envelopes, nullability, and token expiry behavior.

# Under the Hood

HTTP responses complete Futures even when status is 4xx/5xx; network failures throw separately. DRF object permissions do not automatically filter collection results.

# Performance Notes

Avoid repeated requests in build methods, unbounded lists, and blind retries.

# Security Notes

Never put tokens in URLs, ship server secrets, trust client roles, or disable TLS validation.

# Best Practices

- Maintain shared contract docs.
- Test each layer and the end-to-end boundary.
- Keep auth/error logic centralized.
- Redact logs and handle lifecycle safely.

# Common Mistakes

- Assuming `localhost` targets development machine on a phone.
- Ignoring paginated JSON shape.
- Treating UI authorization as real authorization.
- Storing tokens in plain preferences.

# Debugging

Compare raw request/response with expected contract, then trace DRF route/policy and Flutter parsing/state.

# Interview Questions

1. Why can valid HTTP response still fail Flutter parsing?
2. Why must DRF scope data independently?
3. Why does a 204 need special handling?

# Practice Exercises

1. Debug a wrong base URL and a wrong response envelope.
2. Audit app for token leakage.
3. Add regression tests for a contract mismatch.

# Chapter Summary

Integration failures happen at boundaries. Verify exact HTTP behavior, keep backend authorization authoritative, and test status/body semantics end to end.
