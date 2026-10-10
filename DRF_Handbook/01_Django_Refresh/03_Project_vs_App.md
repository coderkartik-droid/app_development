# Introduction

A Django project is the configuration and deployment container for a website. A Django app is a reusable component that provides a focused feature, such as products, accounts, or reporting.

# Why this concept exists

Separating features into apps keeps a growing codebase organized and encourages reusable components. One project can install several apps.

# Real-world analogy

A project is a shopping mall; apps are the independent stores inside it. The mall's shared configuration brings the stores together.

# Internal Working

`startproject` creates project settings and root URL configuration. `startapp` creates app modules. Adding an app's dotted Python path to `INSTALLED_APPS` lets Django discover its models, migrations, admin integration, and app configuration.

```text
project settings + root urls
  ├── products app
  ├── orders app
  └── reporting app
```

# Syntax

```bash
django-admin startproject config
python manage.py startapp catalog
```

# Complete Example 1

From an activated virtual environment, create a project and app:

```bash
python -m pip install django
django-admin startproject config
cd config
python manage.py startapp catalog
```

Register the app:

```python
# config/settings.py
INSTALLED_APPS = [
    # Django apps...
    "catalog",
]
```

**Line-by-line:** Install Django into the selected environment; `startproject` creates project configuration; `cd` enters the project directory; `startapp` creates the app package. `INSTALLED_APPS` tells Django to load the app. **Execution flow:** management command reads project settings -> app module becomes discoverable -> Django includes app features. **Internal:** Apps can contain models, migrations, views, URLs, admin, and tests. **Professional tip:** Choose app boundaries around cohesive domain capabilities, not every individual model.

# Complete Example 2

An app can expose its own routes:

```python
# catalog/urls.py
from django.urls import path
from . import views

urlpatterns = [path("items/", views.item_list, name="item-list")]
```

```python
# config/urls.py
from django.urls import include, path

urlpatterns = [path("catalog/", include("catalog.urls"))]
```

The root URLconf delegates the `catalog/` prefix to the app URLconf. Request `/catalog/items/` is resolved through both patterns. **Internal:** `include` composes route trees. **Tip:** Namespace app routes in larger projects to avoid reverse-name collisions.

# Advanced Example

Use an app config path such as `"catalog.apps.CatalogConfig"` when custom app metadata or startup configuration is needed. Avoid database queries and side effects at import/startup time; application startup should be predictable.

# Common Mistakes

- Calling the whole project an app.
- Forgetting to install the app.
- Making one app for every tiny file or one app for the entire domain.
- Importing app URLs without `include` or a correct prefix.

# Best Practices

- Keep project-level settings and root routing central.
- Give apps cohesive names and responsibilities.
- Namespace routes and use explicit app config when useful.
- Keep reusable apps independent of project-specific assumptions.

# Interview Questions

1. What is a project? The configured Django website/application container.
2. What is an app? A focused component that provides functionality.
3. Can a project have many apps? Yes.
4. What does `INSTALLED_APPS` do? Registers apps for Django discovery and integration.

# Practice

Create `books` and `reviews` apps. Write one sentence explaining each boundary and register both.

# Summary

Projects configure the whole Django application; apps package coherent features. Routes and installed-app configuration connect them.
