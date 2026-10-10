# Introduction

API testing verifies routing, authentication, permissions, validation, persistence, status codes, and response contracts.

# Why this concept exists

Security and integration defects often occur between layers, so testing only serializers or view methods cannot prove the full request behavior.

# Real-world analogy

Testing is a rehearsal with actors playing authorized users, strangers, faulty clients, and overloaded dependencies.

# Internal Working

DRF's `APIClient` sends requests through Django's URL and view stack in-process. `force_authenticate` injects identity for policy tests; actual headers/cookies test authenticators.

# Authentication Flow (where applicable)

Test request -> authenticator or injected identity -> permission -> view/queryset/serializer/database -> response assertions.

# Syntax

```python
response = self.client.post(path, payload, format="json")
self.assertEqual(response.status_code, 201)
```

# Complete Example 1

```python
from rest_framework.test import APITestCase
from django.contrib.auth import get_user_model
from .models import Note

class NoteAccessTests(APITestCase):
    def test_user_cannot_read_another_users_note(self):
        User = get_user_model()
        owner = User.objects.create_user(username="owner", password="test-pass")
        stranger = User.objects.create_user(username="stranger", password="test-pass")
        note = Note.objects.create(owner=owner, title="Private")
        self.client.force_authenticate(user=stranger)
        response = self.client.get(f"/api/notes/{note.pk}/")
        self.assertEqual(response.status_code, 404)
```

The test creates separate identities and a private record, authenticates as the stranger, requests the detail route, and asserts the chosen concealment behavior. Add list, mutation, anonymous, and owner-positive cases; ensure app URLs/model setup match.

# Complete Example 2

```python
def test_invalid_note_title_is_rejected(self):
    self.client.force_authenticate(user=self.user)
    response = self.client.post("/api/notes/", {"title": ""}, format="json")
    self.assertEqual(response.status_code, 400)
    self.assertIn("title", response.data)
    self.assertEqual(Note.objects.count(), 0)
```

This checks response error shape and absence of persistence. The fixture must start with an empty or otherwise accurately counted table. Tests should assert public contract and durable effects.

# Complete Example 3

```python
from rest_framework.test import APIClient

client = APIClient()
client.credentials(HTTP_AUTHORIZATION=f"Bearer {access_token}")
response = client.get("/api/private/")
assert response.status_code == 200
```

This sends a real auth header through configured authentication, unlike `force_authenticate`. Obtain a synthetic token in setup and test invalid/expired tokens as separate cases.

# Advanced Example

Use unit tests for policy/service logic, API integration tests for contracts, schema checks for docs, and targeted performance/security tests. Avoid brittle timing thresholds in shared CI.

# Under the Hood

Django's test runner isolates database state and wraps tests according to test class behavior. `APIClient` skips the network stack but exercises Django request handling and DRF dispatch.

# Performance Notes

Keep tests deterministic and targeted. Use realistic factories, avoid slow external calls, and measure query counts where regression risk exists.

# Security Notes

Never use production credentials/data. Test unauthorized and cross-tenant behavior, CSRF for sessions, and token revocation.

# Best Practices

- Test success and failure contracts.
- Test authentication separately from authorization.
- Assert database state and response shape.
- Mock external systems at clear boundaries.

# Common Mistakes

- Using `force_authenticate` as proof JWT works.
- Testing only happy paths.
- Allowing broad status alternatives that hide regressions.

# Debugging

Inspect request URL, auth state, response data, test DB fixtures, and logs. Reproduce via APIClient with the smallest case.

# Interview Questions

1. Difference between APIClient and `force_authenticate`?
2. What should an authorization test assert?
3. Why test persisted state after invalid input?

# Practice Exercises

1. Test login, refresh, and logout lifecycle.
2. Test all role/resource combinations.
3. Add schema and query-count regression checks.

# Chapter Summary

API tests validate cross-layer behavior. Use the right test mechanism for credentials, assert both response and persistence, and include adversarial access cases.
