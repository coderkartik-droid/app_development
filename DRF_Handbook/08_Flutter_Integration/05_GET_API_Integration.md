# Introduction

GET integration retrieves data from DRF and converts JSON into typed Flutter models for display.

# Why this concept exists

Mobile screens need current server data while handling latency, pagination, errors, and changing response schemas.

# Real-world analogy

Flutter asks the catalogue desk for a page of products, then turns the returned list into readable cards.

# Internal Working

Flutter sends GET; DRF checks policy and evaluates a queryset; serializer produces primitive values; pagination may wrap results; Flutter decodes the envelope and updates state.

# Data Flow

```text
Flutter GET -> DRF list -> scoped queryset -> serializer/paginator
<- JSON page <- Flutter model mapping <- state/UI
```

# Complete Example 1

Flutter model/service:

```dart
class Product {
  const Product({required this.id, required this.name});
  final int id;
  final String name;

  factory Product.fromJson(Map<String, dynamic> json) => Product(
        id: json['id'] as int,
        name: json['name'] as String,
      );
}

Future<List<Product>> loadProducts(http.Client client) async {
  final response = await client.get(Uri.https('api.example.com', '/api/products/'));
  if (response.statusCode != 200) {
    throw StateError('Unable to load products (${response.statusCode})');
  }
  final body = jsonDecode(response.body) as Map<String, dynamic>;
  return (body['results'] as List)
      .map((row) => Product.fromJson(row as Map<String, dynamic>))
      .toList();
}
```

The model defines expected JSON fields. Service checks status, decodes a page-number envelope, maps each row, and returns typed data. Add robust errors, timeout, and pagination metadata; a bare-list backend requires a different decode shape.

# Complete Example 2

DRF list backend:

```python
class ProductListView(ListAPIView):
    serializer_class = ProductSerializer
    permission_classes = [AllowAny]
    pagination_class = PageNumberPagination

    def get_queryset(self):
        return Product.objects.filter(active=True).order_by("name", "pk")
```

The public endpoint only returns active products and uses stable ordering. Configure page size globally or on a custom paginator. Serializer fields must match the Flutter model contract.

# Complete Example 3

```dart
FutureBuilder<List<Product>>(
  future: loadProducts(http.Client()),
  builder: (context, snapshot) {
    if (snapshot.hasError) return const Text('Load failed');
    if (!snapshot.hasData) return const CircularProgressIndicator();
    return ListView(
      children: snapshot.data!
          .map((p) => ListTile(title: Text(p.name)))
          .toList(),
    );
  },
)
```

The builder renders loading/error/data states, but constructing the Future in `build` can refetch on rebuild. Store the Future in state or use a state-management layer and dispose/close resources appropriately.

# Advanced Example

Represent page results as `Page<Product>` with `next`/`previous` URLs and implement incremental loading with duplicate-request prevention and cancellation on screen disposal.

# Under the Hood

The database QuerySet may remain lazy until serializer iteration. Flutter JSON decoding is synchronous CPU work after bytes arrive; very large payloads can block UI and should be paginated.

# Performance Notes

Use server pagination, bounded fields, relation prefetching, and avoid repeated fetches during rebuilds.

# Security Notes

Do not assume a public GET is harmless; constrain exposed fields and apply per-user authorization to private collections.

# Best Practices

- Use typed models and explicit envelope parsing.
- Handle nullability and schema evolution deliberately.
- Include loading, empty, and error UI states.
- Use stable ordering and pagination.

# Common Mistakes

- Assuming paginated responses are lists.
- Casting unvalidated JSON aggressively.
- Recreating Future on every build.

# Debugging

Inspect status/body/content type, response envelope, field types, null values, and DRF serializer output.

# Interview Questions

1. How does Flutter map JSON to a model?
2. What is the common DRF page-number response shape?
3. Why avoid starting requests in every build?

# Practice Exercises

1. Display a paginated product list.
2. Add refresh and empty states.
3. Test malformed and nullable response fields.

# Chapter Summary

GET integration is a complete async pipeline: status handling, envelope parsing, typed mapping, and UI states. Coordinate Flutter model shape with DRF serializer and pagination.
