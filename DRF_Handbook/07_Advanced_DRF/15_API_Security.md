# Introduction

API security protects identities, data, and operations from unauthorized access, misuse, and exploitation.

# Why this concept exists

Clients can call endpoints directly and modify requests; a secure system cannot rely on client-side visibility or good intentions.

# Real-world analogy

A bank protects money with identity verification, transaction limits, account-specific rules, audit records, and secure transport.

# Internal Working

Security controls span transport, authentication, authorization, validation, database integrity, rate limits, secrets, logging, and deployment configuration.

# Authentication Flow (where applicable)

HTTPS -> credential verification -> permission/query scope -> input validation -> database/domain checks -> safe output and audit.

# Syntax

```python
permission_classes = [IsAuthenticated]

def get_queryset(self):
    return Invoice.objects.filter(account__members=self.request.user)
```

# Complete Example 1

```python
class InvoiceViewSet(ModelViewSet):
    serializer_class = InvoiceSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return Invoice.objects.filter(account__members=self.request.user).distinct()

    def perform_create(self, serializer):
        serializer.save(created_by=self.request.user)
```

Authentication establishes a caller, queryset restricts tenant visibility, `.distinct()` handles duplicate joins through membership, and create assigns creator server-side. Add write-specific role policy and tests. Do not expose account selection from arbitrary client data.

# Complete Example 2

```python
class PaymentInputSerializer(serializers.Serializer):
    invoice_id = serializers.IntegerField()
    idempotency_key = serializers.UUIDField()

    def validate_invoice_id(self, value):
        if value <= 0:
            raise serializers.ValidationError("Invalid invoice.")
        return value
```

The serializer validates types and defines a retry identity. The view must still scope invoice lookup to the caller and enforce payment state. Idempotency keys must be persisted with a unique constraint and payment operation, not merely accepted.

# Complete Example 3

```python
def safe_audit_log(logger, event, request_id, user_id):
    logger.info(event, extra={"request_id": request_id, "user_id": user_id})
```

This logs minimal identifying context without credentials or request bodies. Apply retention/access controls and pseudonymize where required. Never log authorization headers, passwords, or payment secrets.

# Advanced Example

Use threat modeling and defense in depth: TLS, secure CORS, CSRF for cookies, permissions, tenant scope, validation, quotas, dependency patching, secret management, audit, backups, and incident response.

# Under the Hood

DRF authentication and permissions run before view handling, while database constraints protect integrity against races and alternate code paths. Reverse proxies and cloud identity boundaries also affect security.

# Performance Notes

Abuse controls should bound expensive operations. Keep permission queries efficient without bypassing policy.

# Security Notes

Use production `DEBUG=False`, secure cookies, HSTS at TLS termination, trusted proxy configuration, strict allowed hosts, least-privilege DB credentials, and secret rotation.

# Best Practices

- Deny by default and scope resources.
- Validate client data and enforce DB constraints.
- Keep secrets out of code/logs.
- Test abuse and cross-tenant scenarios.
- Patch dependencies and monitor suspicious activity.

# Common Mistakes

- Treating CORS as API authorization.
- Trusting UUIDs as access control.
- Returning detailed exception traces.
- Disabling CSRF for session-authenticated APIs.

# Debugging

Reproduce with exact identity and request; inspect security settings, policy chain, queryset, headers, proxy/TLS, and sanitized logs.

# Interview Questions

1. What is an IDOR?
2. Why are database constraints security-relevant?
3. How do CSRF and CORS differ?

# Practice Exercises

1. Threat-model a tenant invoice API.
2. Audit writable serializer fields for privilege escalation.
3. Create secure deployment settings checklist.

# Chapter Summary

API security is layered and must be enforced server-side. Protect transport, credentials, object visibility, writes, secrets, operational configuration, and observability as one system.
