# Introduction

DRF beginner mistakes often come from mixing Django and DRF APIs, skipping validation or permissions, or misunderstanding when parsing and rendering happen. Correcting these early prevents unreliable endpoint contracts.

# Why this concept exists

An endpoint can return JSON and still be unsafe, inconsistent, or incorrect. Knowing framework boundaries helps identify the right tool and the right layer for a fix.

# Real-world analogy

Using the wrong framework object is like sending a fragile item in an envelope marked “handle with care” but never packing it safely. Labels do not replace the underlying process.

# Internal Working

```text
request bytes -> parser -> validation -> permission-checked logic
             -> database operation -> Response -> renderer -> client
```

Skipping a stage can cause data errors, access leaks, or malformed output. Each stage has a distinct responsibility.

# Request Lifecycle (where applicable)

DRF first routes and negotiates, then authenticates/authorizes, parses input when accessed, runs view logic, and renders the response. A framework setting or decorator can change the pipeline; test effective behavior.

# Syntax

```python
serializer = ItemSerializer(data=request.data)
if serializer.is_valid():
    item = serializer.save()
    return Response(serializer.data, status=201)
return Response(serializer.errors, status=400)
```

# Complete Example 1

This endpoint illustrates basic validation without a database:

```python
from rest_framework import status
from rest_framework.decorators import api_view
from rest_framework.response import Response


@api_view(["POST"])
def create_tag(request):
    name = request.data.get("name")
    if not isinstance(name, str) or not name.strip():
        return Response(
            {"name": ["A non-empty string is required."]},
            status=status.HTTP_400_BAD_REQUEST,
        )
    return Response(
        {"name": name.strip()},
        status=status.HTTP_201_CREATED,
    )
```

**Line-by-line:** The decorator restricts methods. `request.data` asks DRF to parse the body. Type and blank checks happen before `.strip`; invalid data returns a field error and 400; valid data returns a representation and 201. **Flow:** request -> parser -> validation -> success/error Response -> renderer. **Internal working:** A missing type check could trigger an exception or unsafe assumptions. **Professional tip:** Replace repeated manual checks with a serializer and add permission policy before database writes.

# Complete Example 2

Use `Response`, not a raw dict:

```python
return Response({"count": 0})
```

Returning a Python mapping directly from a view is not the DRF response contract. `Response` delays rendering and participates in content negotiation.

# Complete Example 3

Scope destructive ORM writes:

```python
Item.objects.filter(owner=request.user, pk=item_id).delete()
```

The ownership and ID predicates prevent a client from deleting another user's row by guessing an identifier. In production, enforce access through permissions/querysets and check whether a row matched.

# Advanced Example

Never use `except Exception: return Response({"ok": True})`. Broad catches hide programming and database failures and convert them into success-shaped output. Handle known exceptions specifically, log unexpected failures safely, and let DRF's exception handler produce an appropriate server error.

# Under the Hood

`request.data` and `Response` are DRF abstractions; `request.POST` and `JsonResponse` are Django interfaces with different semantics. DRF policy defaults may permit access unless deliberately configured.

# Performance Notes

Avoid giant response collections, repeated related-object queries, and expensive custom parsing. Performance problems are separate from correctness/security problems—address both.

# Best Practices

- Use serializers for input and output contracts.
- Set restrictive permission defaults.
- Bind database operations to caller/resource ownership.
- Use status codes accurately and test error paths.

# Common Mistakes

- Mixing `request.POST` with JSON requests.
- Returning ORM objects directly.
- Forgetting permissions because the UI hides controls.
- Using mutable global lists as production storage.
- Returning 200 for validation or missing-resource failures.

# Debugging

Trace the full request path; check media type, parser, serializer errors, permissions, queryset filtering, status, and response shape. Reproduce using API tests rather than guessing from a browser page.

# Interview Questions

1. Why not trust IDs from a client? They identify a target but do not prove authorization.
2. Why use serializers? Central validation and representation.
3. Why avoid broad exception catches? They hide failures and can misreport success.
4. What does a hidden UI button enforce? Nothing server-side; endpoint permissions must do it.

# Practice Exercises

1. Find a raw dictionary return and replace it with `Response`.
2. Add validation for missing, wrong-type, and blank input.
3. Add an ownership-aware query and test access denial.

# Chapter Summary

Keep Django and DRF interfaces distinct, validate input, enforce permissions on the server, scope data operations, and report failures honestly. Test negative cases as rigorously as success.
