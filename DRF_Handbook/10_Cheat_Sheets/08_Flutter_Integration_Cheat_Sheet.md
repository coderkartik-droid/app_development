# Flutter Integration Cheat Sheet

## Key Concepts

- Flutter UI -> state -> repository -> API client -> HTTP -> DRF.
- Typed models map JSON; repositories hide data-source details.
- HTTP error statuses are responses, not automatic Dart exceptions.
- Native Flutter and Flutter Web have different CORS behavior.

## Important Syntax

```dart
final response = await http.get(
  Uri.https('api.example.com', '/api/v1/products/'),
  headers: {'Accept': 'application/json'},
);
if (response.statusCode != 200) throw ApiException.fromResponse(response);
```

## Frequently Used Classes and Methods

`http.Client`, `Uri.https`, `jsonEncode`, `jsonDecode`, `FutureBuilder`, `MultipartRequest`, `fromJson`, repository interface.

## Common Decorators

Flutter/Dart has no DRF-style endpoint decorators. Use widget annotations only as supported by a chosen package; keep network policy in client/service layer.

## HTTP Status Codes

200 success · 201 created · 202 queued · 204 no body · 400 field errors · 401 invalid/missing auth · 403 forbidden · 404 missing · 429 throttled · 5xx server issue.

## Request/Response Flow

`widget/state -> repository/client -> encode -> HTTP -> DRF -> DB -> JSON/status -> decode/model -> UI state`.

## Serializer Flow

Flutter model `toJson` creates client payload; DRF serializer validates and persists. DRF serializer response maps through Flutter `fromJson`.

## View/ViewSet Comparison

DRF View/APIView/ViewSet determines endpoint handling. Flutter repository hides which endpoint/action is called from the widget.

## Authentication Flow

Login -> token pair -> secure storage -> bearer access header -> DRF authentication -> permission/queryset.

## JWT Flow

Access token for resource calls; refresh token only for renewal. Coordinate refreshes, replace rotated refresh, clear credentials on terminal failure.

## Common Mistakes

- Device `localhost` points to device itself.
- Parsing 204 body.
- Blindly retrying POST.
- Storing refresh tokens in plain preferences.
- Trusting UI visibility for authorization.

## Best Practices

- Configurable base URL, HTTPS, timeouts.
- Typed errors/models and pagination envelopes.
- Centralized auth/error handling.
- Server-side owner/tenant scope.
- Test contract on both sides.

## Interview Revision

Network exception means no response; HTTP 400/500 is a completed response that app must inspect.

## Memory Tricks

- **Check status before decode.**
- **Client suggests; server decides.**

## One-Page Summary

Keep Flutter UI independent from HTTP, align response schemas and pagination with DRF, secure tokens, handle async/loading/error states, and rely on DRF for validation and authorization.
