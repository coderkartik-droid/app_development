# Introduction

Postman is a tool for manually constructing, sending, and inspecting HTTP requests. It is useful for exploring an API contract before writing Flutter code.

# Why this concept exists

Separating API investigation from UI implementation helps verify endpoints, authentication, headers, and payload shapes quickly.

# Real-world analogy

Postman is a test bench for a machine. You can send controlled inputs and inspect outputs before connecting the machine to a full product.

# Internal Working

You configure method, URL, query, headers, authentication, and body. Postman sends a request and displays response status, headers, body, and timing. Collections and environments organize requests and variables. Secrets in environments must be protected and never committed publicly.

# Syntax

Example request setup:

```text
GET https://api.example.com/v1/products?limit=20
Accept: application/json
Authorization: Bearer <temporary test token>
```

# Complete Example 1

Postman itself is an external desktop/web tool, so this is a reproducible request recipe rather than a Flutter `main.dart`. Create a request with these settings:

```text
Method: GET
URL: https://jsonplaceholder.typicode.com/posts/1
Headers:
  Accept: application/json
Body: none
```

### Step-by-step explanation

- Select GET because the goal is to retrieve one post.
- Enter the HTTPS endpoint and resource path.
- Set `Accept` to express that JSON is expected.
- Leave the body empty for this GET.
- Send and inspect status, response headers, body, and timing.
- Compare the response shape with the Dart model you plan to write.

### Widget tree

No Flutter widget tree applies; Postman provides its own request editor and response viewer.

### Execution flow

```text
Postman request editor -> HTTP endpoint -> response panel
                                      -> inspect status/headers/body
                                      -> implement same contract in Dart
```

### ASCII UI preview

```text
+--------------------------------+
| GET  https://.../posts/1 [Send]|
+--------------------------------+
| 200 OK                         |
| { "userId": 1, "id": 1, ... }  |
+--------------------------------+
```

### Example explanation

The purpose is to discover the exact request and response contract before embedding it in an app. Do not paste production credentials into a shared workspace or save them in a collection that will be committed.

**Common mistakes:** A request succeeding in Postman does not prove it will succeed in Flutter; mobile platform networking, CORS on web, certificates, and auth configuration can differ.

# Complete Example 2

Create a JSON POST request:

```text
Method: POST
Header: Content-Type: application/json
Body (raw JSON): {"title":"Example"}
```

Inspect validation errors and the documented success response.

# Complete Example 3

Use a Postman environment variable:

```text
{{baseUrl}}/v1/profile
```

Keep secrets in protected local variables and avoid exporting them to source control.

# Advanced Example

Use collections for regression requests, pre-request scripts for test-only setup, and tests for status/schema checks. Do not treat a manually passing request as a substitute for automated Flutter repository tests or backend contract tests.

# Under the Hood

Postman constructs a standard HTTP request and displays the response. It may add defaults, follow redirects, or use environment settings, so compare the actual request details when reproducing behavior elsewhere.

# Performance Notes

- Use saved examples and collections to reduce repeated manual setup.
- Keep test payloads representative but not sensitive.
- Avoid running destructive requests against production environments.

# Best Practices

- Confirm API contract and status codes.
- Use separate development/staging environments.
- Keep credentials protected.
- Export collections without secrets.
- Convert discoveries into automated tests/documentation.

# Common Mistakes

- Testing against production by accident.
- Committing tokens in exported collections.
- Ignoring request headers Postman adds.
- Assuming desktop CORS behavior matches browser Flutter.
- Skipping automated tests after manual verification.

# Debugging

- Compare exact method, URI, headers, body, and redirects.
- Check environment variables resolve correctly.
- Inspect response status and raw body.
- Reproduce on the target device/runtime.

# Interview Questions

1. What is Postman used for? Manual API request construction and inspection.
2. Can it replace automated tests? No.
3. Why use environments? Separate base URLs/configuration.
4. What security risk exists in collections? Accidentally exporting secrets.

# Try It Yourself

1. Test a GET and a POST endpoint.
2. Save a collection with no secrets.
3. Compare Postman request headers to Flutter.
4. Add a response status assertion.

# Challenge Project

Create a safe Postman collection for a sample catalog API, with environment variables, CRUD requests, schema assertions, and no committed credentials.

# Quick Revision Sheet

- Use Postman to explore request/response contracts.
- Record method, URL, headers, body, status.
- Use environments for base URLs.
- Protect secrets; avoid production destructive tests.
- Follow manual exploration with automated tests.
