# Introduction

Installing DRF means adding its Python package to the Django project's environment and registering the framework app in Django settings. This makes DRF modules and optional browsable API integration available.

# Why this concept exists

Python package installation and Django app registration are separate steps. Both are needed for a correctly configured project.

# Real-world analogy

Installing the package brings a tool into the workshop; adding it to the inventory/configuration tells the team to initialize and use it.

# Internal Working

```text
active Python environment -> pip installs package -> INSTALLED_APPS loads integration
                           -> URL/view imports -> DRF runtime
```

`pip` controls which interpreter receives the dependency. Django reads `INSTALLED_APPS` during startup; missing either step causes import or integration errors.

# Request Lifecycle (where applicable)

Installation itself does not process requests. After configuration, an API view is imported through URL routing and DRF dispatches incoming requests.

# Syntax

```bash
python -m pip install djangorestframework
```

# Complete Example 1

Assuming Django is already installed in a project virtual environment:

```bash
python -m pip install djangorestframework
python -m pip show djangorestframework
```

Then update settings:

```python
# config/settings.py
INSTALLED_APPS = [
    # Existing Django applications...
    "rest_framework",
]
```

Run:

```bash
python manage.py check
```

**Line-by-line:** `python -m pip` targets the active interpreter; `install` downloads and installs DRF and dependencies; `show` verifies the installed distribution. `INSTALLED_APPS` registers Django integration. `check` validates project setup. **Flow:** environment -> package importable -> Django app registry initialized -> endpoint can import DRF classes. **Internal working:** Installing alone does not alter Django settings. **Professional tip:** Add the dependency to the project's declared dependency manifest/lock strategy so other environments can reproduce it. Removing `"rest_framework"` may hide app-provided integration such as browsable API templates, even if imports work.

# Complete Example 2

Verify the package using the same interpreter as Django:

```bash
python -c "import rest_framework; print(rest_framework.VERSION)"
python manage.py shell -c "from rest_framework.response import Response; print(Response)"
```

The first command checks the Python module; the second checks import under Django project settings. **Execution:** interpreter resolves package -> import executes -> version/class prints. **Tip:** If one command succeeds and another fails, inspect virtual environment and working directory before reinstalling.

# Complete Example 3

Add a minimal API view after installation:

```python
from rest_framework.decorators import api_view
from rest_framework.response import Response


@api_view(["GET"])
def ping(request):
    return Response({"ping": "pong"})
```

This proves DRF's view and response imports work. It is reachable only when included in a URLconf. Removing `@api_view` changes it back to a regular Django function.

# Advanced Example

For teams, pin compatible Django/DRF versions in a dependency file and test upgrades in CI. Avoid installing random versions globally or changing production dependencies without running the full test suite and reviewing release notes.

# Under the Hood

Django app registration builds an app registry. DRF's settings are read from Django configuration, and imported DRF classes use that runtime context.

# Performance Notes

Package installation has no direct request performance effect. Runtime behavior depends on middleware, authentication, serializers, ORM queries, rendering, and payload size.

# Best Practices

- Use and verify the project's active virtual environment.
- Declare dependencies reproducibly.
- Register `"rest_framework"` in `INSTALLED_APPS`.
- Run Django checks and a small API test after installation.

# Common Mistakes

- Installing with a different Python interpreter.
- Forgetting `INSTALLED_APPS`.
- Typing a package name differently (`djangorestframework` is the distribution; `rest_framework` is the module/app).
- Assuming install success means API URLs are configured.

# Debugging

Check `python -m pip show`, `python -m django --version`, interpreter path, project settings module, and import errors. Run `manage.py check` from the project root.

# Interview Questions

1. What is the pip distribution name? `djangorestframework`.
2. What app path is registered? `"rest_framework"`.
3. Why use `python -m pip`? It installs into the selected interpreter.
4. Is installation enough to expose an endpoint? No; code and URL routing are still required.

# Practice Exercises

1. Install DRF in a virtual environment and verify its version.
2. Register it, run system checks, and import `Response`.
3. Document the dependency in the project's manifest.

# Chapter Summary

Install DRF into the right Python environment, register its Django app, verify imports, and declare the dependency reproducibly. Installation is setup, not endpoint creation.
