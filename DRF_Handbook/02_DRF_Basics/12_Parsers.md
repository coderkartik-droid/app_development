# Introduction

Parsers convert an incoming request body from bytes into Python data. DRF selects a parser based on the request's `Content-Type` when a view accesses `request.data`.

# Why this concept exists

Clients can submit JSON, HTML form data, or multipart uploads. Endpoint code should work with parsed values rather than manually decoding raw bytes for every content format.

# Real-world analogy

A parser is a mailroom clerk who recognizes the package label, opens it using the right procedure, and passes the contents in a usable form.

# Internal Working

```text
request body bytes + Content-Type -> parser selection -> request.data
```

Parsing is generally lazy. If the content type has no configured parser, DRF can return `415 Unsupported Media Type`; malformed content can produce `400 Bad Request`.

# Request Lifecycle (where applicable)

APIView creates a DRF `Request` with parser instances. On first access to `.data`, DRF checks the media type, chooses a parser, parses the stream, and caches the resulting data/files.

# Syntax

```python
from rest_framework.parsers import JSONParser

class MyView(APIView):
    parser_classes = [JSONParser]
```

# Complete Example 1

This endpoint accepts JSON only. It requires a configured Django/DRF project and an app URLconf.

```python
# api/views.py
from rest_framework import status
from rest_framework.parsers import JSONParser
from rest_framework.response import Response
from rest_framework.views import APIView


class JsonNoteView(APIView):
    parser_classes = [JSONParser]

    def post(self, request):
        title = request.data.get("title")
        if not isinstance(title, str) or not title.strip():
            return Response(
                {"title": ["A non-empty string is required."]},
                status=status.HTTP_400_BAD_REQUEST,
            )
        return Response({"title": title.strip()}, status=status.HTTP_201_CREATED)
```

```python
# api/urls.py
from django.urls import path
from .views import JsonNoteView

urlpatterns = [path("notes/", JsonNoteView.as_view(), name="json-note")]
```

Send a POST with `Content-Type: application/json` and a JSON object body.

**Line-by-line:** `parser_classes` restricts accepted body formats to JSON. `post` accesses parsed values, validates the title's type and nonblank content, returns 400 on invalid input, and 201 for accepted input. `as_view()` registers the class with URL routing. **Flow:** bytes -> JSON parser -> `request.data` -> validation -> `Response` -> renderer. **Internal working:** Parsing is triggered by `.data`, not by the class definition. **Professional tip:** Restrict parsers when an endpoint truly requires a format; keep only formats the client contract supports. Removing `JSONParser` restriction may allow other globally configured parsers.

# Complete Example 2

For browser form data, DRF provides `FormParser`; for file uploads, `MultiPartParser` works with multipart input. A view may declare both:

```python
parser_classes = [FormParser, MultiPartParser]
```

The endpoint can then read text values from `request.data` and uploaded files from request file data. **Flow:** multipart boundaries/content type -> multipart parser -> fields/files -> view. **Tip:** Apply upload size/type validation and storage controls separately; parser selection is not file safety.

# Complete Example 3

A parser configuration can be tested with a JSON request:

```python
response = self.client.post(
    "/api/notes/",
    {"title": "Draft"},
    format="json",
)
self.assertEqual(response.status_code, 201)
```

DRF's test client encodes the payload and supplies the matching media type. Test an unsupported type and malformed JSON as negative cases.

# Advanced Example

A custom parser subclasses `BaseParser`, defines `media_type`, and implements `parse(stream, media_type, parser_context)`. Only create one for a genuine supported representation; parser code is an input boundary and must handle malformed/untrusted bytes safely.

# Under the Hood

DRF compares request media type with parser media types and invokes the chosen parser. Parsed data is cached on the request. Parser exceptions are translated by DRF's exception handling.

# Performance Notes

Large request bodies consume parsing time and memory. Enforce upload/body limits at deployment and application boundaries; do not accept large payloads where a bounded form is sufficient.

# Best Practices

- Configure the narrowest parser set that meets the contract.
- Validate parsed values with serializers.
- Handle malformed input as a client error, not success.
- Test media type mismatch and invalid body cases.

# Common Mistakes

- Confusing parsers (incoming) with renderers (outgoing).
- Using `request.POST` for JSON.
- Assuming multipart parsing validates content safety.
- Accepting arbitrary formats unintentionally.

# Debugging

Inspect request `Content-Type`, configured `parser_classes`, raw body in a safe test, and the resulting `request.data`. Distinguish parser 400/415 errors from serializer validation errors.

# Interview Questions

1. What do parsers do? Convert request body bytes to Python data.
2. When does DRF choose a parser? When parsing request data, based on media type and configuration.
3. JSON parser media type? `application/json`.
4. What does 415 indicate? No parser supports the request's media type.

# Practice Exercises

1. Restrict an endpoint to JSON and test JSON success.
2. Test malformed JSON and an unsupported media type.
3. Compare form and multipart parser behavior.

# Chapter Summary

Parsers handle incoming bodies based on `Content-Type` and produce `request.data`. Configure parsers intentionally, validate parsed values, and treat all request bytes as untrusted.
