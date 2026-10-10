# Introduction

DRF includes common permission classes such as `AllowAny`, `IsAuthenticated`, `IsAdminUser`, and `IsAuthenticatedOrReadOnly`.

# Why this concept exists

Reusable policy classes express standard access rules consistently and reduce repeated custom logic.

# Real-world analogy

Built-in permissions are standard door rules: open lobby, badge required, staff-only, or public viewing with staff editing.

# Internal Working

Each permission class implements `has_permission` and optionally `has_object_permission`. Configure them globally or per view; view-level configuration overrides the global default.

# Authentication Flow (where applicable)

Authentication establishes `request.user`; built-in permission evaluates that identity and request method; the view runs only when policy passes.

# Syntax

```python
from rest_framework.permissions import IsAuthenticatedOrReadOnly
```

# Complete Example 1

```python
from rest_framework.permissions import IsAuthenticatedOrReadOnly
from rest_framework.viewsets import ModelViewSet

class ArticleViewSet(ModelViewSet):
    queryset = Article.objects.filter(published=True)
    serializer_class = ArticleSerializer
    permission_classes = [IsAuthenticatedOrReadOnly]
```

Anyone may use safe methods; authenticated users may attempt writes. This does not restrict writes to authors or staff, and the queryset only filters published entries. Add object permissions and scoped write rules for production.

# Complete Example 2

```python
from rest_framework.permissions import IsAdminUser
from rest_framework.generics import ListAPIView

class AuditLogView(ListAPIView):
    queryset = AuditLog.objects.all()
    serializer_class = AuditLogSerializer
    permission_classes = [IsAdminUser]
```

`IsAdminUser` checks `user.is_staff`; it does not mean “superuser only.” Audit records are sensitive, so consider stricter role policies, logging, pagination, and tenant scope.

# Complete Example 3

```python
REST_FRAMEWORK = {
    "DEFAULT_PERMISSION_CLASSES": [
        "rest_framework.permissions.IsAuthenticated",
    ],
}
```

This makes authentication required by default. Public endpoints must explicitly set `AllowAny`. Do not leave a project-wide default open by accident.

# Advanced Example

Use built-ins as building blocks; create a custom permission when the rule is domain-specific, such as manager-of-department or owner-or-collaborator. Keep policy readable and testable.

# Under the Hood

`APIView.initial()` calls `check_permissions`; each configured class must allow the request. Object checks happen only when the view explicitly invokes object permission handling.

# Security Notes

`IsAuthenticatedOrReadOnly` treats any authenticated user as writable; it does not impose ownership. `IsAdminUser` uses the staff flag, which must itself be protected.

# Best Practices

- Choose the narrowest fitting built-in class.
- Keep a secure global default.
- Add queryset scoping and object rules as needed.
- Test the method/user matrix.

# Common Mistakes

- Using `IsAuthenticatedOrReadOnly` for owner-only edits.
- Assuming `IsAdminUser` means superuser.
- Forgetting class-level permission does not filter each list row.

# Debugging

Check effective permission class (view overrides global), user flags, request method, and object check behavior.

# Interview Questions

1. What does `AllowAny` do?
2. What does `IsAdminUser` check?
3. Does `IsAuthenticatedOrReadOnly` enforce ownership?

# Practice Exercises

1. Select permissions for public catalogue and private orders.
2. Test staff versus superuser behavior.
3. Add owner enforcement beyond `IsAuthenticatedOrReadOnly`.

# Chapter Summary

Built-in permission classes handle common request-level policies, but object visibility and domain authorization often need additional queryset and custom permission logic.
