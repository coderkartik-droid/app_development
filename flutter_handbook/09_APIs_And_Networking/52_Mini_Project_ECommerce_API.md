# Introduction

An ecommerce API feature reads product catalogs, supports filters and pagination, displays product details, and submits cart or order actions. It combines read and write request patterns.

# Why this concept exists

Shopping apps must show current prices and availability while protecting customer data and preventing duplicate orders.

# Real-world analogy

A shop's catalog is a changing shelf; the cart is a draft order; checkout is a final transaction confirmed by the store's server.

# Internal Working

Catalog GET requests can be cached and paginated. Product detail requests use stable IDs. Cart state may be local or server-backed. Checkout POST must be authenticated and idempotent. The server, not the client, determines final price, inventory, tax, and authorization.

# Syntax

```dart
final uri = Uri.https('api.example.com', '/v1/products', {
  'category': category,
  'cursor': cursor ?? '',
});
```

# Complete Example 1

This runnable app builds a product catalog UI using a fake repository. The sample prices and inventory are illustrative and not real commerce data.

```dart
import 'package:flutter/material.dart';

class Product {
  const Product({
    required this.id,
    required this.name,
    required this.priceCents,
  });
  final String id;
  final String name;
  final int priceCents;
}

class ProductRepository {
  Future<List<Product>> loadProducts() async {
    await Future<void>.delayed(const Duration(milliseconds: 400));
    return const [
      Product(id: 'p1', name: 'Canvas bag', priceCents: 2499),
      Product(id: 'p2', name: 'Desk lamp', priceCents: 4590),
    ];
  }
}

void main() => runApp(const ShopApp());

class ShopApp extends StatelessWidget {
  const ShopApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: ProductPage());
}

class ProductPage extends StatefulWidget {
  const ProductPage({super.key});

  @override
  State<ProductPage> createState() => _ProductPageState();
}

class _ProductPageState extends State<ProductPage> {
  final _repository = ProductRepository();
  List<Product> _products = [];
  final Set<String> _cart = {};
  bool _loading = true;

  @override
  void initState() {
    super.initState();
    _load();
  }

  Future<void> _load() async {
    final products = await _repository.loadProducts();
    if (!mounted) return;
    setState(() {
      _products = products;
      _loading = false;
    });
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: Text('Shop · cart ${_cart.length}')),
        body: _loading
            ? const Center(child: CircularProgressIndicator())
            : ListView(
                children: _products.map((product) {
                  final added = _cart.contains(product.id);
                  return ListTile(
                    title: Text(product.name),
                    subtitle: Text('\$${(product.priceCents / 100).toStringAsFixed(2)}'),
                    trailing: IconButton(
                      icon: Icon(added ? Icons.check : Icons.add_shopping_cart),
                      onPressed: () => setState(() {
                        if (added) {
                          _cart.remove(product.id);
                        } else {
                          _cart.add(product.id);
                        }
                      }),
                    ),
                  );
                }).toList(),
              ),
      );
}
```

### Line-by-line explanation

- `Product` stores a stable ID and price in integer cents to avoid floating-point currency arithmetic.
- `ProductRepository` simulates an API fetch.
- `_cart` stores product IDs in local UI state for this demo.
- `_load` fetches products and updates state if mounted.
- Prices are formatted from cents for display.
- Cart icon toggles product identity and updates the count.
- The sample omits checkout and real inventory by design.

### Widget tree

```text
ProductPage → Scaffold
            ├→ AppBar(cart count)
            └→ loading or ListView → Product ListTile*
```

### Execution flow

```text
Page startup -> repository catalog request -> typed products -> render
Add/remove -> local cart IDs -> update count/icon
Production checkout -> authenticated idempotent POST -> server confirmation
```

### ASCII UI preview

```text
+---------------------------+
| Shop · cart 1             |
+---------------------------+
| Canvas bag      $24.99 [✓]|
| Desk lamp       $45.90 [+]|
+---------------------------+
```

### Example explanation

The catalog UI is intentionally backed by a fake repository. Currency is stored as integer minor units to avoid arithmetic surprises. A real checkout must send product IDs/quantities, not trust client-calculated totals.

If the client price is treated as authoritative, a modified app can submit a different total. If checkout POST retries without idempotency, duplicate orders can be created.

**Common mistakes:** The server must recalculate price, tax, discounts, and inventory.

# Complete Example 2

Add a product to a server cart:

```dart
await client.post(
  Uri.https('api.example.com', '/v1/cart/items'),
  headers: {'content-type': 'application/json'},
  body: jsonEncode({'productId': productId, 'quantity': 1}),
);
```

Authentication and server validation are required.

# Complete Example 3

Submit checkout with an idempotency key:

```text
POST /v1/orders
Idempotency-Key: unique-checkout-operation-id
```

Reuse the same key only for retries of that checkout attempt.

# Advanced Example

Handle product availability changes, price refresh, inventory reservation, payment authorization, order confirmation, and retry ambiguity as distinct steps. Never store payment card secrets unless using a compliant provider flow.

# Under the Hood

Catalog responses are cached/paginated representations. Cart mutations update server state. Checkout is a business transaction whose final result comes from the backend/payment provider, not client state.

# Performance Notes

- Paginate product listings.
- Resize/cache product images.
- Cache catalog data with freshness rules.
- Avoid per-row detail requests.

# Best Practices

- Use stable product IDs.
- Store currency as integer minor units or precise decimal representation.
- Revalidate price and stock on server.
- Make checkout idempotent.
- Follow payment-provider security/compliance guidance.

# Common Mistakes

- Trusting client totals.
- Duplicate checkout on retry.
- No stale price handling.
- Exposing payment secrets.
- Unbounded product fetch.

# Debugging

- Correlate cart/order IDs and request IDs.
- Verify server totals vs displayed totals.
- Test stock race and duplicate submission.
- Keep payment data out of logs.

# Interview Questions

1. Why store cents as int? Avoid binary floating-point currency rounding issues.
2. Who calculates final order total? Server.
3. Why idempotency key? Prevent duplicate orders from retries.
4. What can be cached? Catalog/product details with freshness policy; not final checkout authority.

# Try It Yourself

1. Add pagination and categories.
2. Persist cart IDs locally.
3. Simulate price change before checkout.
4. Add idempotency key handling.

# Challenge Project

Build a shop flow with paginated products, product details, local/server cart, price refresh, idempotent checkout request, and order-status polling.

# Quick Revision Sheet

- Catalog: GET/cache/paginate.
- Cart: stable IDs and quantities.
- Server validates price, stock, tax, and permissions.
- Checkout needs idempotency.
- Never log or store payment secrets casually.
