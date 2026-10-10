# Introduction

This chapter reviews interview questions about integrating Flutter clients with DRF backends.

# Why this concept exists

Interviewers want candidates to reason about the complete client-server contract, not only a Flutter package or a DRF class.

# Real-world analogy

An integration engineer explains the whole relay race: each handoff, what is carried, and what happens if a runner drops it.

# Internal Working

Explain client encoding -> network -> DRF routing/auth/policy -> serializer/database -> response status/rendering -> Flutter mapping/state.

# Data Flow

```text
Flutter UI -> repository -> HTTP -> DRF -> serializer/ORM/DB
<- JSON/status <- model mapping <- UI state
```

# Complete Example 1

**Q: How do you handle a paginated GET response?**

```dart
final body = jsonDecode(response.body) as Map<String, dynamic>;
final results = (body['results'] as List)
    .map((e) => Product.fromJson(e as Map<String, dynamic>))
    .toList();
```

DRF page-number pagination commonly wraps records in `results` with count/navigation fields. Flutter should model the envelope and follow `next`; a bare-list assumption breaks as soon as pagination is enabled.

# Complete Example 2

**Q: How do you secure user-owned resources?**

```python
def get_queryset(self):
    return Note.objects.filter(owner=self.request.user)
```

Require authentication, scope list/detail querysets, assign owner from the request identity on creation, and test cross-user access. Hiding a button in Flutter is not a security control.

# Complete Example 3

**Q: What happens when a request times out after POST?**

```text
Client cannot infer whether server committed.
Use idempotency key or query operation state before retrying.
```

The server may have completed the create while the response was lost. Blind retry can duplicate records. Idempotency must be persisted and enforced by DRF/database.

# Advanced Example

**Q: How do you refresh JWTs safely?** Coordinate one refresh for concurrent failures, send refresh only to refresh endpoint, replace rotated credentials atomically, clear credentials on terminal failure, and test expiry/revocation. Never retry forever.

# Under the Hood

Flutter's HTTP Future completes with HTTP responses for error statuses; DRF maps serializer and permission exceptions into responses. Transport exceptions are a separate failure class.

# Performance Notes

Discuss pagination, debouncing, cancellation, payload size, query optimization, and avoiding duplicate requests.

# Security Notes

Discuss HTTPS, secure refresh storage, CSRF for cookie auth, scoped querysets, token redaction, and narrow CORS for web.

# Best Practices

- Answer with both client and server behavior.
- Include status, payload, and failure semantics.
- State which side is authoritative for policy.

# Common Mistakes

- Saying the client can secure a resource.
- Treating JWT as encrypted.
- Ignoring offline/retry ambiguity.

# Debugging

Use request IDs and compare raw HTTP details on both sides.

# Interview Questions

1. What is the client/server request lifecycle?
2. How are DRF validation errors represented?
3. How do PUT and PATCH differ?
4. How should Flutter handle 204?
5. Why is CORS relevant only to browser enforcement?
6. How do you prevent duplicate POST on timeout?
7. Where store refresh tokens?
8. How test real JWT validation?

# Practice Exercises

1. Explain a login, refresh, and protected request.
2. Design upload progress plus server validation.
3. Debug an API returning HTML 404 to a JSON client.

# Chapter Summary

Integration answers should trace both sides of HTTP and cover failure, security, and contract shape. The backend validates and authorizes; Flutter maps results into user experience.
