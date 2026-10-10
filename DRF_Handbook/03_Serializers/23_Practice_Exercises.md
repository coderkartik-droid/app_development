# Introduction

These exercises progress from basic field declarations to production concerns. Use a test database and API test client where persistence is involved.

# Why this concept exists

Serializer concepts become durable when implemented and tested across valid, invalid, partial, and unauthorized inputs.

# Real-world analogy

Exercises are controlled production runs: vary one input at a time, inspect the result, then test combined conditions.

# Internal Working

For each exercise, draw the flow:

```text
request -> parser -> serializer fields -> validators -> save/service -> DB -> response
```

Mark where the database is accessed and which layer owns each rule.

# Request Lifecycle (where applicable)

For write exercises, distinguish parser, serializer validation, authorization, transactional persistence, and response rendering. For output-only exercises, identify the queryset and representation steps.

# Syntax

```python
serializer = InputSerializer(data=request.data)
serializer.is_valid(raise_exception=True)
```

# Complete Example 1

Build a serializer for a safe task-creation command:

```python
# tasks/serializers.py
from rest_framework import serializers


class TaskInputSerializer(serializers.Serializer):
    title = serializers.CharField(max_length=120, trim_whitespace=True)
    priority = serializers.ChoiceField(choices=["low", "normal", "high"])

    def validate_title(self, value):
        if not value:
            raise serializers.ValidationError("Title cannot be blank.")
        return value
```

```python
# tasks/views.py
from rest_framework.decorators import api_view
from rest_framework.response import Response
from .serializers import TaskInputSerializer


@api_view(["POST"])
def task_preview(request):
    serializer = TaskInputSerializer(data=request.data)
    serializer.is_valid(raise_exception=True)
    return Response({"preview": serializer.validated_data})
```

```python
# tasks/urls.py
from django.urls import path
from .views import task_preview

urlpatterns = [path("tasks/preview/", task_preview)]
```

**Line-by-line:** `title` trims and caps input; `priority` restricts choices. Field validator rejects an empty trimmed title and returns valid value. View binds parser output, validates, and returns preview. **Request → Serializer → Validation → Database → Response:** parse -> fields/validator -> no DB in preview -> response. **Internal working:** This is a command serializer, not a model serializer. **Professional tip:** Exercise field boundaries and error shape before adding persistence; removing the validator allows whitespace-only title after trimming.

# Complete Example 2

Exercise: create a ModelSerializer for a `Book` with `title`, `year`, and server-owned `owner`. Required outcomes:

1. Explicit public field list.
2. Owner and ID are read-only.
3. The generic view assigns owner from the authenticated request.
4. Tests prove another user's books cannot be listed or updated.

Implement the serializer, view, URLs, and tests in an installed app. Explain request -> validation -> ORM -> response.

# Complete Example 3

Exercise: nested order creation. Accept an order note and 1–10 line items; validate each quantity; reject duplicate product IDs; create parent and children in one transaction; return a read representation. Test forced child failure and assert rollback.

# Advanced Example

Exercise: build separate list, detail, and write serializers. Optimize list queryset for all represented relations. Add tests for field exposure, permission scope, query count, partial update, and invalid nested data.

# Under the Hood

Direct serializer tests isolate conversion and validation. API tests include routing, parser, permission, response status, and renderer. Database tests verify constraints and transaction behavior.

# Performance Notes

Include representative list sizes, related data, and query counts. A passing validation test does not prove the endpoint is efficient.

# Best Practices

- Use deterministic test fixtures.
- Test omitted/null/blank values distinctly.
- Test the public HTTP contract and database result.
- Keep failure cases explicit.

# Common Mistakes

- Testing only valid data.
- Asserting exact internal error text when it is not a promised contract.
- Forgetting transaction rollback tests.
- Testing serializer field behavior without view permissions.

# Debugging

Inspect `.errors`, `.validated_data`, `.data`, DB rows, and response status at different stages. Do not conflate parser errors with serializer errors.

# Interview Questions

1. What tests prove validation? Invalid payload produces expected errors and no side effects.
2. What tests prove persistence? Valid request creates expected rows and response representation.
3. How test rollback? Cause controlled failure and assert all transactional writes are absent.
4. How test performance? Query-count/benchmark tests with representative data.

# Practice Exercises

1. Complete the task preview example and boundary tests.
2. Complete Book owner-scoping exercise.
3. Complete nested order transaction exercise.

# Chapter Summary

Practice should cover field behavior, model mapping, cross-field validation, ownership, nested writes, transactions, and query performance. Test both serializer logic and the full HTTP/data lifecycle.
