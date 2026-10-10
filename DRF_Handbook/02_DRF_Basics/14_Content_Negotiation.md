# Introduction

Content negotiation is the process of choosing a response representation supported by the server and requested by the client. In DRF it commonly uses the request's `Accept` header and configured renderers.

# Why this concept exists

Different clients may prefer JSON, browsable HTML, or another supported format. Negotiation lets one endpoint choose an appropriate representation without duplicating its application logic.

# Real-world analogy

A customer orders the same information in a preferred language; the service checks which formats it can provide and returns a matching version.

# Internal Working

```text
Accept header + renderer_classes -> negotiator -> accepted_renderer/media_type
Response.data + accepted renderer -> rendered response
```

The selected renderer is attached to the request/view context before the handler response is finalized.

# Request Lifecycle (where applicable)

DRF performs content negotiation early in API dispatch. If no renderer is acceptable, the request may receive 406. The same Python response data can then be rendered by the selected format.

# Syntax

```http
Accept: application/json
```

# Complete Example 1

This endpoint offers JSON only:

```python
# api/views.py
from rest_framework.renderers import JSONRenderer
from rest_framework.response import Response
from rest_framework.views import APIView


class NegotiatedStatusView(APIView):
    renderer_classes = [JSONRenderer]

    def get(self, request):
        return Response({
            "accepted_media_type": request.accepted_media_type,
            "format": request.accepted_renderer.format,
        })
```

```python
# api/urls.py
from django.urls import path
from .views import NegotiatedStatusView

urlpatterns = [path("negotiation/", NegotiatedStatusView.as_view())]
```

Request it with `Accept: application/json`.

**Line-by-line:** The view limits renderers to JSON. `get` returns the selected media type and renderer format as diagnostic data. `APIView` performs negotiation before entering the handler, so these request properties are available. **Flow:** Accept header -> negotiator selects JSON renderer -> GET handler -> Response -> selected renderer -> JSON bytes. **Internal working:** If the client accepts no configured format, dispatch can fail before the method handler. **Professional tip:** Avoid depending on a browser's broad `Accept` header for API clients; send explicit expectations. Removing `renderer_classes` uses global defaults instead.

# Complete Example 2

Allow JSON and browsable HTML:

```python
renderer_classes = [JSONRenderer, BrowsableAPIRenderer]
```

The same `Response.data` can be rendered differently according to the request's accepted types. This is useful for development but should be a deliberate deployment choice.

# Complete Example 3

Force a format with a URL suffix if the project configures DRF's format suffix support, for example `.json`. Format suffixes are an alternative input to negotiation, not a reason to manually serialize the body.

# Advanced Example

Negotiation does not change endpoint authorization or the underlying resource. It chooses presentation. API versioning and language negotiation are related concerns but require their own explicit policies.

# Under the Hood

The negotiator evaluates available renderer classes and request preferences, then sets `accepted_renderer`, `accepted_media_type`, and renderer context. A `Response` is rendered after the view returns.

# Performance Notes

Negotiation itself is lightweight. Multiple renderers can increase maintenance and test surface; expensive renderer processing and large payloads should be measured.

# Best Practices

- Define supported formats globally or per view, not accidentally.
- Use stable JSON contracts for machine clients.
- Test supported and unacceptable `Accept` values.
- Treat browsable API exposure as a deployment decision, not an access-control mechanism.

# Common Mistakes

- Confusing `Accept` (desired response) with `Content-Type` (submitted body format).
- Assuming renderer order always overrides client preference.
- Treating content negotiation as API versioning.
- Forgetting tests for unsupported formats.

# Debugging

Inspect the incoming `Accept` header, view/global renderer lists, accepted renderer, status, and response content type. Verify the client sends the header expected by the contract.

# Interview Questions

1. What does `Accept` mean? The response media types the client can accept.
2. What does `Content-Type` mean? The media type of the request body or response body.
3. What happens when no renderer matches? Typically a 406 response.
4. Does negotiation select permissions? No, it selects representation.

# Practice Exercises

1. Test JSON and HTML preferences on a view configured for both.
2. Test a media type the view does not support.
3. Explain the difference between `Accept` and `Content-Type`.

# Chapter Summary

Content negotiation matches client response preferences to server renderers. It runs before the handler and affects representation, not business logic or authorization.
