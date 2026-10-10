# Introduction

Renderers convert DRF response data into a final representation such as JSON or browsable HTML. The selected renderer determines the response body format and media type.

# Why this concept exists

An API view should describe data rather than manually encode every output format. Renderers centralize serialization of primitives and allow multiple client representations where appropriate.

# Real-world analogy

A renderer is a packaging station: the same product information can be packed as a machine-readable JSON parcel or a human-friendly browsable display.

# Internal Working

```text
Response.data + accepted renderer -> render() -> bytes + Content-Type
```

The renderer is selected during content negotiation, then invoked as DRF finalizes the response.

# Request Lifecycle (where applicable)

After a handler returns `Response`, DRF attaches the accepted renderer and media type. Rendering occurs before the response is sent. Renderer selection can be influenced by `Accept` headers and format suffixes.

# Syntax

```python
from rest_framework.renderers import JSONRenderer

class JsonOnlyView(APIView):
    renderer_classes = [JSONRenderer]
```

# Complete Example 1

Restrict an endpoint to JSON output:

```python
# api/views.py
from rest_framework.renderers import JSONRenderer
from rest_framework.response import Response
from rest_framework.views import APIView


class SummaryView(APIView):
    renderer_classes = [JSONRenderer]

    def get(self, request):
        return Response({"service": "catalog", "items": 3})
```

```python
# api/urls.py
from django.urls import path
from .views import SummaryView

urlpatterns = [path("summary/", SummaryView.as_view(), name="summary")]
```

**Line-by-line:** Import JSON renderer, response, and APIView. `renderer_classes` restricts output to JSON. `get` returns primitive data; DRF finalizes it using that renderer. The URL connects the endpoint. **Flow:** GET with `Accept` -> content negotiation -> view -> `Response.data` -> JSON renderer -> response body. **Internal working:** The response stores Python data until rendering. **Professional tip:** Restrict formats only when your client contract needs it; global renderer choices also affect browsable API usability. Removing `JSONRenderer` restriction permits other configured renderers.

# Complete Example 2

The browsable API renderer adds an interactive HTML representation useful during development. It can be retained in development while production policies choose JSON-only output if required.

```python
from rest_framework.renderers import JSONRenderer, BrowsableAPIRenderer

renderer_classes = [JSONRenderer, BrowsableAPIRenderer]
```

`Accept: text/html` may select browsable HTML, while JSON clients request JSON. **Flow:** request preferences -> negotiation -> selected renderer. **Tip:** Do not mistake the browsable interface for authorization; permissions still control endpoint access.

# Complete Example 3

Check the returned media type in an API test:

```python
response = self.client.get("/api/summary/", HTTP_ACCEPT="application/json")
self.assertEqual(response["Content-Type"].split(";")[0], "application/json")
```

The `Accept` header communicates desired response format. Test the actual production renderer policy, especially if HTML is disabled.

# Advanced Example

A custom renderer subclasses `BaseRenderer`, declares `media_type`, `format`, and implements `render(data, accepted_media_type, renderer_context)`. Keep custom formats simple, documented, and tested; do not use rendering as a place for business logic.

# Under the Hood

DRF's response is rendered with context containing request/view information. The renderer encodes data and may determine charset/content type. Unsupported output preferences can cause `406 Not Acceptable`.

# Performance Notes

Large nested response structures cost CPU and memory. Use pagination, avoid redundant fields, and profile renderer cost only after measuring the full request path.

# Best Practices

- Return primitive/serializer data and let renderers encode it.
- Document supported media types.
- Keep renderer sets small and intentional.
- Test content type and representative output.

# Common Mistakes

- Confusing renderers with parsers.
- Manually JSON-encoding data before passing to `Response`.
- Assuming the browsable API is a security boundary.
- Returning unsupported Python objects.

# Debugging

Inspect `Accept`, configured renderer classes, negotiated media type, response `Content-Type`, and renderer exceptions. Test requests with explicit accepted formats.

# Interview Questions

1. What do renderers do? Convert response data into bytes/representations.
2. How is a renderer selected? Content negotiation, commonly using `Accept`.
3. What is the JSON renderer? DRF's renderer for JSON output.
4. What can 406 mean? No configured renderer satisfies the client's accepted formats.

# Practice Exercises

1. Configure JSON-only output and test `Content-Type`.
2. Add browsable API output in development.
3. Send an unsupported `Accept` header and inspect behavior.

# Chapter Summary

Renderers produce the outgoing representation from `Response.data`. Configure them deliberately, use content negotiation, and test the media type and output clients receive.
