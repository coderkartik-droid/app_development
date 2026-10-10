# Introduction

DRF settings provide project-wide defaults for API behavior, including authentication, permissions, renderers, parsers, pagination, and exception handling. Individual views can override many defaults.

# Why this concept exists

Without shared defaults, each endpoint may behave differently by accident. Central configuration makes policy consistent, while per-view overrides support deliberate exceptions.

# Real-world analogy

API settings are the organization's default operating policy; a specific endpoint can receive an approved exception when its purpose requires one.

# Internal Working

DRF reads its settings through Django's configured settings module. A view-specific attribute generally takes precedence for that view; global defaults are applied when no local override exists.

```text
settings.py REST_FRAMEWORK defaults -> APIView policy resolution
                                      -> endpoint overrides
```

# Request Lifecycle (where applicable)

During API view initialization, DRF resolves authentication, permission, parser, renderer, and throttle classes. These affect request processing before/around the handler. Pagination is used by the relevant generic views or configured pagination flow.

# Syntax

```python
REST_FRAMEWORK = {
    "DEFAULT_PERMISSION_CLASSES": [
        "rest_framework.permissions.IsAuthenticated",
    ],
}
```

# Complete Example 1

Configure a restrictive default permission and JSON output:

```python
# config/settings.py
REST_FRAMEWORK = {
    "DEFAULT_PERMISSION_CLASSES": [
        "rest_framework.permissions.IsAuthenticated",
    ],
    "DEFAULT_RENDERER_CLASSES": [
        "rest_framework.renderers.JSONRenderer",
    ],
}
```

Use an explicit public exception only when intended:

```python
# api/views.py
from rest_framework.permissions import AllowAny
from rest_framework.response import Response
from rest_framework.views import APIView


class PublicHealthView(APIView):
    permission_classes = [AllowAny]

    def get(self, request):
        return Response({"status": "ok"})
```

**Line-by-line:** `REST_FRAMEWORK` is a settings dictionary. The permission default requires authenticated access for views that use defaults. JSON renderer limits output format. `permission_classes` overrides the global default for this public health endpoint. **Flow:** Django loads settings -> DRF resolves view policy -> permission check -> handler -> renderer. **Internal working:** Defaults reduce repetition but can have broad effects. **Professional tip:** Start with least privilege and review public exceptions. Removing the global permission setting may make views fall back to DRF's defaults, which can be permissive depending on configuration.

# Complete Example 2

Set pagination defaults:

```python
REST_FRAMEWORK = {
    "DEFAULT_PAGINATION_CLASS":
        "rest_framework.pagination.PageNumberPagination",
    "PAGE_SIZE": 25,
}
```

Generic list views can use the configured paginator to bound result pages. **Flow:** list queryset -> paginator slices/counts -> response includes page metadata. **Tip:** Consider count-query cost and maximum page size; clients should not control unbounded limits.

# Complete Example 3

Set parser defaults explicitly:

```python
REST_FRAMEWORK = {
    "DEFAULT_PARSER_CLASSES": [
        "rest_framework.parsers.JSONParser",
        "rest_framework.parsers.FormParser",
        "rest_framework.parsers.MultiPartParser",
    ],
}
```

This controls accepted request representations. Configure only formats your endpoints need; parser defaults are distinct from response renderer defaults.

# Advanced Example

Use settings validation during startup/CI to ensure production has deliberate authentication and permission policies. Keep environment-specific secrets outside source code; DRF settings are configuration, not secret storage.

# Under the Hood

DRF's settings wrapper exposes configured values and defaults. Class paths are imported when needed; view attributes provide local overrides. Incorrect import paths or malformed setting shapes can fail at runtime.

# Performance Notes

Global settings have little direct cost. Authentication backends, pagination counts, parser choice, and serializer/query design can affect endpoint runtime.

# Best Practices

- Define secure global permission defaults.
- Configure parsers/renderers/pagination consistently.
- Document and test per-view overrides.
- Keep settings declarative and environment-aware.

# Common Mistakes

- Assuming an endpoint is protected without checking effective permissions.
- Making a broad `AllowAny` exception.
- Confusing parser settings with renderer settings.
- Returning unbounded lists because pagination was omitted.

# Debugging

Inspect the active settings module, `REST_FRAMEWORK` dictionary, view attributes, effective authentication/permission classes, and response behavior. Run tests under the same settings used in deployment.

# Interview Questions

1. What can DRF settings configure? API-wide defaults such as permissions, authentication, parsers, renderers, and pagination.
2. Can views override defaults? Yes, many policies can be set per view.
3. Why set restrictive permissions globally? To avoid accidental public access.
4. Are settings a secret store? No.

# Practice Exercises

1. Configure a global authenticated-only permission.
2. Create one deliberate public health route.
3. Test anonymous behavior on both endpoints.

# Chapter Summary

DRF settings centralize API defaults; view-level declarations override them deliberately. Effective security and behavior must be tested, not inferred from configuration appearance.
