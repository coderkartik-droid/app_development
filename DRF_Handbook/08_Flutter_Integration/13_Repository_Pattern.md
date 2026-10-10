# Introduction

The repository pattern abstracts data sources behind feature-oriented methods, giving Flutter code a stable API whether data comes from DRF, cache, or local storage.

# Why this concept exists

Widgets should not know HTTP paths, JSON envelopes, token headers, or whether a response is cached.

# Real-world analogy

A librarian handles requests for books regardless of whether a copy comes from a local shelf or an inter-library loan.

# Internal Working

UI/controller calls repository; repository coordinates remote API and optional cache; API client performs HTTP; DRF validates and persists; repository maps results to models.

# Data Flow

UI -> repository -> API client -> DRF -> serializer/DB -> JSON -> repository model -> UI.

# Complete Example 1

```dart
abstract interface class ProductRepository {
  Future<List<Product>> list();
  Future<Product> create(ProductDraft draft);
}

class DrfProductRepository implements ProductRepository {
  DrfProductRepository(this.api);
  final ProductApi api;

  @override
  Future<List<Product>> list() => api.list();

  @override
  Future<Product> create(ProductDraft draft) => api.create(draft);
}
```

The interface defines feature-level capabilities; implementation delegates to API-specific code. UI depends on the abstraction. Add mapping and errors inside API/repository boundaries, not widgets.

# Complete Example 2

```python
class ProductViewSet(ModelViewSet):
    permission_classes = [IsAuthenticated]
    serializer_class = ProductSerializer

    def get_queryset(self):
        return Product.objects.filter(owner=self.request.user)

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)
```

Repository abstraction on Flutter does not replace backend access control. DRF scopes records and assigns ownership from identity. Always test this contract regardless of which Flutter repository implementation is used.

# Complete Example 3

```dart
class FakeProductRepository implements ProductRepository {
  @override
  Future<List<Product>> list() async => const [];

  @override
  Future<Product> create(ProductDraft draft) async =>
      Product(id: 1, name: draft.name);
}
```

A fake repository supports UI/controller tests without network or Django. Keep fake behavior aligned with expected success/error cases; use integration tests for the real HTTP contract.

# Advanced Example

Use a cache-aware repository with explicit freshness policy, conflict handling, and offline queue semantics. Do not silently return stale data as current; expose freshness and sync state.

# Under the Hood

Repository is an architectural boundary, not a Flutter framework feature. It translates data-source details into application-level results and errors.

# Performance Notes

Avoid unnecessary repository layers that only rename methods; add them where they isolate change or enable testing/caching.

# Security Notes

Repositories may hold user data but cannot enforce authorization. Clear user-specific cache on account switch/logout.

# Best Practices

- Define interfaces around feature needs.
- Keep transport DTO mapping centralized.
- Model errors and cache freshness.
- Inject repositories into state/UI layers.

# Common Mistakes

- Putting all business state in the API client.
- Duplicating authorization rules only in Flutter.
- Reusing private cache across users.

# Debugging

Trace repository method, API call, response mapping, cache source, and UI state.

# Interview Questions

1. What problem does repository pattern solve?
2. How does fake repository help?
3. Does a repository enforce server authorization?

# Practice Exercises

1. Introduce repository interface for a list screen.
2. Write UI tests with a fake.
3. Add explicit stale-cache metadata.

# Chapter Summary

Repositories isolate feature data access from Flutter UI and data-source details. They improve testability and enable cache strategies but do not replace DRF security.
