# Introduction

Django can produce JSON responses, but DRF adds reusable API concepts: structured validation, request parsing, renderer selection, permission hooks, pagination, and test helpers. It is most useful when an application needs consistent API behavior across several endpoints.

# Why this concept exists

Without a shared API framework, each endpoint may implement JSON parsing, validation, status codes, and errors differently. Inconsistent contracts make clients harder to build and failures harder to diagnose.

# Real-world analogy

Without DRF, every service desk invents its own forms and receipt format. DRF provides shared rules so callers know how to submit information and interpret results.

# Internal Working

```text
request bytes -> DRF Request/parser -> view policy -> validation -> application logic
              -> Response(data, status) -> renderer -> response bytes
```

DRF components are composable. A small function view can use decorators; larger resource endpoints can use `APIView`, generic views, or viewsets.

# Request Lifecycle (where applicable)

The client sends a method and representation. DRF dispatch applies authentication and permissions, parses submitted data as required, then runs endpoint logic. The response data is rendered according to acceptable media types.

# Syntax

```python
return Response({"detail": "Created"}, status=201)
```

# Complete Example 1

This complete endpoint shows method restriction and a consistent response. It requires a Django project with DRF installed and `"rest_framework"` registered.

```python
# api/views.py
from rest_framework.decorators import api_view
from rest_framework.response import Response


@api_view(["GET"])
def service_info(request):
    return Response({
        "name": "inventory",
        "version": 1,
        "features": ["catalog", "stock"],
    })
```

```python
# api/urls.py
from django.urls import path
from .views import service_info

urlpatterns = [path("info/", service_info, name="service-info")]
```

Include `api.urls` in the project's URLconf and start the development server.

**Line-by-line:** The decorator declares the only allowed method. The function accepts DRF's request wrapper. `Response` holds a dictionary/list of primitive values; `status` defaults to 200. The URL maps a stable name to the handler. **Flow:** request -> route -> DRF method dispatch -> Python data -> JSON rendering -> client. **Internal working:** The endpoint itself remains ordinary Python; DRF supplies shared request/response policy around it. **Professional tip:** Define a stable contract and avoid coupling clients to internal model structure. Without the decorator, method restrictions and DRF dispatch behavior are not applied.

# Complete Example 2

Use DRF's test client to test an endpoint:

```python
from rest_framework.test import APITestCase
from django.urls import reverse


class ServiceInfoTests(APITestCase):
    def test_info_returns_json(self):
        response = self.client.get(reverse("service-info"))
        self.assertEqual(response.status_code, 200)
        self.assertEqual(response.data["name"], "inventory")
```

`APITestCase` supplies a DRF-aware client; `reverse` resolves the named route; assertions cover status and representation. **Execution:** test client builds a request -> Django/DRF handles it -> response data is asserted. **Tip:** Add negative tests for unsupported verbs and permission failures.

# Complete Example 3

A plain Django alternative:

```python
from django.http import JsonResponse


def simple_status(request):
    return JsonResponse({"status": "ok"})
```

This is valid for a small fixed response. As input parsing, validation, permissions, and response negotiation become important, DRF avoids duplicated custom mechanisms. Do not add DRF if a simple HTML site has no API requirement.

# Advanced Example

DRF is not a replacement for sound API design. It cannot decide resource boundaries, authorization rules, database transaction semantics, or compatibility policy on your behalf. Select the framework abstraction that matches complexity; avoid converting every simple route into an oversized viewset.

# Under the Hood

DRF's request wrapper parses content lazily, and its `Response` is rendered after the view returns. Authentication, permission, throttle, and exception hooks provide extension points around dispatch.

# Performance Notes

The framework overhead is rarely the first bottleneck. Optimize query count, pagination, payload size, and serialization work using measurements.

# Best Practices

- Use DRF where API-specific capabilities provide value.
- Make validation and permission rules explicit.
- Keep output shape stable and version changes deliberately.
- Test behavior at HTTP boundary as well as unit-level logic.

# Common Mistakes

- Using DRF without understanding Django routing/settings.
- Assuming DRF chooses safe permissions automatically.
- Returning large unpaginated lists.
- Treating framework adoption as API design.

# Debugging

Verify DRF installation, app registration, URL mapping, method, content type, and configured policies. Compare the response status/body with the documented contract.

# Interview Questions

1. Why use DRF rather than raw JSON views? Shared parsing, validation, policy, rendering, and testing tools.
2. Does DRF enforce authorization automatically? No.
3. Is DRF needed for every Django route? No.
4. What should be optimized first? Measured database/query and payload costs.

# Practice Exercises

1. Implement equivalent status endpoints using `JsonResponse` and DRF `Response`.
2. Compare how you would add input validation and permissions to each.
3. Write one API test for status and one for an unsupported method.

# Chapter Summary

DRF reduces repeated API plumbing while preserving Django's flexibility. Adopt it for API-specific needs, not as a substitute for resource design, security policy, and tests.
