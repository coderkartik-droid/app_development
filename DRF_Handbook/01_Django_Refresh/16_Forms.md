# Introduction

Django forms define input fields, validate submitted values, and expose cleaned data. Model forms can derive fields from a model. DRF serializers perform analogous validation and representation work for APIs, with API-specific behavior.

# Why this concept exists

Raw request input is untrusted text. Forms provide a central place to parse and validate values before application code uses them.

# Real-world analogy

A form is an intake checklist: it verifies required details and acceptable formats before passing a cleaned submission to the next step.

# Internal Working

```text
request data -> bind form -> is_valid() -> clean/validate -> cleaned_data -> save/use
```

The form collects field-level and cross-field errors. Only use `cleaned_data` after validation succeeds.

# Syntax

```python
form = ContactForm(request.POST)
if form.is_valid():
    message = form.cleaned_data["message"]
```

# Complete Example 1

A small runnable Django form and view:

```python
# contact/forms.py
from django import forms


class ContactForm(forms.Form):
    name = forms.CharField(max_length=80)
    message = forms.CharField(max_length=1000, widget=forms.Textarea)
```

```python
# contact/views.py
from django.shortcuts import render
from .forms import ContactForm


def contact(request):
    form = ContactForm(request.POST or None)
    sent = False
    if request.method == "POST" and form.is_valid():
        sent = True
    return render(request, "contact.html", {"form": form, "sent": sent})
```

```html
<form method="post">
  {% csrf_token %}
  {{ form.as_p }}
  <button type="submit">Send</button>
</form>
```

Connect `contact` to a URL and configure templates. **Line-by-line:** `ContactForm` declares bounded text fields; view binds POST data (or an unbound form on GET); `is_valid` validates; template renders errors and fields; CSRF token protects the cookie-authenticated form submission. **Flow:** GET -> blank form; POST -> parse -> validate -> render result/errors. **Internal:** `cleaned_data` is available only after valid submission. **Professional tip:** Store or send the message only after validation and apply abuse controls.

# Complete Example 2

A field-level validator:

```python
from django import forms


class QuantityForm(forms.Form):
    quantity = forms.IntegerField(min_value=1, max_value=100)
```

The field parses text to an integer and checks inclusive bounds. A malformed or out-of-range value appears in `form.errors`; it is not present as valid cleaned input. **Tip:** Business invariants also belong in domain/database safeguards when multiple entry paths exist.

# Advanced Example

Use `clean()` for cross-field validation, call `super().clean()`, and add errors with `add_error`. In DRF, serializers provide similar input validation for API payloads, but template CSRF and browser form behavior should not be copied blindly.

# Common Mistakes

- Reading raw `request.POST` instead of validated data.
- Accessing `cleaned_data` before `is_valid()`.
- Omitting CSRF tokens from HTML forms.
- Assuming a form alone protects all API/database entry paths.

# Best Practices

- Validate type, length, range, and cross-field rules.
- Render errors clearly.
- Keep side effects after validation.
- Use serializers for DRF request/response representation.

# Interview Questions

1. What does `is_valid()` do? Runs parsing and validation.
2. Where are validated values? `cleaned_data`.
3. Why CSRF token? Protects applicable browser session form requests from cross-site forgery.
4. Forms vs DRF serializers? Both validate, but serializers target API data/representations.

# Practice

Add a quantity form with bounds and display field errors for invalid input.

# Summary

Django forms parse and validate untrusted browser input. For DRF APIs, serializers offer the API-focused validation and representation layer.
