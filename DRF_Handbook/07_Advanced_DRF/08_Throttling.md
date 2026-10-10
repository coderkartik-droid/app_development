# Introduction

Throttling limits request rates for users or anonymous clients. DRF throttles are useful for fairness and basic abuse reduction, but are not a complete denial-of-service defense.

# Why this concept exists

Unbounded clients can exhaust database, worker, or third-party service capacity and degrade the API for everyone.

# Real-world analogy

A venue admits visitors at a safe rate so the entrance and staff are not overwhelmed.

# Internal Working

DRF throttle classes inspect identity/request history and may deny a request with 429 Too Many Requests. Built-in cache-based throttles are approximate under concurrency and should be paired with edge controls for strong enforcement.

# Authentication Flow (where applicable)

Request -> authentication -> permission -> throttle check -> view/service/database -> response or 429 with retry metadata.

# Syntax

```python
throttle_classes = [UserRateThrottle]
```

# Complete Example 1

```python
# settings.py
REST_FRAMEWORK = {
    "DEFAULT_THROTTLE_CLASSES": [
        "rest_framework.throttling.AnonRateThrottle",
        "rest_framework.throttling.UserRateThrottle",
    ],
    "DEFAULT_THROTTLE_RATES": {
        "anon": "60/hour",
        "user": "1000/hour",
    },
}
```

Anonymous and authenticated users receive different rates. Rates are policy examples; tune by endpoint cost and business need. Configure a shared production cache if multiple workers must coordinate; local-memory caches do not provide global rate limits.

# Complete Example 2

```python
from rest_framework.throttling import ScopedRateThrottle
from rest_framework.views import APIView

class PasswordResetRequestView(APIView):
    throttle_classes = [ScopedRateThrottle]
    throttle_scope = "password_reset"

    def post(self, request):
        ...
```

The endpoint declares a named scope; configure `"password_reset"` under throttle rates. The omitted body must validate email and return non-enumerating results. Throttling should include network-edge controls and monitoring due to the endpoint's abuse sensitivity.

# Complete Example 3

```python
response = APIClient().get("/api/products/")
assert response.status_code in (200, 429)
```

This loose assertion is suitable only for a demonstration; deterministic tests should configure a very small test rate and assert exact first/next outcomes. Reset or isolate cache between tests to avoid cross-test state.

# Advanced Example

For distributed enforcement, use a trusted API gateway or centralized rate-limit store, keying carefully by authenticated identity and safe network identity. Account for proxies and spoofed forwarded headers.

# Under the Hood

Built-in throttles use cache-backed request history and return a wait estimate. They are not intended as precise billing counters and can have race conditions.

# Performance Notes

Throttle checks add cache work to each request. Use shared low-latency cache and avoid overly granular scopes that create excessive keys.

# Security Notes

Rate limits are one layer. Avoid exposing account existence, protect login/reset flows, and never trust client-controlled IP headers unless set by a trusted proxy.

# Best Practices

- Set endpoint-specific rates for expensive/sensitive operations.
- Use shared distributed infrastructure in multi-worker deployments.
- Return 429 and honor Retry-After behavior.
- Monitor throttle patterns and false positives.

# Common Mistakes

- Treating DRF throttling as DDoS protection.
- Using per-process memory cache in a cluster.
- Applying one rate to all endpoints regardless of cost.

# Debugging

Inspect throttle class/scope, rate config, cache backend, identity key, proxy headers, and 429 response.

# Interview Questions

1. What status indicates throttling? 429.
2. Why are built-in throttles approximate?
3. Why use an edge rate limiter too?

# Practice Exercises

1. Set separate anonymous and authenticated rates.
2. Add a stricter login/reset scope.
3. Test rate behavior with isolated cache.

# Chapter Summary

Throttling controls normal request volume and discourages abuse. Use shared infrastructure and endpoint-aware policy, while relying on network-edge defenses for serious traffic attacks.
