# Introduction

Static files are application assets such as CSS, JavaScript, and icons. Media files are user-uploaded content such as profile images or documents. Django configures and handles these categories differently.

# Why this concept exists

Applications need a predictable way to reference design assets and accept uploaded files. Separating static assets from user media improves deployment and security planning.

# Real-world analogy

Static files are the printed signs installed by the business; media files are documents visitors submit and the organization must store carefully.

# Internal Working

During development, Django can serve static files under limited settings. Production deployments typically collect static assets for a web server/CDN and configure media storage separately. Upload handling passes file data through request parsing and storage backends.

```text
static source files -> collectstatic -> deployment static host
uploaded file -> validation -> configured media storage -> controlled access
```

# Syntax

```python
# settings.py
STATIC_URL = "static/"
MEDIA_URL = "media/"
MEDIA_ROOT = BASE_DIR / "media"
```

# Complete Example 1

Configure a local development project and reference a CSS file:

```python
# settings.py
STATIC_URL = "static/"
```

```text
catalog/
  static/
    catalog/
      site.css
```

```html
{% load static %}
<link rel="stylesheet" href="{% static 'catalog/site.css' %}">
```

`STATIC_URL` defines the URL prefix; the namespaced app folder avoids asset collisions; `{% load static %}` enables the template tag; `{% static ... %}` builds the asset URL. **Flow:** template rendering -> static URL construction -> browser requests asset separately. **Internal:** Django does not embed CSS into the HTML response. **Professional tip:** Production static serving is typically handled outside Django's development server.

# Complete Example 2

Configure an uploaded-file field:

```python
class Profile(models.Model):
    avatar = models.ImageField(upload_to="avatars/", blank=True)
```

```python
# settings.py
MEDIA_URL = "media/"
MEDIA_ROOT = BASE_DIR / "media"
```

The field stores a storage-relative file name; the configured storage handles the file. This requires appropriate image support and secure deployment configuration. **Flow:** form/request upload -> validation -> storage backend saves bytes -> model stores reference. **Tip:** Validate file size/type and consider private storage; never trust filename extensions alone.

# Advanced Example

In production, use a dedicated storage backend or object store where appropriate, enforce upload limits, scan risky files, and serve user content from a separate origin when threat model warrants it. Do not expose private documents through a public URL unintentionally.

# Common Mistakes

- Serving production assets with `runserver`.
- Confusing static assets with user uploads.
- Trusting user-provided file names or MIME types.
- Allowing unlimited upload size or public access by default.

# Best Practices

- Configure `STATIC_ROOT` and run `collectstatic` for deployment.
- Set explicit upload validation and size limits.
- Use safe storage names and access rules.
- Back up media separately from database metadata.

# Interview Questions

1. Static vs media? App-owned assets vs user-uploaded files.
2. What does `collectstatic` do? Collects static assets for deployment.
3. What does `upload_to` define? A relative media storage path.
4. Why validate uploads? File metadata can be spoofed and content may be unsafe.

# Practice

Add a static stylesheet and a validated media field. Document where each is stored in development and production.

# Summary

Static assets and uploaded media have different ownership, serving, and security requirements. Configure each explicitly and do not rely on Django's development server in production.
