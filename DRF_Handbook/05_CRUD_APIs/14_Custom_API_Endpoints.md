# Introduction

Custom endpoints represent operations that do not fit ordinary resource CRUD, such as publishing an article, approving an order, or calculating a quote.

# Why this concept exists

Real business behavior is not always a simple field edit. A named operation can enforce domain rules and communicate intent clearly.

# Real-world analogy

Instead of allowing a customer to edit an order's internal `paid` flag, provide a checkout operation that performs the complete payment transition.

# Internal Working

A custom route invokes a view action or APIView method, checks permissions, validates operation-specific input, performs domain work (often transactionally), then returns an explicit result.

# Request Lifecycle

Custom route -> DRF action -> permission -> operation serializer -> domain/service logic -> database transaction -> response.

# Syntax

```python
@action(detail=True, methods=["post"])
def publish(self, request, pk=None):
    ...
```

# Complete Example 1

```python
from rest_framework import serializers, status, viewsets
from rest_framework.decorators import action
from rest_framework.permissions import IsAuthenticated
from rest_framework.response import Response
from .models import Article
from .serializers import ArticleSerializer

class PublishInput(serializers.Serializer):
    confirm = serializers.BooleanField()

class ArticleViewSet(viewsets.ModelViewSet):
    serializer_class = ArticleSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return Article.objects.filter(owner=self.request.user)

    @action(detail=True, methods=["post"], url_path="publish")
    def publish(self, request, pk=None):
        article = self.get_object()
        data = PublishInput(data=request.data)
        data.is_valid(raise_exception=True)
        if not data.validated_data["confirm"]:
            return Response({"detail": "Confirmation required."},
                            status=status.HTTP_400_BAD_REQUEST)
        article.status = Article.Status.PUBLISHED
        article.save(update_fields=["status"])
        return Response(self.get_serializer(article).data)
```

The action is detail-scoped (`pk` identifies one article) and uses POST for a state-changing command. `get_object()` honors the owner queryset and object permissions. The input serializer validates confirmation. The model enum is assumed to exist; production should enforce allowed transitions and audit actor/time, preferably through a domain service or atomic transaction.

# Complete Example 2

```python
from django.urls import path
from rest_framework.views import APIView
from rest_framework.response import Response

class HealthView(APIView):
    authentication_classes = []
    permission_classes = []

    def get(self, request):
        return Response({"status": "ok"})

urlpatterns = [path("health/", HealthView.as_view(), name="health")]
```

This is a complete minimal custom read endpoint. Policy is explicitly public because health checks often need unauthenticated access; avoid exposing sensitive dependency details. `APIView` provides DRF request/response handling.

# Complete Example 3

```python
response = APIClient().post("/api/articles/5/publish/",
                            {"confirm": True}, format="json")
assert response.status_code in (200, 404, 403)
```

The request shape tests route and payload parsing, but production tests should authenticate an owner and assert exactly 200, then separately verify non-owner denial/not-found and invalid confirmation.

# Advanced Example

Move complex domain rules into a service function so API, background jobs, and admin workflows share one implementation. Use transactions and idempotency where retries could duplicate effects.

# Under the Hood

Routers discover `@action` metadata and create URL patterns/method mappings. The action remains part of the ViewSet lifecycle, including authentication, permissions, and exception handling.

# Performance Notes

Do not make expensive synchronous integrations inside a request without timeouts and failure policy. Use a task queue for long-running work and return an operation identifier.

# Best Practices

- Name the action as a domain verb.
- Use an appropriate HTTP method and input serializer.
- Scope the object and enforce transition rules.
- Keep external side effects idempotent/retry-safe.

# Common Mistakes

- Using a custom endpoint to bypass serializers or permissions.
- Exposing arbitrary state mutation.
- Returning success before the state change is durable.

# Debugging

Verify generated action URL, method list, permission classes, `get_object`, serializer errors, and transaction outcome.

# Interview Questions

1. When create a custom action? For a domain operation beyond standard CRUD.
2. What does `detail=True` mean? The action targets a single resource.
3. Why use a domain service? Share rules across entry points and keep views orchestration-focused.

# Practice Exercises

1. Add an owner-only `archive` action.
2. Add a custom input serializer and transition validation.
3. Test duplicate/retried action requests.

# Chapter Summary

Custom endpoints should express real operations, not expose arbitrary model mutation. Keep routing, validation, authorization, domain rules, and transactional side effects explicit.
