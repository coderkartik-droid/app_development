# Introduction

Django commonly describes its design as MTV: Model, Template, View. A model represents data, a template renders presentation, and a view coordinates request handling and response creation.

# Why this concept exists

Separating responsibilities makes code easier to change and test. A data rule should not be hidden in HTML, and a template should not decide how a request is routed.

# Real-world analogy

For a restaurant, the model is the menu and order data, the view is the coordinator, and the template is the printed receipt customers see.

# Internal Working

```text
URL -> View -> Model/ORM (optional) -> Template (optional) -> HTTP response
```

Django's “View” plays a role similar to a controller in other frameworks. In DRF, serializers often handle representation/validation and API views coordinate request logic.

# Syntax

```python
# View chooses context and template.
return render(request, "catalog/list.html", {"items": items})
```

# Complete Example 1

After creating a project and app named `catalog`, register the app, then use:

```python
# catalog/views.py
from django.shortcuts import render


def item_list(request):
    items = ["Notebook", "Pen"]
    return render(request, "catalog/items.html", {"items": items})
```

```html
<!-- catalog/templates/catalog/items.html -->
<h1>Items</h1>
<ul>
  {% for item in items %}
    <li>{{ item }}</li>
  {% endfor %}
</ul>
```

```python
# project urls.py
from django.urls import path
from catalog.views import item_list

urlpatterns = [path("items/", item_list, name="item-list")]
```

**Line-by-line:** The view imports `render`, defines a request handler, creates example data, and passes it as template context. The template loops through `items` and escapes variable output by default. The URL pattern connects `/items/` to `item_list`. **Flow:** request -> resolver -> view -> context -> template rendering -> HTML response. **Internal:** Template rendering is a presentation step; Django's ORM could supply the values instead of the sample list. **Professional tip:** Keep template context explicit and avoid putting business rules in templates.

# Complete Example 2

```python
from django.http import JsonResponse


def item_count(request):
    items = ["Notebook", "Pen"]
    return JsonResponse({"count": len(items)})
```

This endpoint skips a template and returns JSON. `JsonResponse` encodes a dictionary; the browser/client receives JSON rather than HTML. **Flow:** URL -> view -> Python mapping -> JSON serialization -> response. **Internal:** DRF provides richer content negotiation and serialization for APIs. **Professional tip:** Do not duplicate complex query or validation logic across separate HTML and API views; extract shared domain operations.

# Advanced Example

A request may use model data and then present it as HTML or through an API-specific serializer. The model remains the data layer; the representation mechanism differs. **Tip:** Do not assume MTV means every app must have templates—API services can return structured data.

# Common Mistakes

- Treating Django's View as identical to a browser template.
- Putting database queries into templates.
- Returning HTML from an endpoint expected to serve JSON.
- Overloading one module with routing, data access, and presentation.

# Best Practices

- Keep models focused on stored data and domain invariants.
- Keep views focused on coordinating request/response behavior.
- Keep templates focused on rendering.
- For DRF, use serializers for validated API representation.

# Interview Questions

1. What does MTV stand for? Model, Template, View.
2. Which part coordinates the request? The Django view.
3. Is a template required for an API? No.
4. What does a serializer add in DRF? Validation and conversion between complex data and primitive representations.

# Practice

Create one HTML view and one JSON view for the same sample item data. Identify which MTV responsibilities each uses.

# Summary

MTV separates data, coordination, and presentation. Django's view receives the request and chooses the response path; DRF adds serializers and API-oriented views.
