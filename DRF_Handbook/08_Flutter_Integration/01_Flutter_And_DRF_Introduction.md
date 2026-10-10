# Introduction

Flutter and Django REST Framework (DRF) work together through HTTP. Flutter builds the user interface and sends requests; DRF applies server-side rules, reads or changes data, and returns a representation such as JSON.

# Why this concept exists

A mobile application often needs shared accounts, centrally stored data, and access from multiple devices. A backend provides a trusted place for those responsibilities.

# Real-world analogy

Flutter is the customer-facing service counter; DRF is the secured operations desk that checks requests, updates records, and returns receipts.

# Internal Working

Flutter serializes request data to JSON or multipart form data, sends it over HTTP(S), and awaits a response. Django routes it to DRF; authenticators and permissions run, serializers validate, ORM queries the database, and a renderer returns a response. Flutter decodes the response and updates UI state.

# Data Flow

```text
Flutter widget -> service/repository -> HTTP -> DRF route/view
-> auth/permission -> serializer -> ORM/database
<- JSON/status <- renderer <- DRF response <- Future/state/UI
```

# Complete Example 1

Flutter client:

```dart
import 'dart:convert';
import 'package:http/http.dart' as http;

Future<List<Map<String, dynamic>>> fetchProducts() async {
  final uri = Uri.parse('https://api.example.com/api/products/');
  final response = await http.get(uri, headers: {'Accept': 'application/json'});
  if (response.statusCode != 200) {
    throw Exception('Product request failed: ${response.statusCode}');
  }
  final decoded = jsonDecode(response.body) as Map<String, dynamic>;
  return (decoded['results'] as List)
      .cast<Map<String, dynamic>>();
}
```

DRF backend:

```python
from rest_framework import generics
from rest_framework.permissions import AllowAny
from .models import Product
from .serializers import ProductSerializer

class ProductListView(generics.ListAPIView):
    queryset = Product.objects.filter(active=True).order_by("name", "pk")
    serializer_class = ProductSerializer
    permission_classes = [AllowAny]
```

**Line-by-line:** Flutter builds an HTTPS URI, sends GET with an Accept header, checks status before decoding, parses the paginated JSON object, and extracts its `results`. DRF exposes an intentionally public, active-only ordered list and serializes the queryset.

**Flow:** Flutter Future -> HTTP GET -> Django URL -> DRF view -> queryset/database -> serializer/rendering -> JSON response -> JSON decode -> UI. The result key assumes page-number pagination; if the backend returns a bare list, adapt parsing. Production code should use a typed model, timeout, and explicit error types rather than `Exception`.

# Complete Example 2

Flutter UI:

```dart
FutureBuilder<List<Map<String, dynamic>>>(
  future: fetchProducts(),
  builder: (context, snapshot) {
    if (snapshot.hasError) return Text('Could not load products');
    if (!snapshot.hasData) return const CircularProgressIndicator();
    return ListView(
      children: snapshot.data!
          .map((product) => ListTile(title: Text(product['name'] as String)))
          .toList(),
    );
  },
)
```

The builder renders loading, failure, or data states. In a stateful production screen, create the Future once rather than starting a request on every rebuild; use a repository and state-management solution as the app grows.

# Complete Example 3

DRF URL:

```python
from django.urls import path
from .views import ProductListView

urlpatterns = [path("products/", ProductListView.as_view(), name="product-list")]
```

Mount this app URL under `api/` in the project URL configuration to match Flutter's `/api/products/`. URL configuration is part of the contract; a missing include results in 404 before the view runs.

# Advanced Example

Define one API contract with documented base URL, pagination envelope, error schema, authentication, and versioning. Generate Flutter models from schema where practical, but review nullability and date/decimal handling.

# Under the Hood

Flutter `Future` represents pending asynchronous work; the HTTP client uses the platform network stack. DRF parses request content lazily, runs policy checks, and renders response data after the view returns.

# Performance Notes

Paginate list data, avoid duplicate requests, set timeouts, and keep JSON payloads focused.

# Security Notes

Use HTTPS, never embed server secrets in Flutter, validate authorization server-side, and store refresh credentials only with platform-backed secure storage where necessary.

# Best Practices

- Separate UI, repository, and transport concerns.
- Use typed models and predictable error handling.
- Treat the backend as authoritative for permissions.
- Test both client parsing and server contract.

# Common Mistakes

- Calling localhost from a physical phone without a reachable host.
- Assuming a successful HTTP transport means a successful API result.
- Hard-coding secrets or trusting hidden UI controls.

# Debugging

Inspect base URL, DNS/TLS, HTTP status/body, DRF logs, route configuration, and Flutter parsing exceptions.

# Interview Questions

1. How does Flutter communicate with DRF?
2. Why separate repository from widget?
3. Why must authorization remain server-side?

# Practice Exercises

1. Fetch and display a paginated product list.
2. Add typed parsing and a timeout.
3. Document a request/response contract.

# Chapter Summary

Flutter and DRF communicate through a defined HTTP contract. Flutter handles asynchronous transport and UI state; DRF remains responsible for validation, persistence, and access control.
