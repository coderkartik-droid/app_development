# Introduction

Django templates generate text-based presentation, commonly HTML, from a template and context data. They are useful for server-rendered pages; JSON APIs typically use serializers and renderers instead.

# Why this concept exists

Separating presentation markup from Python behavior makes pages easier to maintain and allows the same view to supply different data.

# Real-world analogy

A template is a reusable form letter: fixed wording and layout are filled with values for a particular request.

# Internal Working

```text
view context + template source -> template engine -> rendered HTML -> response
```

Django resolves a template by configured loaders, evaluates supported template tags/filters, escapes variables by default, and returns rendered content.

# Syntax

```django
<h1>{{ title }}</h1>
{% for item in items %}<li>{{ item }}</li>{% endfor %}
```

# Complete Example 1

Create `catalog/templates/catalog/list.html` and connect this view:

```python
# catalog/views.py
from django.shortcuts import render


def item_list(request):
    return render(
        request,
        "catalog/list.html",
        {"items": ["Notebook", "Pen"]},
    )
```

```html
<!-- catalog/templates/catalog/list.html -->
<h1>Catalog</h1>
<ul>
  {% for item in items %}
    <li>{{ item }}</li>
  {% empty %}
    <li>No items available</li>
  {% endfor %}
</ul>
```

Add a route to `item_list`. **Line-by-line:** `render` accepts the request, template name, and context. The heading is static; `for` loops over context items; `empty` provides an empty-state branch; `{{ item }}` is autoescaped. **Flow:** URL -> view context -> loader finds template -> tags render -> HTML response. **Internal:** Django's template language is intentionally not arbitrary Python execution. **Professional tip:** Keep business decisions out of template tags.

# Complete Example 2

Use template inheritance:

```html
<!-- base.html -->
<!doctype html>
<html><body>{% block content %}{% endblock %}</body></html>
```

```html
<!-- child.html -->
{% extends "base.html" %}
{% block content %}<h1>{{ heading }}</h1>{% endblock %}
```

The child fills a named block in the parent layout. **Flow:** loader resolves child -> inheritance loads base -> block content is inserted. **Tip:** Escape user-generated values; avoid `safe` unless content has been securely sanitized.

# Advanced Example

A form can include CSRF protection with `{% csrf_token %}` when rendered in a Django template. This protects applicable cookie-authenticated form submissions; API authentication has its own security model and must not be confused with template CSRF behavior.

# Common Mistakes

- Placing complex Python logic in templates.
- Disabling autoescaping for untrusted content.
- Forgetting to configure template directories/loaders.
- Using templates to produce a complex API schema.

# Best Practices

- Use inheritance for shared layout.
- Keep context minimal and explicit.
- Preserve autoescaping and use CSRF tokens for forms.
- Use DRF serializers/renderers for structured APIs.

# Interview Questions

1. What is template context? Data supplied to a template for rendering.
2. What does autoescaping protect against? Unsafe HTML/script injection in rendered values.
3. What is template inheritance? Reusing a base template with overridable blocks.
4. Are templates required for DRF? No.

# Practice

Create a base template, a child list template, and an empty-state branch. Verify user-controlled text is escaped.

# Summary

Templates render server-side presentation from context. Keep presentation separate, retain escaping, and use DRF serialization rather than HTML templates for API representations.
