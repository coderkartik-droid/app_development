# Introduction

Testing an API verifies its observable HTTP contract: route, method, status code, response data, headers, and access policy. DRF includes an API-aware test client and test case classes.

# Why this concept exists

An endpoint can appear to work in a browser while returning an incorrect status, shape, or permission result to a real client. Automated tests catch those regressions quickly.

# Real-world analogy

An API test is a rehearsal customer: it places a realistic request and checks that the service returns the promised receipt.

# Internal Working

```text
APITestCase -> APIClient builds request -> Django URL/middleware -> DRF view
            <- response status/data/headers <- dispatch/renderer
```

Django's test framework creates an isolated test database for database-backed test cases. DRF's client supports API-oriented methods, JSON formatting, and authentication helpers.

# Request Lifecycle (where applicable)

The test client dispatches through the same URL/view stack as an application request, but within test configuration. `response.data` exposes DRF's unrendered response data for convenient assertions; rendered content and headers can also be checked.

# Syntax

```python
response = self.client.get("/api/health/")
self.assertEqual(response.status_code, 200)
```

# Complete Example 1

Assuming a named `health` URL resolves to a DRF view returning `{"status": "ok"}`, add this test in an installed app's `tests.py`:

```python
from django.urls import reverse
from rest_framework.test import APITestCase


class HealthApiTests(APITestCase):
    def test_health_returns_expected_contract(self):
        response = self.client.get(reverse("health"))

        self.assertEqual(response.status_code, 200)
        self.assertEqual(response.data, {"status": "ok"})
        self.assertIn("application/json", response["Content-Type"])
```

Run `python manage.py test`.

**Line-by-line:** Import URL reversal and DRF's test case. Define a test class; each `test_` method is discovered by Django's runner. `reverse` resolves the named route, avoiding hard-coded paths. `client.get` makes an in-process request. Assertions verify success, payload, and media type. **Flow:** test -> client -> router -> DRF view -> response -> assertions. **Internal working:** No live network server is required. **Professional tip:** Assert contract behavior, not private implementation details. Removing the status assertion can let an error response with a similar body pass unnoticed.

# Complete Example 2

Test method restriction:

```python
def test_health_rejects_post(self):
    response = self.client.post(reverse("health"), {}, format="json")
    self.assertEqual(response.status_code, 405)
```

The client serializes the body as JSON; DRF should reject POST when only GET is allowed. **Flow:** POST -> method dispatch -> 405 -> assertion. **Tip:** Test behavior for unsupported verbs, not just the happy path.

# Complete Example 3

Test invalid request data on a POST endpoint:

```python
response = self.client.post(
    reverse("create-label"),
    {"name": ""},
    format="json",
)
self.assertEqual(response.status_code, 400)
self.assertIn("name", response.data)
```

`format="json"` encodes the payload and supplies a JSON content type. Assert the error field as part of the API contract while avoiding brittle full-message checks unless exact wording is promised.

# Advanced Example

Use `force_authenticate(user=...)` for focused permission tests where appropriate; separately test the actual authentication mechanism if token/session behavior matters. Test database-backed endpoints with fixtures/factories and verify writes, ownership, and transaction outcomes.

# Under the Hood

`APITestCase` integrates Django's test runner/database lifecycle with DRF's `APIClient`. Requests still pass through URL resolution and DRF dispatch. Tests should be isolated and deterministic.

# Performance Notes

Keep tests fast by using focused fixtures and avoiding unnecessary external services. Test query counts for endpoints where N+1 behavior matters.

# Best Practices

- Use named URLs and `reverse`.
- Cover success, invalid data, missing resources, permissions, and methods.
- Assert status, response shape, and important headers.
- Use real integration tests for authentication/persistence boundaries.

# Common Mistakes

- Testing only that a view function returns a value.
- Forgetting `format="json"` for JSON request bodies.
- Asserting only status while ignoring response contract.
- Sharing mutable global test data across cases.

# Debugging

Read the failure's actual status/body, verify URL names, run one test by dotted path, and check database setup/migrations. Use `response.data` for DRF data and `response.content` for rendered bytes when needed.

# Interview Questions

1. What does `APITestCase` provide? Django test isolation plus DRF-aware client support.
2. Why use `reverse`? Avoid hard-coded routes and follow URL naming.
3. What does `format="json"` do? Encodes request data as JSON with matching media type.
4. What should API tests assert? Status, representation, headers, and policy-relevant behavior.

# Practice Exercises

1. Test a GET success response and content type.
2. Test unsupported method returns 405.
3. Test invalid JSON fields and missing resource returns.

# Chapter Summary

Test APIs through their HTTP boundary using DRF's client. Verify the complete contract—status, data, headers, and policy—and keep tests deterministic and isolated.
