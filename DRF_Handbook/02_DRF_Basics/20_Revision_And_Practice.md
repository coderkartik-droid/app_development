# Introduction

This final chapter condenses DRF basics into a revision path and short exercises. The goal is to explain how a request becomes a response and identify which DRF component owns each step.

# Why this concept exists

Remembering the lifecycle helps you debug new APIs and learn serializers, viewsets, authentication, and permissions without treating them as disconnected features.

# Real-world analogy

An API request is a parcel moving through a warehouse: routing sends it to a desk, checks verify access, parsing opens it, validation inspects it, storage processes it, and rendering prepares the receipt.

# Internal Working

```text
Client
  -> Django middleware
  -> URL resolver
  -> DRF negotiation + Request
  -> authentication / permissions / throttles
  -> parser (when request.data is read)
  -> view / serializer / ORM
  -> Response(data, status)
  -> renderer
  -> HTTP response
```

# Request Lifecycle (where applicable)

The lifecycle is ordered but some work is lazy. Parsers usually run only when request body data is accessed. Querysets may remain unevaluated until iterated. Response rendering happens after the handler returns.

# Syntax

```python
@api_view(["GET"])
def example(request):
    return Response({"result": "ok"}, status=status.HTTP_200_OK)
```

# Complete Example 1

Build a tiny tested GET endpoint:

```python
# api/views.py
from rest_framework.decorators import api_view
from rest_framework.response import Response


@api_view(["GET"])
def revision_status(request):
    return Response({"topic": "DRF basics", "ready": True})
```

```python
# api/urls.py
from django.urls import path
from .views import revision_status

urlpatterns = [path("revision/", revision_status, name="revision-status")]
```

```python
# api/tests.py
from django.urls import reverse
from rest_framework.test import APITestCase


class RevisionStatusTests(APITestCase):
    def test_status_contract(self):
        response = self.client.get(reverse("revision-status"))
        self.assertEqual(response.status_code, 200)
        self.assertTrue(response.data["ready"])
```

Include the URLconf in the project and run `python manage.py test`.

**Line-by-line:** The decorator accepts GET; the view returns a primitive mapping. The app route assigns a stable name. `APITestCase` makes a request using URL reversal and asserts status/data. **Execution flow:** test client -> URL resolver -> DRF function wrapper -> handler -> `Response` -> renderer -> test assertions. **Internal working:** The response is rendered after the handler; API tests execute without a live server. **Professional explanation:** This tests wiring and contract, not authentication or database behavior; those require dedicated cases. Removing the URL include makes reverse resolution fail; removing the status assertion can hide a method or permission error.

# Complete Example 2

Practice a parsed POST:

```python
response = self.client.post(
    reverse("create-label"),
    {"name": "urgent"},
    format="json",
)
```

The test client encodes JSON and sets content type, allowing the configured parser to populate request data. Assert creation status and response fields. If the endpoint uses a serializer, test invalid fields too.

# Complete Example 3

Practice method and representation expectations:

```python
response = self.client.get(
    reverse("revision-status"),
    HTTP_ACCEPT="application/json",
)
self.assertIn("application/json", response["Content-Type"])
```

This asserts negotiated output. Send a disallowed `Accept` value as a separate test where renderer configuration expects rejection.

# Advanced Example

Before moving to the next section, be able to explain the difference between:

- `HttpRequest` and DRF `Request`
- `JsonResponse` and DRF `Response`
- parser and renderer
- authentication and permission
- HTTP status and response body
- project-level settings and view-level policy

# Under the Hood

DRF extends Django dispatch; it does not bypass it. URL resolution, app registration, middleware, model registry, ORM, and migrations remain Django responsibilities.

# Performance Notes

Most basics endpoints are cheap. As APIs grow, paginate, inspect ORM query counts, avoid oversized representations, and profile parsing/serialization only when measured.

# Best Practices

- Follow the request lifecycle during debugging.
- Test happy and unhappy paths with APIClient/APITestCase.
- Keep policy explicit and secure by default.
- Treat request content and IDs as untrusted.

# Common Mistakes

- Returning raw dictionaries from ordinary Django views.
- Confusing parser and renderer direction.
- Assuming global defaults without inspecting settings.
- Skipping route, permission, and error tests.

# Debugging

Ask in order: Did the URL resolve? Was the method allowed? Did negotiation find a renderer? Did parsing succeed? Did authentication/permission pass? Did ORM/serializer logic work? What status and content type were rendered?

# Interview Questions

1. What happens before a DRF handler? Request initialization, negotiation, authentication, permission, and throttle checks according to configuration.
2. When is request data parsed? Usually on first access.
3. When is response data rendered? During response finalization after the handler.
4. What is the most important security default? Explicit least-privilege authentication/permission behavior and tests.

# Practice Exercises

1. Build a GET-only status endpoint and test 200/405.
2. Build a JSON POST endpoint and validate input.
3. Configure JSON-only rendering and test media type.
4. Draw the complete request lifecycle from memory.

# Chapter Summary

DRF basics are a layered request-to-response system built on Django. Routing finds a handler; DRF negotiates, authenticates, authorizes, parses, and dispatches; serializers and ORM handle data; `Response` is rendered for the client. Understand these boundaries before adding advanced abstractions.
