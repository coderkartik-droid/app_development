# Introduction

This capstone outlines a production-oriented document processing API. Authenticated users upload documents, processing occurs asynchronously, and users can check status and access results only when authorized.

# Why this concept exists

A realistic API must coordinate file storage, database state, background tasks, authentication, permissions, and operational visibility.

# Real-world analogy

The API is a secure records office: it accepts a document, checks its owner and format, places it in quarantine, processes it in a back office, and releases an approved result.

# Internal Working

```text
Client -> TLS/auth -> permission -> multipart parser/serializer
-> transaction: document + outbox -> response 202
-> dispatcher/worker: scan/process -> DB status
-> owner-scoped status/download endpoint -> response
```

# Authentication Flow (where applicable)

The client authenticates, uploads under its own identity, receives an operation ID, polls a scoped status route, and downloads only an approved result through an authorized path.

# Syntax

```http
POST /api/v1/documents/
Authorization: Bearer <access-token>
Content-Type: multipart/form-data; boundary=...
```

# Complete Example 1

```python
# models.py
from django.conf import settings
from django.db import models

class Document(models.Model):
    class State(models.TextChoices):
        QUEUED = "queued", "Queued"
        PROCESSING = "processing", "Processing"
        READY = "ready", "Ready"
        REJECTED = "rejected", "Rejected"

    owner = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    source = models.FileField(upload_to="quarantine/%Y/%m/")
    state = models.CharField(max_length=20, choices=State.choices, default=State.QUEUED)
    created_at = models.DateTimeField(auto_now_add=True)
```

The model associates an uploaded file with its owner and a constrained processing state. Quarantine storage should be private. Add size/type metadata, expiry/cleanup, and indexes for common owner/state queries; model paths alone do not grant or deny access.

# Complete Example 2

```python
# serializers.py
from rest_framework import serializers
from .models import Document

class DocumentUploadSerializer(serializers.ModelSerializer):
    class Meta:
        model = Document
        fields = ["id", "source", "state", "created_at"]
        read_only_fields = ["id", "state", "created_at"]

    def validate_source(self, value):
        if value.size > 10 * 1024 * 1024:
            raise serializers.ValidationError("Maximum upload size is 10 MB.")
        return value
```

The serializer accepts a file and keeps state/server timestamps read-only. Size validation is a useful application check but must be backed by proxy/request limits. Add actual content validation and malware scanning; client MIME type is not trustworthy.

# Complete Example 3

```python
# views.py
from django.db import transaction
from rest_framework import generics, status
from rest_framework.parsers import MultiPartParser, FormParser
from rest_framework.permissions import IsAuthenticated
from rest_framework.response import Response
from .models import Document
from .serializers import DocumentUploadSerializer

class DocumentUploadView(generics.GenericAPIView):
    serializer_class = DocumentUploadSerializer
    parser_classes = [MultiPartParser, FormParser]
    permission_classes = [IsAuthenticated]

    def post(self, request):
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        with transaction.atomic():
            document = serializer.save(owner=request.user)
            transaction.on_commit(lambda: enqueue_document_scan(document.pk))
        return Response(self.get_serializer(document).data,
                        status=status.HTTP_202_ACCEPTED)
```

The parser handles multipart; authentication and permission run before the method; serializer validates; transaction writes owner-bound state; `on_commit` queues work only after commit; 202 means asynchronous processing. For guaranteed enqueue, replace the callback-only gap with a transactional outbox. Ensure upload storage cleanup if DB persistence fails.

# Advanced Example

Add an owner-scoped status endpoint and a download endpoint that checks `state == READY` before returning a short-lived storage URL. The worker must be idempotent, record sanitized failure state, and never expose raw scanner output.

# Under the Hood

File storage is not part of the database transaction. A worker consumes a durable task/outbox message, loads the document by ID, updates state, and stores a processed result. Retries require idempotent state transitions.

# Performance Notes

Use proxy body limits, stream/direct uploads for large objects, separate worker queues, cap concurrency, monitor queue age, and paginate document history.

# Security Notes

Use private quarantine storage, generated paths, actual content validation, malware scanning, tenant-scoped queries, signed short-lived downloads, TLS, and safe audit logs.

# Best Practices

- Separate upload, processing, status, and download responsibilities.
- Assign owner server-side.
- Use outbox/idempotency for reliable processing.
- Define retention and cleanup.
- Test every state and access boundary.

# Common Mistakes

- Making a quarantined upload publicly accessible.
- Returning 200 while processing is still queued.
- Trusting file extension or user-provided path.
- Assuming storage writes roll back with database transactions.

# Debugging

Trace request ID and document ID across API logs, DB state, storage key, outbox/broker, and worker logs. Verify current state and owner before investigating content.

# Interview Questions

1. Why return 202 for this upload?
2. How do you avoid lost task enqueue after DB commit?
3. Why is file storage not transactionally coupled to the DB?

# Practice Exercises

1. Implement owner-only status and download endpoints.
2. Add an outbox and idempotent worker state machine.
3. Test oversize, malicious, cross-user, and worker-failure cases.

# Chapter Summary

The capstone combines identity, permissions, multipart validation, private storage, durable async processing, status contracts, and observability. Production readiness depends on explicitly managing failure between independent systems.
