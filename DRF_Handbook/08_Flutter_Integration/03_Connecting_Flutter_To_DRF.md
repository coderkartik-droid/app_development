# Introduction

Connecting Flutter to DRF requires a reachable API base URL, a compatible HTTP client, matching routes, and correct platform network configuration.

# Why this concept exists

The development server's `localhost` refers to the device itself, not necessarily the computer hosting Django. Network topology must be understood.

# Real-world analogy

The API is a destination address. A phone and a simulator may be on different roads, so the same hostname does not necessarily reach the same server.

# Internal Working

Flutter resolves the URL through device networking and TLS; Django's host/proxy configuration accepts the request; URL routing selects the endpoint; DRF returns data or an error.

# Data Flow

```text
Flutter base URL -> DNS/device network -> TLS/proxy -> Django host validation
-> URL resolver -> DRF -> response -> Flutter decoder
```

# Complete Example 1

Flutter configuration and request:

```dart
class ApiConfig {
  static const baseUrl = String.fromEnvironment(
    'API_BASE_URL',
    defaultValue: 'http://10.0.2.2:8000/api/v1',
  );
}

Future<http.Response> getProducts() {
  return http.get(Uri.parse('${ApiConfig.baseUrl}/products/'));
}
```

`String.fromEnvironment` allows build-time configuration. `10.0.2.2` is commonly the Android emulator's host-machine alias; iOS simulator/device addresses differ. Use HTTPS and environment-specific values in production; do not ship a development URL.

# Complete Example 2

DRF project configuration:

```python
# settings.py
ALLOWED_HOSTS = ["10.0.2.2", "api.example.com"]

# urls.py
urlpatterns = [
    path("api/v1/", include("catalog.urls")),
]
```

`ALLOWED_HOSTS` validates the request Host header; it is not a CORS setting. The URL prefix must match Flutter's path. Production host names should be narrowly configured and served through trusted HTTPS infrastructure.

# Complete Example 3

Flutter dependency usage:

```dart
import 'package:http/http.dart' as http;

Future<void> checkConnection() async {
  final response = await http
      .get(Uri.parse('${ApiConfig.baseUrl}/health/'))
      .timeout(const Duration(seconds: 10));
  if (response.statusCode != 200) {
    throw StateError('Health check failed: ${response.statusCode}');
  }
}
```

The request is asynchronous and has a timeout. A health route should return minimal information and be intentionally public or protected according to infrastructure needs. Catch and map timeout/network exceptions at the repository boundary.

# Advanced Example

Configure separate development/staging/production endpoints, certificate/TLS policy, proxy requirements, and CORS only for browser-based Flutter Web. Avoid insecure “accept all certificates” callbacks.

# Under the Hood

Android emulator, iOS simulator, physical device, and Flutter Web have different network contexts. Browser CORS applies to Flutter Web; native Flutter HTTP clients are not governed by browser CORS enforcement.

# Performance Notes

Reuse an HTTP client where appropriate, set timeouts, and avoid repeated health checks on every widget rebuild.

# Security Notes

Use HTTPS, restrict Django `ALLOWED_HOSTS`, configure CORS narrowly for web, and never expose internal API secrets in app binaries.

# Best Practices

- Keep base URL configurable.
- Use a device-reachable host during development.
- Match versioned route prefixes exactly.
- Test on emulator and physical devices.

# Common Mistakes

- Using `localhost` on a phone.
- Adding the API hostname to `ALLOWED_HOSTS` and assuming CORS is solved.
- Disabling TLS certificate validation.

# Debugging

Check device network, base URL, DNS, port/firewall, Host header, TLS certificate, route prefix, and browser console for web CORS errors.

# Interview Questions

1. Why does `localhost` fail on a phone?
2. What is Android emulator host alias commonly used for local dev?
3. Does CORS apply equally to native Flutter and Flutter Web?

# Practice Exercises

1. Configure emulator and physical-device development URLs.
2. Add a health check with timeout.
3. Configure narrow CORS for a web frontend.

# Chapter Summary

Connection issues are often topology or configuration mismatches. Configure reachable URLs per platform, align Django hosts/routes, and use secure transport.
