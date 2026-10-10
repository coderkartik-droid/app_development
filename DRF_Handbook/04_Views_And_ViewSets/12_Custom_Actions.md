# Introduction

A custom ViewSet action adds a resource-specific operation beyond standard CRUD, such as publishing an item, archiving a task, or retrieving a summary. DRF's `@action` decorator lets a router generate its route.

# Why this concept exists

Not every resource operation is a simple create/read/update/delete. Custom actions provide named endpoints while keeping related behavior within the ViewSet.

# Real-world analogy

A library record supports standard checkout operations, but may also have a special “renew” service action.

# Internal Working

```text
router discovers @action metadata -> generates route/method mapping
HTTP request -> action method -> policies/serializer/query -> Response
```

The action name becomes `self.action`, useful for selecting serializers/permissions. Detail actions operate on one resource; collection actions operate on the collection.

# Request Lifecycle

Router resolves the custom route and binds allowed methods to the decorated method. DRF dispatch and policy checks run before the action handler.

# Syntax

```python
@action(detail=True, methods=["post"])
def publish(self, request, pk=None):
    ...
```

# Complete Example 1

Add a publish action to an article ViewSet. Assume `Article` has `owner`, `is_published`, and a serializer.

```python
# articles/views.py
from rest_framework import status, viewsets
from rest_framework.decorators import action
from rest_framework.permissions import IsAuthenticated
from rest_framework.response import Response
from .models import Article
from .serializers import ArticleSerializer


class ArticleViewSet(viewsets.ModelViewSet):
    serializer_class = ArticleSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return Article.objects.filter(owner=self.request.user)

    @action(detail=True, methods=["post"], url_path="publish")
    def publish(self, request, pk=None):
        article = self.get_object()
        article.is_published = True
        article.save(update_fields=["is_published"])
        return Response(
            self.get_serializer(article).data,
            status=status.HTTP_200_OK,
        )
```

```python
# articles/urls.py
from rest_framework.routers import SimpleRouter
from .views import ArticleViewSet

router = SimpleRouter()
router.register("articles", ArticleViewSet, basename="article")
urlpatterns = router.urls
```

```python
# config/urls.py
from django.urls import include, path

urlpatterns = [path("api/", include("articles.urls"))]
```

**Line-by-line:** The ViewSet uses standard CRUD; permission requires authentication. `get_queryset` scopes to owner. `@action(detail=True)` tells router this action targets one article; `methods` permits POST; `url_path` sets route suffix. `get_object()` performs scoped lookup and object checks. The state changes and persists; response uses configured serializer.

**Request → Router → View/ViewSet → Serializer → Database → Response flow:** POST `/articles/{pk}/publish/` -> router binds `publish` -> permission -> scoped object lookup -> update database -> serialize -> response. **Internal working:** `self.action` is `"publish"` and router derives a route name from basename/action. **Professional use case:** Domain command with explicit method. Removing `get_queryset` scoping can let users publish others' articles; for real transitions, use a transaction and enforce allowed state changes.

# Complete Example 2

A collection action can return a summary:

```python
@action(detail=False, methods=["get"], url_path="summary")
def summary(self, request):
    total = self.get_queryset().count()
    return Response({"count": total})
```

`detail=False` creates a collection route such as `/articles/summary/`. The queryset remains owner-scoped. For complex aggregates, calculate in SQL rather than loading all rows.

# Complete Example 3

Customize permission by action:

```python
def get_permissions(self):
    if self.action == "public_feed":
        permission_classes = [AllowAny]
    else:
        permission_classes = [IsAuthenticated]
    return [permission() for permission in permission_classes]
```

This must include imports and be placed on a ViewSet; make the public action explicit and test every action. An incorrect default branch can unintentionally expose the whole ViewSet.

# Advanced Example

Use a dedicated serializer for action request data when it accepts input. Action endpoints still need input validation, authorization, idempotency/concurrency rules, and accurate status semantics; `@action` only creates routing metadata.

# Under the Hood

The decorator annotates the method with route metadata including detail flag, methods, URL path/name, and optional policies. Router inspects metadata and creates patterns/action maps.

# Performance Notes

Avoid loading large collections for summaries; use aggregate queries. Keep action work bounded and transaction scope appropriate.

# Best Practices

- Name action after domain intent, not implementation detail.
- Specify detail vs collection and allowed methods.
- Use `get_object`/`get_queryset` for scoped access.
- Validate action input with a serializer.

# Common Mistakes

- Forgetting `@action`.
- Using wrong `detail` value.
- Assuming action inherits correct permission without checking overrides.
- Creating custom action for ordinary CRUD behavior.

# Debugging

Inspect router URL list, `self.action`, method map, permission selection, and lookup scope. Reverse the generated action URL in tests.

# Interview Questions

1. What does `@action` do? Marks custom ViewSet route metadata.
2. Meaning of `detail=True`? Action targets an individual resource.
3. Does it validate request data? No, use serializer/parser as normal.
4. How router creates action route? It inspects decorator metadata.

# Practice Exercises

1. Add a detail `archive` POST action with state validation.
2. Add a collection `summary` GET action using an aggregate.
3. Test permissions and unsupported methods.

# Chapter Summary

Custom actions model resource-specific operations beyond CRUD. Router generation is convenient, but validation, state rules, authorization, database consistency, and tests remain the developer's responsibility.
