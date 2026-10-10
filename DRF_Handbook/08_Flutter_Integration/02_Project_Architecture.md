# Introduction

Project architecture separates Flutter presentation, application state, data access, and HTTP transport from DRF routing, domain logic, and persistence.

# Why this concept exists

Without boundaries, widgets become coupled to URLs and JSON shape, and backend views accumulate unrelated business logic.

# Real-world analogy

A restaurant separates dining room, order coordination, kitchen, and inventory; each team has clear responsibilities and communicates through agreed tickets.

# Internal Working

Flutter screen -> state/controller -> repository interface -> API client/model mapper -> HTTP -> DRF view -> service/serializer -> ORM. Responses return through the same layers; UI does not directly manage database details.

# Data Flow

```text
Widget -> ViewModel/Bloc/Provider -> Repository -> ApiClient
-> DRF endpoint -> service/serializer -> database
<- JSON <- repository model <- state <- widget
```

# Complete Example 1

Flutter repository:

```dart
class ProductRepository {
  ProductRepository(this._client);
  final ApiClient _client;

  Future<List<Product>> list() async {
    final json = await _client.getJson('/products/');
    return (json['results'] as List)
        .map((item) => Product.fromJson(item as Map<String, dynamic>))
        .toList();
  }
}
```

The constructor injects a transport abstraction; `list` owns mapping from API envelope to typed domain models. Widgets depend on this repository instead of URL and JSON details. Add pagination metadata to the return type if the UI needs next-page behavior.

# Complete Example 2

DRF API:

```python
class ProductViewSet(ModelViewSet):
    serializer_class = ProductSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return Product.objects.filter(owner=self.request.user)
```

The backend owns access scope and returns only permitted data. Client architecture cannot enforce this policy. Add versioned URL routing and serializers matching the documented contract.

# Complete Example 3

Minimal client model:

```dart
class Product {
  const Product({required this.id, required this.name});
  final int id;
  final String name;

  factory Product.fromJson(Map<String, dynamic> json) =>
      Product(id: json['id'] as int, name: json['name'] as String);
}
```

This model maps only needed fields and fails clearly on contract mismatch. For optional backend fields, use nullable types and explicit defaults only when contract semantics permit.

# Advanced Example

Define API DTOs separately from UI/domain models when contracts evolve independently. Use dependency injection for client/repository, centralized auth/error interceptors, and contract tests or OpenAPI generation.

# Under the Hood

Repository methods return Futures/Streams. State management translates asynchronous outcomes into loading/success/error states. DRF views orchestrate policy and serializers; domain services own cross-entry-point rules.

# Performance Notes

Avoid unnecessary model copies and duplicate network calls; support paging and cancellation where useful.

# Security Notes

Do not put backend secrets in Flutter assets. Token handling belongs in a dedicated secure layer; backend always authorizes every request.

# Best Practices

- Keep widgets free of HTTP details.
- Inject dependencies for testability.
- Define a stable schema and error contract.
- Keep backend domain rules centralized.

# Common Mistakes

- One giant API service with UI state and parsing mixed in.
- Treating repository separation as security.
- Duplicating business rules only in the client.

# Debugging

Trace one operation through widget, state, repository, client, URL, DRF view, serializer, and database.

# Interview Questions

1. What does a repository abstract?
2. Why separate DTO and domain model?
3. Where should permission enforcement live?

# Practice Exercises

1. Refactor a widget's inline HTTP call to a repository.
2. Add a fake repository for widget tests.
3. Align Flutter model fields with DRF schema.

# Chapter Summary

Architecture creates testable boundaries between Flutter UI and DRF implementation. The repository abstracts data access; the server remains authoritative for identity, policy, and persistence.
