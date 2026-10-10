# APIs Cheat Sheet

## Most Important Concepts

- A request includes method, URI, headers, and sometimes a body.
- A response includes status code, headers, and body.
- Network work is asynchronous and can fail or time out.
- Decode, validate, and map response data before presenting it.

## Request Lifecycle

```text
user action -> repository -> HTTP client -> server
            <- model/error <- decode <- status + headers + body
            -> state -> Flutter UI
```

## Syntax Reference

```dart
final uri = Uri.https('api.example.com', '/v1/items', {'limit': '20'});
final response = await client.get(uri).timeout(const Duration(seconds: 10));
if (response.statusCode != 200) {
  throw Exception('Request failed: ${response.statusCode}');
}
final json = jsonDecode(response.body);
```

## Common Classes / Methods

- `http.Client`, `http.get`, `http.post`
- `Uri.https`, `Uri.http`
- `jsonDecode`, `jsonEncode`
- `Future`, `.timeout`, `Stream`
- Response: `statusCode`, `headers`, `body`, `bodyBytes`

## Parameters & Properties

- `headers`: content type, accept type, authorization, request metadata.
- `body`: serialized request payload.
- `queryParameters`: filter/paging values in URI.
- `timeout`: upper wait bound; may not cancel underlying operation.
- `statusCode`: inspect success and error response classes explicitly.

## Short Example

```dart
final response = await client.post(
  uri,
  headers: {'Content-Type': 'application/json'},
  body: jsonEncode({'title': title}),
);
```

## Common Mistakes

- Assuming every response is successful.
- Parsing JSON without shape/type validation.
- Putting secrets in source control or logs.
- Creating clients repeatedly without clear lifecycle.
- Retrying non-idempotent writes blindly.

## Interview Revision

- GET reads; POST creates/submits; PUT replaces; PATCH partially updates; DELETE removes.
- 2xx indicates success class, not necessarily an expected body shape.
- Bearer credentials belong in authorization headers; protect token storage.
- Repository separates transport from widgets.

## Best Practices

- Use HTTPS, timeouts, cancellation where supported, and typed models.
- Handle loading, empty, malformed, server, and connectivity errors distinctly.
- Keep API keys out of client code when they must be secret.
- Use pagination and debounce search requests.

## Memory Tricks

- Request = **M U H B**: method, URL, headers, body.
- Response = **S H B**: status, headers, body.
- Decode only after checking status and content shape.

## One-Page Summary

Build URIs safely, await requests outside `build`, inspect status, validate decoded data, and update explicit UI state. Use repository boundaries, secure credentials, bounded retries, and server-specific pagination/error contracts.
