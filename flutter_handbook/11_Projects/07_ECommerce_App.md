# Project Overview

Build a tiny product catalog with a shopping cart. This educational prototype stores products in a local constant list and cart quantities in widget state. It does not collect payment or create real orders.

# Features

- Product cards with name, price, and category
- Add products to an in-memory cart
- Increment/decrement quantities
- Cart count and subtotal
- Empty-cart state

# Folder Structure

```text
lib/
  main.dart
  models/product.dart
  data/catalog_repository.dart
  state/cart_state.dart
  screens/catalog_page.dart
  screens/cart_page.dart
```

# ASCII UI Preview

```text
+--------------------------+
| Shop              Cart 2 |
|--------------------------|
| Canvas backpack   $42.00 |
| Everyday carry          |
|                  [Add]   |
| Ceramic mug      $12.50  |
|                  [Add]   |
| Subtotal: $54.50         |
+--------------------------+
```

# Step-by-Step Development Plan

1. Create immutable product data with stable IDs and integer cents.
2. Track quantities by product ID.
3. Derive cart count and subtotal instead of storing duplicate totals.
4. Add clear empty-cart and quantity controls.
5. Introduce a trusted order flow only when a real backend exists.

# Complete Runnable Code

Use this in `lib/main.dart`. It needs no external packages and performs no real checkout.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const ShopApp());

class ShopApp extends StatelessWidget {
  const ShopApp({super.key});

  @override
  Widget build(BuildContext context) => MaterialApp(
        title: 'Shop demo',
        theme: ThemeData(colorSchemeSeed: Colors.deepOrange, useMaterial3: true),
        home: const ShopPage(),
      );
}

class Product {
  const Product({
    required this.id,
    required this.name,
    required this.category,
    required this.priceCents,
  });
  final int id;
  final String name;
  final String category;
  final int priceCents;
}

class ShopPage extends StatefulWidget {
  const ShopPage({super.key});

  @override
  State<ShopPage> createState() => _ShopPageState();
}

class _ShopPageState extends State<ShopPage> {
  static const _products = [
    Product(id: 1, name: 'Canvas backpack', category: 'Everyday carry', priceCents: 4200),
    Product(id: 2, name: 'Ceramic mug', category: 'Home', priceCents: 1250),
    Product(id: 3, name: 'Desk lamp', category: 'Home', priceCents: 3599),
  ];
  final Map<int, int> _cart = {};

  int get _itemCount => _cart.values.fold(0, (sum, quantity) => sum + quantity);

  int get _subtotal => _products.fold(
        0,
        (sum, product) =>
            sum + product.priceCents * (_cart[product.id] ?? 0),
      );

  String _money(int cents) =>
      '\$${cents ~/ 100}.${(cents % 100).toString().padLeft(2, '0')}';

  void _changeQuantity(int productId, int delta) {
    setState(() {
      final next = (_cart[productId] ?? 0) + delta;
      if (next <= 0) {
        _cart.remove(productId);
      } else {
        _cart[productId] = next;
      }
    });
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(
          title: const Text('Shop'),
          actions: [
            Padding(
              padding: const EdgeInsets.all(16),
              child: Center(child: Text('Cart $_itemCount')),
            ),
          ],
        ),
        body: ListView(
          padding: const EdgeInsets.all(16),
          children: [
            for (final product in _products)
              Card(
                child: ListTile(
                  title: Text(product.name),
                  subtitle: Text('${product.category} · ${_money(product.priceCents)}'),
                  trailing: _cart.containsKey(product.id)
                      ? Row(
                          mainAxisSize: MainAxisSize.min,
                          children: [
                            IconButton(
                              tooltip: 'Remove one',
                              onPressed: () => _changeQuantity(product.id, -1),
                              icon: const Icon(Icons.remove),
                            ),
                            Text('${_cart[product.id]}'),
                            IconButton(
                              tooltip: 'Add one',
                              onPressed: () => _changeQuantity(product.id, 1),
                              icon: const Icon(Icons.add),
                            ),
                          ],
                        )
                      : FilledButton(
                          onPressed: () => _changeQuantity(product.id, 1),
                          child: const Text('Add'),
                        ),
                ),
              ),
            const SizedBox(height: 12),
            Text('Subtotal: ${_money(_subtotal)}',
                style: Theme.of(context).textTheme.titleLarge),
            if (_cart.isEmpty) const Text('Your cart is empty.'),
          ],
        ),
      );
}
```

# Line-by-Line Explanation

- `Product` is immutable and uses a stable ID; `priceCents` keeps arithmetic in integer minor units.
- `_products` is a static sample catalog, not a live product feed.
- `_cart` maps product IDs to quantities.
- `_itemCount` and `_subtotal` are derived getters, preventing duplicate state from becoming inconsistent.
- `_changeQuantity` updates the selected product quantity and removes entries when quantity reaches zero.
- The app bar displays total unit count.
- Product cards show descriptive data and context-sensitive controls: add, or decrement/count/increment.
- `_money` converts cents into a simple dollar string; production formatting should use locale-aware currency formatting.

Removing the product-ID map makes quantity updates ambiguous. Storing subtotal separately would require carefully updating it for every cart mutation. This client-side amount is only a display estimate; it is not an authoritative price.

# Widget Tree

```text
ShopApp > MaterialApp > ShopPage > Scaffold
├── AppBar > Cart count
└── ListView
    ├── Product Card > ListTile > product details + cart controls
    ├── Subtotal
    └── Empty-cart message when appropriate
```

# State Management Used

Local state holds cart quantities because the prototype has one screen. A multi-screen shop should use a cart controller/provider and a repository boundary, with immutable state updates and tests.

# API/Database Used (if applicable)

None. Catalog and cart are in-memory examples. A real commerce app needs server-authoritative prices, inventory, order validation, and a compliant payment provider; never treat a client subtotal as proof of payment.

# Common Mistakes

- Using floating-point prices.
- Trusting client-calculated totals as final order amounts.
- Forgetting to remove zero-quantity products from the cart.
- Claiming the sample performs a real checkout.

# Improvements

- Add product detail, search, sorting, and cart screen.
- Persist a draft cart in a local database.
- Localize currency display and handle currencies with different minor-unit rules.
- Add a server-backed order workflow with duplicate-submit protection.

# Challenge Version

Implement a cart screen and saved cart, then model checkout as an explicit pending/confirmed/failed order state. Make the server authoritative for price, stock, and final total.

# Summary

This project practices stable product identity, derived totals, integer money representation, and cart quantity changes while clearly separating a demo cart from real commerce.
