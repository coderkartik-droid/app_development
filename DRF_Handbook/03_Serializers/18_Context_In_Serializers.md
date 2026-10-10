# Introduction

Serializer context is a dictionary of extra information available to serializer fields and methods. Generic DRF views commonly provide the current request, view, and format.

# Why this concept exists

Some representation or validation decisions depend on request context, such as the current user or a URL base. Context passes that information without adding internal values to client input.

# Real-world analogy

Context is an instruction folder handed to a form processor: it contains trusted situation details, such as the current operator, that are not part of the submitted form.

# Internal Working

```text
APIView -> get_serializer_context() -> Serializer(context=...)
                                   -> field/method reads self.context
```

Context is server-supplied. It should not be confused with validated request fields, and a serializer used directly must receive context explicitly if it depends on it.

# Request Lifecycle (where applicable)

Generic views instantiate serializers with context. Custom views can pass `context={"request": request}`. Serializer methods read the context during validation or representation.

# Syntax

```python
serializer = ItemSerializer(
    instance=item,
    context={"request": request},
)
```

# Complete Example 1

Return an absolute image URL only when a request is available:

```python
# catalog/serializers.py
from rest_framework import serializers
from .models import Product


class ProductSerializer(serializers.ModelSerializer):
    image_url = serializers.SerializerMethodField()

    class Meta:
        model = Product
        fields = ["id", "name", "image_url"]

    def get_image_url(self, obj):
        if not obj.image:
            return None
        request = self.context.get("request")
        url = obj.image.url
        return request.build_absolute_uri(url) if request else url
```

```python
# catalog/views.py
from rest_framework import generics
from .models import Product
from .serializers import ProductSerializer


class ProductListView(generics.ListAPIView):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
```

```python
# catalog/urls.py
from django.urls import path
from .views import ProductListView

urlpatterns = [path("products/", ProductListView.as_view())]
```

**Line-by-line:** The method field is output-only. The getter handles absent image; obtains request context if supplied; returns an absolute URI when possible and relative storage URL otherwise. Generic view provides request context by default. **Request → Serializer → Validation → Database → Response:** GET -> queryset -> serializer reads model and context -> representation -> renderer. **Internal working:** Context is not a serialized field. **Professional use case:** Construct request-aware links. Removing context falls back to relative URL; calling `build_absolute_uri` without request would fail.

# Complete Example 2

Pass explicit context in a custom view:

```python
serializer = ProductSerializer(
    products,
    many=True,
    context={"request": request, "currency": "USD"},
)
return Response(serializer.data)
```

The request supports absolute URLs and the currency can inform a representation method. The API must define currency semantics; context is not a substitute for a persistent product currency field.

# Complete Example 3

Use context for a validation rule based on authenticated user:

```python
def validate(self, attrs):
    request = self.context.get("request")
    if request and not request.user.is_staff and attrs.get("featured"):
        raise serializers.ValidationError({"featured": "Only staff may set this."})
    return attrs
```

This is an application rule; a permission or server-controlled field design may be clearer. Never assume context request is present when serializer is instantiated manually.

# Advanced Example

Override `get_serializer_context()` in a view only for trusted server-side values. Keep context small and avoid passing entire service containers or mutable global state.

# Under the Hood

The serializer stores the supplied dictionary as `_context`; fields access the root serializer's context through their binding. Generic view context commonly includes `request`, `format`, and `view`.

# Performance Notes

Context lookup is inexpensive. The data accessed through context may not be; avoid database/network lookups inside method fields merely because request is available.

# Best Practices

- Use context for trusted request/view metadata.
- Provide context explicitly in custom serializer construction.
- Handle missing request where reuse is possible.
- Keep client-supplied values in `initial_data`, not context.

# Common Mistakes

- Expecting manually created serializers to have request context.
- Treating context as validated client data.
- Putting secrets or mutable global state into context.
- Using context-dependent validation without test coverage.

# Debugging

Inspect serializer `.context`, the view's `get_serializer_context`, and invocation path. Compare generic-view and direct-instantiation behavior.

# Interview Questions

1. What is serializer context? Extra server-supplied data passed to serializer.
2. What does generic view usually include? Request, view, and format context.
3. Is context serialized? No, it informs serializer behavior.
4. Why handle missing request? Serializer may be used outside generic views.

# Practice Exercises

1. Build an absolute URL output field with fallback.
2. Manually instantiate the serializer and pass request context.
3. Test context-dependent output and validation.

# Chapter Summary

Context carries trusted runtime information into serializers without expanding client payloads. Generic views supply it; custom code must pass it explicitly and handle its absence appropriately.
