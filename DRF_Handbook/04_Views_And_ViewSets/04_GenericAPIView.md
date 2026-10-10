# Introduction

`GenericAPIView` extends APIView with common resource facilities: `queryset`, `serializer_class`, `get_queryset()`, `get_serializer()`, lookup configuration, and pagination/filter hooks. It is the base for DRF's generic concrete views.

# Why this concept exists

Many endpoints repeat object lookup, serializer selection, pagination, and queryset behavior. GenericAPIView centralizes these mechanisms while still allowing custom handler methods or mixins.

# Real-world analogy

GenericAPIView is a reusable service-counter framework with standard drawers for records and forms; concrete views add the specific operation.

# Internal Working

```text
GenericAPIView -> get_queryset -> filter/lookup -> get_serializer
               -> mixin action -> Response
```

Always call `get_queryset()` rather than directly using `self.queryset` in request logic; DRF can clone/re-evaluate it safely and subclasses can scope it.

# Request Lifecycle

The APIView lifecycle runs first. A mixin handler then asks GenericAPIView for the queryset, object, serializer, pagination, and response helpers.

# Syntax

```python
class BookList(GenericAPIView, ListModelMixin):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
```

# Complete Example 1

Combine GenericAPIView with a list mixin. Assume a `Book` model and serializer exist.

```python
# books/views.py
from rest_framework import generics, mixins
from .models import Book
from .serializers import BookSerializer


class BookList(mixins.ListModelMixin, generics.GenericAPIView):
    serializer_class = BookSerializer

    def get_queryset(self):
        return Book.objects.filter(is_published=True).order_by("title")

    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)
```

```python
# books/urls.py
from django.urls import path
from .views import BookList

urlpatterns = [path("books/", BookList.as_view(), name="book-list")]
```

```python
# config/urls.py
from django.urls import include, path

urlpatterns = [path("api/", include("books.urls"))]
```

**Line-by-line:** `ListModelMixin` provides `list`; GenericAPIView provides serializer/queryset helpers and APIView lifecycle. `serializer_class` selects representation. `get_queryset` scopes to published books and orders. `get` delegates to mixin. URL registers the callable.

**Request → Router → View/ViewSet → Serializer → Database → Response flow:** route -> APIView dispatch -> GET -> list mixin calls `get_queryset` -> queryset executes -> serializer maps rows -> paginated/standard Response -> renderer. **Internal execution:** Generic helpers provide context and lookup behavior. **Professional use case:** Custom list behavior with reusable DRF pagination/serializer pipeline. Removing `get_queryset` may expose unpublished rows if replaced with an unrestricted queryset.

# Complete Example 2

Use built-in `get_object()` for detail lookup:

```python
class BookDetail(generics.GenericAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
    lookup_field = "slug"

    def get(self, request, *args, **kwargs):
        book = self.get_object()
        return Response(self.get_serializer(book).data)
```

`get_object` applies filter backends and lookup configuration, then raises 404 when absent. Use `get_queryset` for caller-specific visibility; lookup is not authorization by itself.

# Complete Example 3

Override serializer context:

```python
def get_serializer_context(self):
    context = super().get_serializer_context()
    context["include_summary"] = True
    return context
```

The superclass includes standard context; adding a value makes it available to the serializer. Removing `super()` can discard request/view/format context.

# Advanced Example

GenericAPIView can be composed with different mixins to build a narrow endpoint. If the code recreates standard CRUD lifecycle manually, consider a concrete generic view instead.

# Under the Hood

GenericAPIView's `get_queryset` returns a request-usable queryset; `filter_queryset` applies configured backends; `get_object` performs lookup and object permission checks; `get_serializer` instantiates the configured serializer with context.

# Performance Notes

Filter and optimize queryset before serialization. Use `select_related` and `prefetch_related` based on serializer access; pagination limits work and response size.

# Best Practices

- Use `get_queryset` for request-aware filtering.
- Use `get_object` to retain lookup/permission behavior.
- Keep serializer selection/context explicit.
- Compose only necessary mixins.

# Common Mistakes

- Evaluating class-level queryset directly.
- Bypassing `get_object` and object permission checks.
- Forgetting `super()` when extending context.
- Including a list mixin without a handler method.

# Debugging

Inspect `get_queryset`, filter backends, lookup kwarg/field, serializer context, and pagination. Trace mixin handler calls.

# Interview Questions

1. What does GenericAPIView add? Queryset, serializer, lookup, pagination/filter helpers.
2. Why use `get_queryset`? Safe overridable request-scoped query behavior.
3. What does `get_object` do? Filtered lookup and object permission checking.
4. Does GenericAPIView implement list by itself? No; combine a mixin or concrete view.

# Practice Exercises

1. Compose list mixin and GenericAPIView.
2. Add a slug-based lookup and owner-scoped queryset.
3. Extend serializer context while preserving superclass values.

# Chapter Summary

GenericAPIView provides reusable resource mechanics without choosing every action. Combine it with mixins for custom compositions, and preserve its queryset, serializer, lookup, and permission hooks.
