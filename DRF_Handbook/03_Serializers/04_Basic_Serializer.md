# Introduction

A basic DRF `Serializer` declares fields explicitly. It can validate incoming values and convert a Python object or mapping into a primitive representation, even when there is no Django model.

# Why this concept exists

API payloads often represent commands, calculations, filters, or service results rather than a database table. A plain serializer supplies validation and representation without requiring model metadata.

# Real-world analogy

A plain serializer is a carefully designed form: every question and accepted answer is specified directly, independent of how the organization stores the result.

# Internal Working

Fields parse primitive values into native Python types, enforce required/range/format rules, then object-level validation runs. For output, each field reads from an object or mapping and represents its value.

# Request Lifecycle (where applicable)

```text
request.data -> Serializer(data=...) -> is_valid()
 -> errors OR validated_data -> service/explicit persistence -> Response(serializer.data)
```

# Syntax

```python
class ContactSerializer(serializers.Serializer):
    name = serializers.CharField(max_length=80)
    email = serializers.EmailField()
```

# Complete Example 1

This complete endpoint validates an input message and returns a derived result; it does not write to a database.

```python
# api/serializers.py
from rest_framework import serializers


class MessageSerializer(serializers.Serializer):
    text = serializers.CharField(max_length=200, allow_blank=False)


class MessageResultSerializer(serializers.Serializer):
    text = serializers.CharField()
    character_count = serializers.IntegerField()
```

```python
# api/views.py
from rest_framework import status
from rest_framework.decorators import api_view
from rest_framework.response import Response
from .serializers import MessageSerializer, MessageResultSerializer


@api_view(["POST"])
def message_summary(request):
    incoming = MessageSerializer(data=request.data)
    incoming.is_valid(raise_exception=True)
    text = incoming.validated_data["text"]
    outgoing = MessageResultSerializer({
        "text": text,
        "character_count": len(text),
    })
    return Response(outgoing.data, status=status.HTTP_200_OK)
```

```python
# api/urls.py
from django.urls import path
from .views import message_summary

urlpatterns = [path("message-summary/", message_summary, name="message-summary")]
```

**Line-by-line:** `MessageSerializer` defines a bounded, nonblank input string. `MessageResultSerializer` defines output shape. The view binds incoming data, raises a DRF validation exception if invalid, reads typed validated data, computes a result, serializes output, and returns 200. The URL connects endpoint.

**Request → Serializer → Validation → Database → Response:** request body is parsed -> input serializer validates -> no database step is required -> output serializer represents computed data -> Response renderer sends JSON. **Internal working:** `raise_exception=True` delegates error conversion to DRF. **Professional use case:** Validate report/search commands and return computed summaries. Removing `is_valid()` makes `validated_data` unavailable and bypasses validation.

# Complete Example 2

Validate a pagination query object:

```python
class PageQuerySerializer(serializers.Serializer):
    page = serializers.IntegerField(min_value=1, default=1)
    page_size = serializers.IntegerField(min_value=1, max_value=100, default=20)
```

Instantiate with `data=request.query_params`, validate, and use the typed result to control a bounded queryset slice. Defaults are applied when omitted. Do not let clients choose unbounded page sizes.

# Complete Example 3

Serialize a plain Python mapping:

```python
serializer = MessageResultSerializer({
    "text": "Hello",
    "character_count": 5,
})
payload = serializer.data
```

With `instance` rather than `data`, the serializer is in representation mode; no validation or database write occurs.

# Advanced Example

A basic serializer can implement `create()`/`update()` for custom persistence, but if the object is a straightforward Django model, ModelSerializer may be clearer. Complex business operations should often delegate to a service to keep serializer methods focused.

# Under the Hood

Serializer fields are bound with source paths and validators. Input validation collects field errors before calling object-level `validate`; representation iterates readable fields and obtains values from the instance/mapping.

# Performance Notes

Keep fields small and avoid expensive custom attributes. For lists, serializer execution runs per object; prevent query-per-object patterns.

# Best Practices

- Use explicit field declarations and bounds.
- Separate input and output serializers when contracts differ.
- Access `validated_data` only after successful validation.
- Keep business side effects out of validators.

# Common Mistakes

- Expecting a plain Serializer to persist automatically.
- Using `.data` from an invalid input serializer as validated output.
- Omitting bounds on user-controlled values.
- Returning a serializer instance rather than `.data` in custom code.

# Debugging

Inspect `serializer.errors`, `validated_data`, and serializer representation. Confirm whether it was initialized with `data` or `instance`, and test defaults and missing values.

# Interview Questions

1. What is `serializers.Serializer`? An explicit field-based serializer independent of a model.
2. Does it infer fields? No.
3. What does `default` do? Provides a value when input is omitted, subject to field semantics.
4. How represent a mapping? Pass it as `instance`, then access `.data`.

# Practice Exercises

1. Build an input serializer for a search command with `query` and `limit`.
2. Add a cross-field rule and test invalid data.
3. Create a separate output serializer for aggregate results.

# Chapter Summary

A basic Serializer is the flexible foundation for validation and representation. Define fields explicitly, distinguish input from instance mode, and decide persistence separately.
