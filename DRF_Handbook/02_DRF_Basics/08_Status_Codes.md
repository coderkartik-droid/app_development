# Introduction

An HTTP status code communicates the broad outcome of a request. DRF responses should use codes that match what happened, so clients can decide whether to display data, correct input, authenticate, or retry.

# Why this concept exists

Clients need consistent machine-readable outcomes independent of message wording. A status code is part of the API contract, not decoration.

# Real-world analogy

A status code is the color and label on a delivery receipt: delivered, accepted, missing address, or forbidden destination.

# Internal Working

```text
view outcome -> Response(data, status) -> renderer + headers -> HTTP response
client interprets status class and documented body
```

DRF exceptions such as `ValidationError` and `NotFound` are translated into appropriate responses by the exception handler.

# Request Lifecycle (where applicable)

The view or framework exception handler chooses a code. DRF finalizes and renders the response; the client receives the numeric code and body. A client should branch on code and structured fields, not parse arbitrary prose.

# Syntax

```python
from rest_framework import status
return Response({"detail": "Created"}, status=status.HTTP_201_CREATED)
```

# Complete Example 1

This view distinguishes successful lookup and missing resource outcomes:

```python
# api/views.py
from rest_framework import status
from rest_framework.decorators import api_view
from rest_framework.response import Response

ITEMS = {1: {"id": 1, "name": "Notebook"}}


@api_view(["GET"])
def item_detail(request, item_id):
    item = ITEMS.get(item_id)
    if item is None:
        return Response(
            {"detail": "Item not found."},
            status=status.HTTP_404_NOT_FOUND,
        )
    return Response(item, status=status.HTTP_200_OK)
```

```python
# api/urls.py
from django.urls import path
from .views import item_detail

urlpatterns = [path("items/<int:item_id>/", item_detail, name="item-detail")]
```

**Line-by-line:** Imports provide status constants, method adaptation, and response. `ITEMS` is sample data. The GET handler looks up the captured integer; a miss returns a structured 404; a match returns 200. **Flow:** route capture -> lookup -> outcome branch -> rendered response. **Internal working:** The numeric code travels in the HTTP status line; the body gives additional context. **Professional tip:** Use DRF's named status constants for readability and write tests for both branches. Removing the missing branch would risk treating absent data as success.

# Complete Example 2

Return creation status:

```python
return Response(
    {"id": created_id},
    status=status.HTTP_201_CREATED,
)
```

201 indicates a resource was created. For a persistent operation, send this only after the database transaction succeeds. A `Location` header may identify the new resource when useful.

# Complete Example 3

A validation failure commonly returns 400:

```python
raise serializers.ValidationError({"name": ["This field is required."]})
```

DRF catches this known exception and emits a structured 400 response. Do not catch it broadly and convert it to 200, because clients would interpret invalid input as success.

# Advanced Example

Common classes:

| Code | Meaning |
|---|---|
| 200 | Successful response with representation |
| 201 | Resource created |
| 204 | Successful operation with no response body |
| 400 | Invalid request/input |
| 401 | Authentication required/failed |
| 403 | Authenticated or identified caller lacks permission |
| 404 | Resource not found |
| 405 | Method not allowed |
| 409 | Conflict, when API semantics define one |
| 500 | Unexpected server failure |

Exact authentication behavior can depend on the authentication scheme; document it.

# Under the Hood

DRF's `status` module contains integer constants. `Response` stores the selected status until finalization; exception handling maps recognized API exceptions to codes.

# Performance Notes

Status selection has negligible cost. Correct codes reduce client retries and unnecessary parsing; caching and retry behavior depend on the code and headers.

# Best Practices

- Match status to completed operation and outcome.
- Use structured error bodies and stable fields.
- Never return success before a write commits.
- Test status and body together.

# Common Mistakes

- Returning 200 for every outcome.
- Using 201 for an operation that did not create anything.
- Putting a body in a 204 response.
- Assuming 401 and 403 are interchangeable.

# Debugging

Inspect status code, `Allow`/authentication headers, response body, and server logs. Reproduce missing, invalid, unauthorized, and successful cases.

# Interview Questions

1. What does 201 mean? A resource was successfully created.
2. 401 vs 403? Authentication is required/failed vs access is refused under the applicable policy.
3. What does 204 imply? Success with no representation body.
4. How are serializer validation errors returned? Typically as structured 400 responses.

# Practice Exercises

1. Add 200, 201, 400, 404, and 405 test cases to a sample endpoint.
2. Verify no success response is returned before a database write completes.

# Chapter Summary

Status codes are part of the API contract. Select them from actual outcomes, pair errors with stable structured details, and test both code and payload.
