# Introduction

These exercises help you build and verify authentication and permission behavior rather than relying on configuration assumptions.

# Why this concept exists

Security behavior is best learned by testing both allowed and denied requests using distinct identities.

# Real-world analogy

Practice a key-control system by checking every key against every door, not only confirming that one key opens one door.

# Internal Working

For each test, identify the credential, `request.user`, policy decision, queryset/object result, and response. Test production authentication separately from authorization shortcuts.

# Authentication Flow (where applicable)

```text
client -> auth -> policy -> scoped object -> operation -> response assertion
```

# Syntax

```python
client.force_authenticate(user=alice)
response = client.get("/api/notes/")
```

# Complete Example 1

Create an authenticated owner-only endpoint:

```python
class NoteViewSet(ModelViewSet):
    serializer_class = NoteSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return Note.objects.filter(owner=self.request.user)

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)
```

This implements authentication, list/detail scope, and trusted ownership. Exercise: create Alice and Bob, then prove Bob cannot list or retrieve Alice's note and cannot spoof its owner during creation.

# Complete Example 2

Test object isolation:

```python
client = APIClient()
client.force_authenticate(user=bob)
response = client.get(f"/api/notes/{alice_note.pk}/")
assert response.status_code == 404
```

The example assumes the queryset intentionally hides out-of-scope records. If your contract uses 403, assert that consistently instead. Also test the list endpoint, since detail denial does not prove list filtering.

# Complete Example 3

Test login and refresh:

```python
pair = client.post("/api/token/", credentials, format="json").data
access_response = client.get("/api/private/", HTTP_AUTHORIZATION=f"Bearer {pair['access']}")
refresh_response = client.post("/api/token/refresh/", {"refresh": pair["refresh"]}, format="json")
assert access_response.status_code == 200
assert refresh_response.status_code == 200
```

Use test credentials and assert exact response shape. Add invalid password, expired access, invalid/revoked refresh, and rate-limit tests. Do not include token contents in failure messages.

# Advanced Example

Build a policy matrix for roles and resources. Convert each matrix cell into tests for list, detail, create, update, delete, and custom actions, including anonymous and cross-tenant identities.

# Under the Hood

`force_authenticate` assigns a user for tests without running actual authenticator logic. `APIClient.credentials()` sends headers and exercises the configured authentication class. Use both at appropriate layers.

# Security Notes

Keep fixtures and credentials synthetic; never test against production accounts. Ensure tests fail closed when a permission or authentication class is removed.

# Best Practices

- Test real authenticator and permission logic separately.
- Include cross-user list and detail cases.
- Assert no database mutation after denied/invalid requests.
- Test token expiry/revocation policy.

# Common Mistakes

- Using `force_authenticate` and assuming JWT validation was tested.
- Testing only successful owner access.
- Accepting several status codes where contract should be exact.

# Debugging

Inspect test user state, auth header, URL routing, permission config, queryset, and DB state before/after request.

# Interview Questions

1. What does `force_authenticate` skip?
2. Why test list and detail separately?
3. How verify failed PATCH did not change data?

# Practice Exercises

1. Test owner, non-owner, anonymous, and admin behavior.
2. Test session CSRF enforcement.
3. Test JWT expiry, refresh rotation, and logout.
4. Test role revocation and tenant boundaries.

# Chapter Summary

Security tests need multiple identities and both allowed/denied outcomes. Match test technique to the layer: forced identity for policy, real credentials for authenticator behavior.
