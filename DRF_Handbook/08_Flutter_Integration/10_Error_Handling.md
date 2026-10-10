# Introduction

Error handling translates transport failures, HTTP error responses, validation errors, and parsing problems into useful Flutter states.

# Why this concept exists

Network requests fail for many reasons. A blank screen or generic “failed” message gives users no recovery path and developers no diagnostic signal.

# Real-world analogy

A courier may fail because the address is wrong, the recipient is unavailable, or a road is closed; each cause needs a different response.

# Internal Working

Transport exceptions occur before an HTTP response. HTTP status errors carry a response body. JSON/type errors occur after bytes arrive. Flutter should map each category without exposing sensitive server details.

# Data Flow

Flutter request -> network/HTTP/DRF -> status/body -> typed exception mapping -> UI retry/field errors.

# Complete Example 1

Flutter exception model:

```dart
class ApiException implements Exception {
  ApiException(this.statusCode, this.message, {this.fields = const {}});
  final int statusCode;
  final String message;
  final Map<String, dynamic> fields;
}

ApiException parseApiError(http.Response response) {
  final decoded = jsonDecode(response.body);
  if (decoded is Map<String, dynamic>) {
    return ApiException(response.statusCode, 'Request failed',
        fields: decoded);
  }
  return ApiException(response.statusCode, 'Request failed');
}
```

The exception retains status and structured data while using a safe generic message. Production parsing should handle empty/non-JSON bodies and DRF custom envelopes explicitly; do not display arbitrary HTML or traceback text.

# Complete Example 2

DRF validation response:

```python
class ProductSerializer(serializers.ModelSerializer):
    def validate_price(self, value):
        if value < 0:
            raise serializers.ValidationError("Price must not be negative.")
        return value
```

DRF converts the field error into a 400 response. Flutter can map the `price` key to a field-level message. Keep messages safe and stable; database or internal exception details should not be returned.

# Complete Example 3

Flutter request boundary:

```dart
try {
  return await client.get(uri).timeout(const Duration(seconds: 12));
} on TimeoutException {
  throw ApiException(0, 'The request timed out. Please try again.');
} on SocketException {
  throw ApiException(0, 'No network connection is available.');
}
```

Timeout and socket failure have no HTTP status. Map them to user-recoverable states and preserve diagnostic cause internally if needed. Import `dart:async` and `dart:io` where required; web targets need platform-compatible handling.

# Advanced Example

Centralize retry rules: retry transient idempotent GETs with bounded exponential backoff and jitter; do not blindly retry POST. Use idempotency keys for retryable creates and respect 429 Retry-After.

# Under the Hood

The Dart Future can fail with transport exceptions or complete with a response for any HTTP status. A 400/500 response does not automatically throw in the `http` package.

# Performance Notes

Avoid rapid retries and duplicate requests. Cancel obsolete work where the client API supports it.

# Security Notes

Show user-safe messages; do not reveal tokens, tracebacks, or internal hostnames. Keep detailed diagnostics redacted.

# Best Practices

- Separate transport, HTTP, validation, and parsing errors.
- Handle empty bodies and unexpected content types.
- Provide retry/reauthentication actions appropriately.
- Keep error contract aligned with DRF.

# Common Mistakes

- Assuming non-2xx throws automatically.
- Parsing every error body as JSON.
- Retrying non-idempotent writes.

# Debugging

Inspect exception category, response status/content type/body shape, timeout configuration, and server logs with correlation ID.

# Interview Questions

1. Does `package:http` throw for 400?
2. Difference between socket error and HTTP error?
3. When is retry safe?

# Practice Exercises

1. Map DRF field errors to Flutter form fields.
2. Add timeout/offline UI.
3. Implement bounded retry for GET and 429.

# Chapter Summary

Reliable error handling classifies failures and gives users a safe next step. Coordinate a stable DRF error schema with typed Flutter exceptions and cautious retries.
