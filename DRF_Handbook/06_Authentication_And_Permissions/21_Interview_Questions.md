# Introduction

This chapter reviews interview topics across DRF authentication mechanisms, permission hooks, JWT lifecycle, and secure resource access.

# Why this concept exists

Security interviews assess whether a developer can reason about real request boundaries rather than merely name authentication classes.

# Real-world analogy

Explaining API security is like explaining an airport journey: identify each checkpoint and what it verifies.

# Internal Working

Structure answers as identity mechanism -> policy check -> object visibility -> credential lifecycle -> failure response and test strategy.

# Authentication Flow (where applicable)

```text
credential -> authenticator -> request.user/auth -> permissions
-> scoped view -> serializer/database -> response
```

# Syntax

```python
authentication_classes = [JWTAuthentication]
permission_classes = [IsAuthenticated]
```

# Complete Example 1

**Q: Authentication versus authorization?**

```python
request.user  # identity established by authentication
permission.has_permission(request, view)  # access policy decision
```

Authentication produces identity; authorization evaluates allowed action/resource. A logged-in user may still lack access. In an owner-scoped API, use both `IsAuthenticated` and a queryset filtered by `request.user`.

# Complete Example 2

**Q: Why are object permissions not enough for lists?**

```python
def get_queryset(self):
    return Document.objects.filter(owner=self.request.user)
```

DRF generally checks object permissions when a detail object is retrieved, not against every row in list serialization. Filter the queryset to prevent list data leakage and reduce unnecessary database work.

# Complete Example 3

**Q: How do access and refresh tokens differ?**

```text
Access: short-lived, sent to resource APIs
Refresh: longer-lived, sent only to token refresh endpoint
```

The separation limits exposure while preserving user sessions. Refresh tokens need stronger storage, rotation, and revocation policy. JWT signatures do not encrypt claims, and logout may not invalidate outstanding access tokens.

# Advanced Example

**Q: How do you prevent race-condition duplicates?** Use serializer validation for useful feedback, a database uniqueness constraint for authoritative integrity, and safe handling of expected conflicts. Application existence checks alone are not concurrency-safe.

# Under the Hood

DRF calls authentication before permissions during request initialization. Generic object retrieval invokes object checks; list queryset scoping remains the view's responsibility.

# Security Notes

Discuss TLS, CSRF for sessions, secure secret storage, token lifetime/revocation, rate limiting, and safe logs when answering production design questions.

# Best Practices

- Explain tradeoffs rather than reciting definitions.
- Distinguish 401 and 403 in context of configured authenticators.
- Include tenant/object scope and adversarial tests.
- Never claim JWT is encrypted or automatically revocable.

# Common Mistakes

- Saying authentication grants ownership.
- Claiming object permissions filter lists.
- Treating Basic Base64 as encryption.

# Debugging

For a failing request, trace credential parsing, request identity, effective policy, scoped lookup, and denial/exception response.

# Interview Questions

1. Compare session, Basic, DRF token, and JWT authentication.
2. What do `has_permission` and `has_object_permission` do?
3. Why are access tokens short-lived?
4. What is refresh rotation?
5. Does logout revoke a stateless JWT?
6. What does `IsAdminUser` check?
7. Why define a custom user model early?
8. How prevent role escalation?

# Practice Exercises

1. Explain a JWT login-to-refresh lifecycle.
2. Design a permission matrix for customer, support agent, and admin.
3. Review an endpoint for IDOR and token leakage.

# Chapter Summary

Strong answers connect authentication to identity, permissions to policy, querysets to visibility, and token design to a realistic lifecycle and threat model.
