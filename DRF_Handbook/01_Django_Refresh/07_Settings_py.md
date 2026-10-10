# Introduction

`settings.py` contains project configuration: installed apps, middleware, URL configuration, templates, database settings, security options, and environment-specific behavior.

# Why this concept exists

Framework components need shared configuration. Central settings make the application predictable and allow deployment environments to configure different databases and security options.

# Real-world analogy

Settings are the control panel of a building: they configure utilities and safety rules without changing each room's internal design.

# Internal Working

Django loads the settings module selected by `DJANGO_SETTINGS_MODULE` (normally set by `manage.py`). Apps and middleware read these values during application setup and request handling.

```text
manage.py -> DJANGO_SETTINGS_MODULE -> settings -> app registry/middleware/DB
```

# Syntax

```python
INSTALLED_APPS = [
    "django.contrib.admin",
    "django.contrib.auth",
    "rest_framework",
    "catalog.apps.CatalogConfig",
]
```

# Complete Example 1

Add an app and DRF to a standard project after installing DRF:

```python
# config/settings.py
INSTALLED_APPS = [
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
    "rest_framework",
    "catalog.apps.CatalogConfig",
]
```

**Line-by-line:** Django's built-in apps provide admin, authentication, content types, sessions, messages, and static-file integration. `"rest_framework"` registers DRF's framework integration. The app config registers the project's catalog. **Execution flow:** Django imports settings -> populates app registry -> loads app models and configuration -> initializes request stack. **Internal:** App registration is distinct from installing its Python package. **Professional tip:** Keep this list deliberate and run `manage.py check` after edits.

# Complete Example 2

Define API defaults in settings:

```python
REST_FRAMEWORK = {
    "DEFAULT_PERMISSION_CLASSES": [
        "rest_framework.permissions.IsAuthenticated",
    ],
    "DEFAULT_PAGINATION_CLASS":
        "rest_framework.pagination.PageNumberPagination",
    "PAGE_SIZE": 20,
}
```

This config applies default DRF permissions and page size to API views unless overridden. **Flow:** DRF loads settings -> view policy is resolved -> request is authorized/paginated. **Internal:** Defaults affect many endpoints; a mistaken global permission can expose or block data. **Professional tip:** Configure least privilege and test anonymous and authenticated access explicitly.

# Advanced Example

Read secret deployment values from environment configuration, validate they exist in production, and keep a safe development-only fallback. Never commit production secret keys or passwords. Use separate settings modules only when the deployment workflow clearly benefits from them.

# Common Mistakes

- Committing secret keys or database passwords.
- Leaving debug mode enabled in production.
- Omitting an app from `INSTALLED_APPS`.
- Misconfiguring global DRF permissions or authentication defaults.

# Best Practices

- Separate deploy-time secrets from source code.
- Review `ALLOWED_HOSTS`, HTTPS, cookies, and security settings for production.
- Keep settings deterministic and validate required environment values.
- Run framework checks and permission tests.

# Interview Questions

1. What does `INSTALLED_APPS` do? Registers Django apps.
2. What is middleware? Request/response processing components surrounding views.
3. Where should secrets live? Deployment-managed secret/environment configuration.
4. What can DRF settings configure? Defaults such as permissions, authentication, renderers, and pagination.

# Practice

Register an app and configure an API page size. Explain which settings are global and which can be overridden per view.

# Summary

Settings configure Django's application registry and runtime behavior. Keep them secure, environment-aware, and especially careful with global API defaults.
