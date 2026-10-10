# Project Overview

Build an e-commerce API for product discovery, carts, orders, and order state. Financial and inventory values remain server-controlled.

# Features

- Product catalogue with filtering/search/order.
- Authenticated cart/order creation.
- Server-calculated totals.
- Inventory reservation and order state transitions.
- Customer-scoped order history.

# Folder Structure

```text
commerce/
  models.py
  serializers.py
  services.py
  permissions.py
  views.py
  urls.py
  tests.py
```

# API Endpoints

| Method | Route | Purpose |
|---|---|---|
| GET | `/api/v1/products/` | Public catalogue |
| GET | `/api/v1/products/{id}/` | Product detail |
| POST | `/api/v1/orders/` | Create order from product quantities |
| GET | `/api/v1/orders/` | Customer's orders |
| POST | `/api/v1/orders/{id}/cancel/` | Cancel eligible order |

# Database Design

Product stores SKU, price, active state, and inventory count. Order belongs to customer and has status/total. OrderItem snapshots product name and unit price at purchase time, preserving historical truth.

# Models

```python
from django.conf import settings
from django.db import models

class Product(models.Model):
    sku = models.CharField(max_length=40, unique=True)
    name = models.CharField(max_length=160)
    price = models.DecimalField(max_digits=10, decimal_places=2)
    stock = models.PositiveIntegerField(default=0)
    active = models.BooleanField(default=True)

class Order(models.Model):
    customer = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.PROTECT)
    status = models.CharField(max_length=20, default="pending")
    total = models.DecimalField(max_digits=12, decimal_places=2)
    created_at = models.DateTimeField(auto_now_add=True)

class OrderItem(models.Model):
    order = models.ForeignKey(Order, on_delete=models.CASCADE, related_name="items")
    product = models.ForeignKey(Product, on_delete=models.PROTECT)
    product_name = models.CharField(max_length=160)
    unit_price = models.DecimalField(max_digits=10, decimal_places=2)
    quantity = models.PositiveIntegerField()
```

# Serializers

```python
from rest_framework import serializers
from .models import Product, Order, OrderItem

class ProductSerializer(serializers.ModelSerializer):
    class Meta:
        model = Product
        fields = ["id", "sku", "name", "price"]

class OrderItemInputSerializer(serializers.Serializer):
    product_id = serializers.IntegerField(min_value=1)
    quantity = serializers.IntegerField(min_value=1, max_value=50)

class OrderSerializer(serializers.ModelSerializer):
    class Meta:
        model = Order
        fields = ["id", "status", "total", "created_at"]
        read_only_fields = fields
```

# Views

```python
from django.db import transaction
from rest_framework import generics, serializers
from rest_framework.permissions import AllowAny, IsAuthenticated
from .models import Product, Order
from .serializers import ProductSerializer, OrderSerializer

class ProductListView(generics.ListAPIView):
    queryset = Product.objects.filter(active=True).order_by("name", "pk")
    serializer_class = ProductSerializer
    permission_classes = [AllowAny]

class OrderListView(generics.ListAPIView):
    serializer_class = OrderSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return Order.objects.filter(customer=self.request.user).order_by("-created_at", "pk")
```

# URLs

```python
urlpatterns = [
    path("products/", ProductListView.as_view(), name="product-list"),
    path("orders/", OrderListView.as_view(), name="order-list"),
]
```

# Authentication

Catalogue reads are public; order endpoints require a user. Use JWT or session auth appropriate to the client and never accept customer identity from request JSON.

# Permissions

Order history uses customer-scoped queryset. Cancellation should call `get_object()` from that queryset, validate pending status, and perform inventory/state changes through a transaction-safe service.

# Testing

Test public catalogue excludes inactive items, anonymous order access is denied, customer isolation, price/total cannot be spoofed, insufficient stock, duplicate retry, and cancellation invariants.

# Deployment

Use PostgreSQL constraints/transactions, HTTPS, secrets manager for payment credentials, background processing for payment integrations, backups, and monitored queue/database capacity.

# Complete Source Code

The snippets define data contracts and read endpoints. A production order-creation service should lock stock rows and calculate prices server-side:

```python
@transaction.atomic
def create_order(customer, lines):
    order = Order.objects.create(customer=customer, total=0)
    total = Decimal("0.00")
    for line in lines:
        product = Product.objects.select_for_update().get(
            pk=line["product_id"], active=True
        )
        if product.stock < line["quantity"]:
            raise InsufficientStock()
        product.stock -= line["quantity"]
        product.save(update_fields=["stock"])
        OrderItem.objects.create(
            order=order, product=product, product_name=product.name,
            unit_price=product.price, quantity=line["quantity"]
        )
        total += product.price * line["quantity"]
    order.total = total
    order.save(update_fields=["total"])
    return order
```

# Line-by-Line Explanation

- Product price/stock are authoritative server fields.
- OrderItem snapshots name and price to preserve purchase history.
- Input serializer accepts IDs and quantities, not unit price or total.
- `select_for_update()` locks inventory rows during stock checks.
- The transaction makes stock decrement, item creation, and total update atomic.
- Order reads are restricted to the authenticated customer.

**Flow:** Client -> auth -> order input validation -> transaction/row locks -> DB writes -> serialized receipt. Payment provider calls should occur outside the DB transaction through idempotent workflow/outbox.

# Best Practices

- Use decimal currency and immutable purchase snapshots.
- Apply database constraints for SKU and quantity invariants.
- Use idempotency keys for checkout.
- Audit state transitions and refund actions.

# Common Mistakes

- Trusting client-submitted prices or totals.
- Decrementing stock without locking/conditional update.
- Calling a payment gateway inside a long-held database transaction.

# Improvements

Add payment intents, coupon rules, shipping/tax calculations, outbox events, inventory reservation expiry, and staff order management.

# Challenge Version

Implement a resilient checkout state machine with retries, payment reconciliation, idempotency, compensation for failed payment, and concurrent inventory tests.

# Summary

E-commerce APIs require strict server authority over price, inventory, customer identity, and order transitions. Transactions, row locks, snapshots, and idempotency protect business invariants.
