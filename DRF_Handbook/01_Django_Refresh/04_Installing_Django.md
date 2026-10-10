# Introduction

Installing Django means adding the framework to a Python environment isolated for a project. A virtual environment prevents one project's dependency versions from interfering with another's.

# Why this concept exists

Reproducible dependencies make development, testing, and deployment reliable. Installing packages globally can create version conflicts and hidden machine-specific assumptions.

# Real-world analogy

A virtual environment is a dedicated toolbox for one workshop. It keeps one project's tools separate from another project's tools.

# Internal Working

Python environments choose which interpreter and package directory a command uses. `pip` installs Django into that environment; `django-admin` becomes available from its scripts directory.

```text
create venv -> activate -> pip install Django -> django-admin/manage.py
```

# Syntax

```bash
python -m venv .venv
python -m pip install django
python -m django --version
```

# Complete Example 1

Run the following from a new project directory. On Windows PowerShell, activation is `.\.venv\Scripts\Activate.ps1`; on macOS/Linux, use `source .venv/bin/activate`.

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install django
python -m django --version
django-admin startproject config .
python manage.py check
```

**Line-by-line:** `venv` creates an isolated interpreter environment. Activation changes command lookup for the current shell. `python -m pip` ensures pip belongs to that interpreter. The version command verifies installation. `startproject config .` creates a project in the current folder. `manage.py check` runs Django's system checks. **Flow:** shell selects venv -> pip installs framework -> management command loads Django -> checks settings and app configuration. **Professional tip:** Record dependencies in a project manifest/lock strategy before collaborating; do not rely on an undocumented global install.

# Complete Example 2

If activation is unavailable, invoke the environment's interpreter directly:

```powershell
.\.venv\Scripts\python.exe -m pip install django
.\.venv\Scripts\python.exe -m django --version
```

This targets the local environment without changing shell activation. The second command verifies the package is visible to the same interpreter. **Internal:** Python resolves modules from the interpreter's environment. **Tip:** Use one interpreter consistently in IDE, terminal, tests, and deployment.

# Advanced Example

Install DRF later into the same environment with `python -m pip install djangorestframework`, then register `"rest_framework"` in `INSTALLED_APPS`. Keep setup commands in the project onboarding instructions; never commit environment-specific absolute paths.

# Common Mistakes

- Using `pip` from a different Python interpreter.
- Forgetting to activate the environment.
- Installing into the system interpreter unintentionally.
- Committing `.venv` to source control.

# Best Practices

- Use a dedicated virtual environment per project.
- Use `python -m pip` for interpreter certainty.
- Pin and review dependencies according to team policy.
- Run `python manage.py check` after setup.

# Interview Questions

1. Why use a virtual environment? Isolate dependencies per project.
2. Why `python -m pip`? It binds pip to the selected Python interpreter.
3. What verifies Django installation? `python -m django --version`.
4. What does `manage.py check` do? Runs framework system checks.

# Practice

Create a fresh virtual environment, install Django, and verify the interpreter path and framework version.

# Summary

Use an isolated Python environment, install with its interpreter, verify Django, then create and check the project. Reproducible setup is part of maintainable software.
