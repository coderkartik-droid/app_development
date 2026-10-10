# Introduction

An API service layer centralizes HTTP transport details such as base URLs, headers, JSON encoding, timeouts, status checks, and error parsing.

# Why this concept exists

Duplicating networking code in widgets leads to inconsistent auth, error handling, and request behavior.

# Real-world analogy

The service layer is a shared dispatch desk that sends every package with the correct address and tracking rules.

# Internal Working

Feature repositories call service methods; the client constructs HTTP requests and maps responses/errors. DRF sees ordinary HTTP requests and remains independent of Flutter UI state.

# Data Flow

Widget/controller -> repository -> API service/client -> HTTP -> DRF -> response/error mapper -> repository result -> UI.

# Complete Example 1

```dart
class ApiClient {
  ApiClient(this._client, this.baseUri);
  final http.Client _client;
  final Uri baseUri;

  Future<Map<String, dynamic>> get(String path) async {
    final response = await _client.get(
      baseUri.resolve(path),
      headers: {'Accept': 'application/json'},
    ).timeout(const Duration(seconds: 15));
    if (response.statusCode < 200 || response.statusCode >= 300) {
      throw parseApiError(response);
    }
    return jsonDecode(response.body) as Map<String, dynamic>;
  }
}
```

Dependency injection enables fake clients; `resolve` builds relative paths; headers and timeout are centralized; status is checked before decoding. Production should add auth injection, empty-body support, typed decoding, and client disposal policy.

# Complete Example 2

```python
class ProductListView(ListAPIView):
    serializer_class = ProductSerializer
    permission_classes = [AllowAny]
    queryset = Product.objects.filter(active=True).order_by("pk")
```

This backend endpoint is intentionally public. The Flutter service does not change DRF policy; backend access decisions remain authoritative. Mount the URL matching the configured client base.

# Complete Example 3

```dart
class ProductApi {
  ProductApi(this._api);
  final ApiClient _api;

  Future<List<Product>> list() async {
    final page = await _api.get('/products/');
    return (page['results'] as List)
        .map((item) => Product.fromJson(item as Map<String, dynamic>))
        .toList();
  }
}
```

The feature API owns route and response mapping; transport details remain in `ApiClient`. This separation lets tests substitute a fake client and lets the UI use typed products.

# Advanced Example

Add an auth token provider, request ID header, redacted logging, refresh coordination, cancellation, and explicit retry policy. Avoid a generic method that obscures endpoint-specific contracts.

# Under the Hood

The API service converts application-level inputs to HTTP representation; the backend parser converts bytes to Python values. HTTP status and response body must be interpreted before model mapping.

# Performance Notes

Reuse clients, avoid redundant serialization and requests, and support pagination/cancellation in service contracts.

# Security Notes

Centralize token injection but never log it. Keep secrets out of Flutter build artifacts and enforce authorization server-side.

# Best Practices

- Inject client/configuration.
- Use typed errors and models.
- Keep UI independent from HTTP package.
- Close resources deliberately.

# Common Mistakes

- Hard-coding URLs throughout widgets.
- Swallowing errors in service methods.
- Treating all endpoints as the same JSON shape.

# Debugging

Log sanitized method/path/status/correlation data and inspect request headers/body only after redaction.

# Interview Questions

1. What belongs in API client versus repository?
2. How does dependency injection aid tests?
3. Why centralize error parsing?

# Practice Exercises

1. Extract inline HTTP calls into ApiClient.
2. Add auth header injection and redaction.
3. Test service methods with a fake HTTP client.

# Chapter Summary

The service layer standardizes transport while repositories express feature data access. Keep contracts typed, errors explicit, and credentials protected.
