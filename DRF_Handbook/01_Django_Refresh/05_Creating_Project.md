# Introduction

A Django project is initialized with `startproject`. It creates a management script and a Python package containing settings and root URL configuration.

# Why this concept exists

The framework needs a consistent place for configuration, routing, application registration, and deployment entry points.

# Real-world analogy

Creating a project is laying out the building's shared utilities and directory before individual departments move in.

# Internal Working

`django-admin` writes a project skeleton. `manage.py` sets the project settings module and forwards commands to Django's management system.

```text
manage.py -> settings module -> Django command -> project configuration
```

# Syntax

```bash
django-admin startproject config .
python manage.py runserver
```

# Complete Example 1

In a virtual environment with Django installed, run:

```bash
django-admin startproject config .
python manage.py check
python manage.py runserver
```

Open the local development URL printed by the server. **Line-by-line:** `startproject config .` creates project package `config` in the current directory. `check` validates common configuration. `runserver` starts Django's development server. **Execution:** `manage.py` loads `config.settings`; Django initializes installed apps and middleware; a browser request is routed and answered. **Internal:** `runserver` is for development, not production hosting. **Professional tip:** Keep source control at the project root and configure production settings separately.

# Complete Example 2

Create an app and register it:

```bash
python manage.py startapp inventory
```

```python
# config/settings.py
INSTALLED_APPS = [
    # Existing Django apps...
    "inventory",
]
```

The management command uses this project's settings; adding the app path makes it discoverable. **Flow:** command creates modules -> setting registers app -> later Django startup loads it. **Tip:** Use `"inventory.apps.InventoryConfig"` if you need the app config class explicitly.

# Advanced Example

For multiple environments, separate non-secret settings by environment or use validated environment variables. Keep `SECRET_KEY`, debug mode, allowed hosts, and database credentials out of committed production configuration.

# Common Mistakes

- Creating nested project directories unintentionally by omitting the final dot when desired.
- Running management commands outside the intended project.
- Leaving `DEBUG=True` in production.
- Treating the development server as a production server.

# Best Practices

- Confirm project root and settings module early.
- Keep secrets in deployment configuration.
- Run checks and tests before serving changes.
- Use a production-grade server setup appropriate to deployment.

# Interview Questions

1. What files does `startproject` create? Management entry point and project configuration package.
2. What does `manage.py` do? Selects settings and invokes management commands.
3. Is `runserver` production-ready? No.
4. What are core production settings to review? Debug, secret key, hosts, security, and database configuration.

# Practice

Create a project in a clean directory, inspect its settings and root URLconf, then add an app.

# Summary

`startproject` provides Django's configuration skeleton. Understand the project root, use management commands from it, and treat development settings/server as development-only.
