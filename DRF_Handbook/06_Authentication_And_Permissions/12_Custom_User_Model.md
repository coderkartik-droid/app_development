# Introduction

A custom user model lets a Django project define its identity fields and authentication behavior. If customization is expected, define it at project start before creating migrations that depend on Django's default user.

# Why this concept exists

Products may identify users by email, require additional profile fields, or need a carefully designed user lifecycle.

# Real-world analogy

A standard membership card has fixed fields; a custom user model is designing the membership record for the actual organization before records accumulate.

# Internal Working

Django's `AUTH_USER_MODEL` points to the model used for authentication and relations. `AbstractUser` preserves standard username/password behavior while allowing extension. Foreign keys should reference `settings.AUTH_USER_MODEL`.

# Authentication Flow (where applicable)

Credentials -> configured authentication backend -> custom user record/password hash -> request.user -> permission -> view -> serializer/database -> response.

# Syntax

```python
AUTH_USER_MODEL = "accounts.User"
```

# Complete Example 1

```python
# accounts/models.py
from django.contrib.auth.models import AbstractUser
from django.db import models

class User(AbstractUser):
    display_name = models.CharField(max_length=120, blank=True)
```

`AbstractUser` supplies Django's standard fields, permissions, and password handling. The added profile field is optional. Set `AUTH_USER_MODEL` before initial migrations; changing it later is a complex data migration, not a simple setting edit.

# Complete Example 2

```python
# settings.py
AUTH_USER_MODEL = "accounts.User"
```

The app label and class name identify the model. Ensure the accounts app is installed and its initial migration creates the user model. All model relationships should use `settings.AUTH_USER_MODEL`, not import `User` directly.

# Complete Example 3

```python
# another_app/models.py
from django.conf import settings
from django.db import models

class Note(models.Model):
    owner = models.ForeignKey(
        settings.AUTH_USER_MODEL,
        on_delete=models.CASCADE,
        related_name="notes",
    )
    title = models.CharField(max_length=200)
```

The relation follows the project's configured user model. `related_name` gives a reverse accessor; `CASCADE` means deleting a user deletes notes, which is a business/data-retention decision that may instead require protected or anonymized records.

# Advanced Example

For email-as-identity, subclass `AbstractUser` and carefully configure `USERNAME_FIELD`, manager creation methods, uniqueness, admin integration, and migrations. Test createsuperuser, password reset, authentication, and third-party package compatibility.

# Under the Hood

Django resolves `get_user_model()` from `AUTH_USER_MODEL`. Authentication backends retrieve users and validate hashes; DRF then receives that user from its authenticator.

# Security Notes

Use Django password hashers, never store raw passwords, avoid exposing authentication fields in serializers, and protect account enumeration in login/reset flows.

# Best Practices

- Choose custom user model before first migration.
- Reference it via `settings.AUTH_USER_MODEL`/`get_user_model()`.
- Keep authentication identity separate from profile data when appropriate.
- Test admin, creation, password changes, and permissions.

# Common Mistakes

- Switching user model after production migrations without a migration plan.
- Hard-coding imports of Django's default `User`.
- Returning password/hash fields in API serializers.

# Debugging

Check `AUTH_USER_MODEL`, app label, initial migration dependencies, manager behavior, and whether foreign keys point to the configured model.

# Interview Questions

1. Why define custom user early? Django migrations and relations depend on it.
2. What is `AbstractUser`? An extensible concrete-user base with standard auth behavior.
3. How reference user model in reusable models? `settings.AUTH_USER_MODEL`.

# Practice Exercises

1. Add a display field to an AbstractUser subclass.
2. Connect an API resource to the custom user model.
3. Test user creation and token authentication.

# Chapter Summary

A custom user model is a foundational schema choice. Plan it before migrations, preserve Django's password/auth machinery, and reference it indirectly throughout the project.
