# Introduction

Postman is an API client for manually composing HTTP requests, organizing environments, inspecting responses, and automating collections of checks.

# Why this concept exists

It lets developers verify the API boundary independently of a frontend and quickly inspect headers, payloads, status codes, and authentication behavior.

# Real-world analogy

Postman is a workbench with reusable gauges: send a request, observe the response, and repeat the same check across environments.

# Internal Working

Postman sends ordinary HTTP requests. Django receives them just as it would from a mobile or browser client; Postman does not bypass routing, serializers, or permissions.

# Request Lifecycle

Postman request configuration -> network -> Django URL/router -> DRF processing -> HTTP response -> Postman response/test results.

# Syntax

Collection request components: method, URL, headers, query parameters, authorization, body, and tests.

# Complete Example 1

Configure a create request:

```text
Method: POST
URL: {{base_url}}/api/products/
Authorization: Bearer {{access_token}}
Header: Content-Type: application/json
Body (raw JSON):
{
  "name": "Desk lamp",
  "sku": "LAMP-01",
  "price": "24.90",
  "active": true
}
```

`base_url` and `access_token` are environment variables, not hard-coded secrets. Select raw JSON so the body matches DRF's JSON parser. Send, then inspect status, response data, and `WWW-Authenticate` or validation errors as relevant. Keep real tokens out of exported collections.

# Complete Example 2

Add a Postman test:

```javascript
pm.test("created successfully", function () {
  pm.response.to.have.status(201);
  const body = pm.response.json();
  pm.expect(body).to.have.property("id");
  pm.expect(body.sku).to.eql("LAMP-01");
});
```

The test asserts status and response contract. It executes after the response arrives. Use unique test data or a cleanup strategy so repeated runs do not fail due to SKU uniqueness.

# Complete Example 3

Test validation:

```javascript
pm.test("invalid price is rejected", function () {
  pm.response.to.have.status(400);
  pm.expect(pm.response.json()).to.have.property("price");
});
```

Send this test with a malformed price body. The assertion verifies the endpoint rejects invalid input using a field error; exact error shape should match the project's documented contract.

# Advanced Example

Organize a collection as CRUD folders with setup/cleanup requests, use non-production environments, and run collections in CI through a collection runner. Avoid depending on request execution order unless the collection explicitly establishes it.

# Under the Hood

Postman is a client, not a server-side test framework. It can expose CORS, DNS, TLS, auth, and serialization issues that an in-process API test may not reveal, but should complement automated Django tests.

# Performance Notes

Do not use manual Postman runs as a load test. Use dedicated load tools and controlled environments for performance measurements.

# Best Practices

- Separate environment configuration and secrets.
- Use disposable test records and cleanup.
- Assert error paths and headers, not only happy-path status.
- Keep collections reviewed and safe to share.

# Common Mistakes

- Sending form-data when endpoint expects JSON.
- Hard-coding production credentials.
- Assuming a passing manual request replaces automated regression tests.

# Debugging

Check method, URL, path/query values, auth token, content type, body validity, response status/body, server logs, and environment selection.

# Interview Questions

1. What does Postman test? The HTTP contract by sending real requests.
2. Why use variables? Reuse configuration and avoid embedding secrets.
3. Does Postman replace API tests? No; it complements automated tests.

# Practice Exercises

1. Create a collection covering create/list/detail/update/delete.
2. Add negative validation and permission tests.
3. Run the collection against a local test environment with cleanup.

# Chapter Summary

Postman makes API behavior visible and repeatable. Use environments safely, test both success and failure contracts, and retain automated tests as the regression foundation.
