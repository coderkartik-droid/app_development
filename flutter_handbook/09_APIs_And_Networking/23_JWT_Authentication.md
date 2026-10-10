# Introduction

JWT (JSON Web Token) is a compact token format containing claims. A JWT is commonly signed so a receiver can verify that claims have not been altered. A signed JWT is not automatically encrypted.

# Why this concept exists

Services need a portable way to convey identity and authorization claims between systems without querying a session store on every request, depending on architecture.

# Real-world analogy

A JWT resembles a signed event ticket: the printed claims describe the ticket, and the signature lets the venue verify it was issued by a trusted authority. The signature does not hide what is printed.

# Internal Working

A compact JWT has header, payload, and signature sections separated by dots. Header and payload are Base64URL-encoded, not secret. The server verifies signature using trusted keys and checks issuer, audience, expiry, not-before, and required claims. Client-side decoding can help display information but cannot validate trust.

# Syntax

```text
header.payload.signature
```

# Complete Example 1

This runnable Flutter app demonstrates why JWT payload content must be treated as readable. It decodes a fabricated, unsigned token for teaching only. Never use this function to authenticate or authorize a user.

```dart
import 'dart:convert';
import 'package:flutter/material.dart';

String decodeDemoPayload(String token) {
  final parts = token.split('.');
  if (parts.length != 3) throw const FormatException('Malformed JWT');
  final normalized = base64Url.normalize(parts[1]);
  final bytes = base64Url.decode(normalized);
  return utf8.decode(bytes);
}

void main() => runApp(const JwtApp());

class JwtApp extends StatelessWidget {
  const JwtApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: JwtPage());
}

class JwtPage extends StatelessWidget {
  const JwtPage({super.key});

  @override
  Widget build(BuildContext context) {
    const payload = '{"sub":"user-7","role":"reader","exp":2000000000}';
    final encoded = base64Url.encode(utf8.encode(payload)).replaceAll('=', '');
    final fakeToken = 'eyJhbGciOiJub25lIn0.$encoded.signature';
    final readable = decodeDemoPayload(fakeToken);
    return Scaffold(
      appBar: AppBar(title: const Text('JWT is not encryption')),
      body: Padding(
        padding: const EdgeInsets.all(20),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Text('Decoded demo payload:'),
            Text(readable),
            const Text('Decoding does not verify the signature.'),
          ],
        ),
      ),
    );
  }
}
```

### Line-by-line explanation

- `dart:convert` provides Base64URL and UTF-8 operations.
- `decodeDemoPayload` checks for three segments and decodes only the payload segment.
- `base64Url.normalize` restores required padding for decoding.
- The sample constructs a fabricated unsigned token; the signature text is not valid.
- The UI displays payload and explicitly warns that no signature is verified.

### Widget tree

```text
MaterialApp → JwtPage → Scaffold → Padding → Column
                                         ├→ label
                                         ├→ decoded payload
                                         └→ security warning
```

### Execution flow

```text
JWT text -> split segments -> Base64URL decode payload -> readable claims
          (no signature verification occurs)
```

### ASCII UI preview

```text
+---------------------------+
| JWT is not encryption     |
+---------------------------+
| {"sub":"user-7",...}      |
| Decoding does not verify  |
+---------------------------+
```

### Example explanation

JWT payloads can be read by anyone who has the token; Base64URL is encoding. Trust comes from signature verification and claim checks performed by a trusted server or vetted auth library.

If signature verification is omitted and decoded claims are used for authorization, an attacker can alter claims. If expiry is ignored, old tokens may be accepted incorrectly.

**Common mistakes:** Never store passwords or secrets in JWT payload claims.

# Complete Example 2

Send a token to an API:

```dart
headers: {'authorization': 'Bearer $jwt'}
```

The API must verify it. The client should not treat local decoding as proof.

# Complete Example 3

Inspect expiry for UX:

```text
exp claim -> compare with current time -> request refresh when policy permits
```

The server still makes the authoritative validation decision.

# Advanced Example

Use vetted protocol libraries for OIDC/JWT validation. Check algorithm allowlists, key rotation, issuer, audience, expiry, not-before, and key ID behavior. Reject `none` or unexpected algorithms according to the trusted validation library and policy.

# Under the Hood

Base64URL encodes bytes into URL-safe text. The signature covers header and payload according to the selected algorithm. Signing provides integrity/authenticity, not confidentiality. JWE is a separate encrypted-token format.

# Performance Notes

- Avoid repeatedly parsing tokens unnecessarily.
- Do not use client-side decoding as a security check.
- Keep claims small.
- Cache validated session state carefully, respecting expiry.

# Best Practices

- Use standard auth flows and maintained libraries.
- Validate tokens on trusted server.
- Check issuer, audience, expiry, and signature.
- Treat payload as visible.
- Use secure storage and HTTPS.

# Common Mistakes

- Confusing Base64URL with encryption.
- Trusting decoded role claims in client.
- Ignoring audience/issuer/expiry.
- Accepting arbitrary algorithms.
- Logging tokens.

# Debugging

- Check token structure without publishing token contents.
- Use trusted JWT inspection tools only with test tokens.
- Verify clock skew, issuer, audience, and key rotation.
- Inspect server validation logs via correlation IDs.

# Interview Questions

1. What are JWT's three compact sections? Header, payload, signature.
2. Is a signed JWT encrypted? No.
3. Where should signature validation occur? At a trusted resource server/auth component.
4. What is `exp`? Expiration-time claim.
5. What is audience validation? Ensuring the token was issued for the intended recipient.

# Try It Yourself

1. Decode a test token and identify readable claims.
2. Explain why a changed role claim invalidates a correctly verified signature.
3. List issuer/audience/expiry checks.
4. Remove token logging from a mock interceptor.

# Challenge Project

Design a JWT-backed API session using an established OIDC client. Document claim validation, token refresh, secure storage, key rotation, and sign-out.

# Quick Revision Sheet

- JWT is `header.payload.signature`.
- Base64URL is encoding, not encryption.
- Signature and claims must be validated by trusted code.
- Validate issuer, audience, time, and algorithm.
- Never put secrets in the payload.
