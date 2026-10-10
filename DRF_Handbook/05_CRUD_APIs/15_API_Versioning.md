# Introduction

API versioning lets an API evolve while existing clients continue using a compatible contract. A version identifies a set of routes, fields, and behavioral expectations.

# Why this concept exists

Mobile clients may remain installed for months. Breaking a response shape immediately can disrupt users who cannot upgrade at the same time as the server.

# Real-world analogy

Versioning is publishing a new edition of a form while continuing to accept the previous edition during a migration period.

# Internal Working

DRF can determine version from URL path, query parameter, hostname, or media type. The selected version is available as `request.version`; views/serializers can choose compatible behavior.

# Request Lifecycle

Request version signal -> DRF versioning scheme -> `request.version` -> route/schema/representation selection -> response with same contract family.

# Syntax

```python
REST_FRAMEWORK = {"DEFAULT_VERSIONING_CLASS": "rest_framework.versioning.URLPathVersioning"}
```

# Complete Example 1

```python
# urls.py
from django.urls import include, path

urlpatterns = [
    path("api/<str:version>/", include("catalog.urls")),
]
```

```python
# settings.py
REST_FRAMEWORK = {
    "DEFAULT_VERSIONING_CLASS": "rest_framework.versioning.URLPathVersioning",
    "ALLOWED_VERSIONS": ["v1", "v2"],
    "DEFAULT_VERSION": "v1",
}
```

The version segment appears in each API URL, for example `/api/v1/products/`. Configure the scheme and supported versions together. Versioning is a compatibility policy, not just a URL decoration; document deprecation and removal timelines.

# Complete Example 2

```python
from rest_framework.response import Response
from rest_framework.views import APIView

class ProfileView(APIView):
    def get(self, request):
        if request.version == "v1":
            return Response({"full_name": request.user.get_full_name()})
        return Response({"name": {
            "first": request.user.first_name,
            "last": request.user.last_name,
        }})
```

The example demonstrates a response contract difference. In production, keep version branches small and tested; separate serializers are often clearer. This endpoint assumes authenticated user configuration and a versioning class is active.

# Complete Example 3

```python
response = APIClient().get("/api/v1/profile/")
assert response.status_code == 200
assert "full_name" in response.data
```

This checks the v1 shape. Add a v2 test and tests for unsupported versions. Version behavior should be explicit rather than silently falling back to an unintended contract.

# Advanced Example

Use schema generation that describes each version and provide a migration window, telemetry on old-version use, and a communicated sunset date. Avoid creating a new version for backward-compatible additive changes unless policy requires it.

# Under the Hood

The configured versioning class extracts version during request initialization/resolution. It may also generate version-aware reverse URLs. Exact configuration differs by scheme; verify URL patterns and schema integration.

# Performance Notes

Version checks are inexpensive; maintaining many divergent code paths is the larger long-term cost. Remove obsolete versions after clients have migrated and policy permits.

# Best Practices

- Version only for compatibility-breaking changes.
- Define allowed/default versions and deprecation policy.
- Test each supported contract.
- Keep business logic shared; isolate representation differences.

# Common Mistakes

- Changing a response without versioning or migration notice.
- Maintaining unbounded old versions.
- Assuming URL versioning automatically versions documentation.

# Debugging

Check configured scheme, URL/query/header signal, `request.version`, allowed versions, reverse URLs, and schema output.

# Interview Questions

1. Why version an API? Preserve compatibility across clients.
2. What are common version signals? URL path, query parameter, hostname, media type.
3. Should every change create a new version? Usually only breaking changes.

# Practice Exercises

1. Add URL-path v1/v2 routing and test both shapes.
2. Write a deprecation policy and telemetry plan.
3. Separate version-specific serialization from shared business logic.

# Chapter Summary

Versioning manages contract evolution, especially for long-lived clients. Choose a clear strategy, test every supported version, and plan deprecation rather than allowing versions to diverge indefinitely.
