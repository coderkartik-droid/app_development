# Introduction

Class-based views (CBVs) express request handling through Python classes and reusable mixins. Django's generic views provide common page patterns such as listing or displaying model records.

# Why this concept exists

Classes and mixins can reduce repeated patterns and make behavior extendable. They are helpful when a view has multiple methods or fits a generic workflow.

# Real-world analogy

A CBV is a configurable service desk template: common behavior is already arranged, and the application supplies the details that differ.

# Internal Working

Django calls `as_view()` to create a callable for URL routing. The callable instantiates the view for a request and dispatches to a method such as `get` or `post`.

```text
URL -> MyView.as_view() -> instantiate -> dispatch(method) -> get/post -> response
```

# Syntax

```python
class HomeView(TemplateView):
    template_name = "home.html"
```

# Complete Example 1

Create a small CBV returning plain text:

```python
# pages/views.py
from django.http import HttpResponse
from django.views import View


class StatusView(View):
    def get(self, request):
        return HttpResponse("Ready")
```

```python
# pages/urls.py
from django.urls import path
from .views import StatusView

urlpatterns = [path("status/", StatusView.as_view(), name="status")]
```

**Line-by-line:** Import response and base `View`; define the class; `get` handles HTTP GET; return response. `as_view()` adapts the class into the callable Django's URL resolver expects. **Flow:** route match -> callable creates request-bound instance -> dispatch selects `get` -> response. **Internal:** `dispatch` routes based on HTTP method. **Professional tip:** Implement only supported methods and return appropriate method-not-allowed behavior rather than accepting everything.

# Complete Example 2

Use a generic template view:

```python
from django.views.generic import TemplateView


class AboutView(TemplateView):
    template_name = "about.html"
    extra_context = {"page_title": "About"}
```

Connect with `path("about/", AboutView.as_view(), name="about")`. Django's generic view renders the configured template and context. **Flow:** route -> generic view -> template resolution -> response. **Tip:** Do not force a CBV when a short function view communicates intent more clearly.

# Advanced Example

DRF also has APIView and generic API views, with API request/response and permission behavior. Django's plain `View` does not provide DRF serializers, authentication classes, or API exception handling.

# Common Mistakes

- Forgetting `.as_view()` in URL patterns.
- Confusing a Django CBV with a DRF API view.
- Overusing inheritance and mixins until behavior is hard to trace.
- Implementing an HTTP method incorrectly.

# Best Practices

- Use generic views for conventional workflows.
- Keep inheritance shallow and explicit.
- Test method dispatch and status behavior.
- Use DRF classes for API endpoints.

# Interview Questions

1. What does `as_view()` do? Creates the callable used by URL routing.
2. What does dispatch do? Selects a handler based on the HTTP method.
3. When use CBVs? Reusable/generic behavior or multi-method view organization.
4. Does Django `View` equal DRF `APIView`? No.

# Practice

Create a `TemplateView` for an about page and a `View` supporting GET only.

# Summary

CBVs provide dispatch, inheritance, and reusable generic behavior. Use them when their structure clarifies the code; use DRF view classes for API-specific endpoints.
