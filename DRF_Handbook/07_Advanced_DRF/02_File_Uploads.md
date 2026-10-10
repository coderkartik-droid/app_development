# Introduction

File uploads let clients send binary content such as documents or exports to an API. The application must validate metadata, enforce size/type policy, store the file safely, and expose it deliberately.

# Why this concept exists

Many products need user-submitted documents, avatars, receipts, or attachments that do not fit ordinary JSON fields.

# Real-world analogy

Uploading is receiving a parcel: check its label, weight, contents policy, and destination before putting it into secure storage.

# Internal Working

An HTTP multipart request is parsed into ordinary form values plus uploaded-file objects. A serializer validates the file field; Django storage writes bytes to the configured backend and persists a name/path in the model.

# Syntax

```python
parser_classes = [MultiPartParser, FormParser]
```

# Complete Example 1

```python
# models.py
from django.conf import settings
from django.db import models

class Attachment(models.Model):
    owner = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    file = models.FileField(upload_to="attachments/%Y/%m/")
    uploaded_at = models.DateTimeField(auto_now_add=True)
```

`FileField` stores a storage-relative name in the database; bytes are managed by the configured storage backend. The date-based path organizes files but is not access control. Apply migrations and configure private storage and authenticated download policy for sensitive files.

# Complete Example 2

```python
from rest_framework import serializers, generics
from rest_framework.parsers import MultiPartParser, FormParser
from rest_framework.permissions import IsAuthenticated
from .models import Attachment

class AttachmentSerializer(serializers.ModelSerializer):
    class Meta:
        model = Attachment
        fields = ["id", "file", "uploaded_at"]
        read_only_fields = ["id", "uploaded_at"]

class AttachmentUploadView(generics.CreateAPIView):
    serializer_class = AttachmentSerializer
    parser_classes = [MultiPartParser, FormParser]
    permission_classes = [IsAuthenticated]

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)
```

The multipart parser handles browser/file-client submissions. The serializer accepts file input and hides owner; `perform_create` assigns ownership from identity. Add size/type validation, malware scanning if appropriate, quotas, and private storage. Without parsers, multipart data may not be decoded as intended.

# Complete Example 3

```python
response = APIClient().post(
    "/api/attachments/",
    {"file": SimpleUploadedFile("receipt.txt", b"sample", content_type="text/plain")},
    format="multipart",
)
assert response.status_code == 201
```

Authenticate the client for the protected endpoint and include the required owner association through the view. This test uses a small synthetic file; also test oversize, empty, invalid type, and unauthorized uploads.

# Advanced Example

For large files, use direct-to-object-storage uploads with short-lived signed upload policy, then verify ownership and completion server-side. Scan or quarantine files before making them available.

# Under the Hood

Django's upload handlers may keep small files in memory and larger files on disk. Storage saves the file and returns a name; the model stores that reference. Database rollback does not necessarily remove an already-written external object.

# Performance Notes

Set request/body limits at proxy and application layers, stream/chunk large content, and avoid reading entire files into memory.

# Security Notes

Do not trust filename, extension, or client MIME type. Generate storage names, prevent path traversal, use private storage for private files, validate access on download, and scan untrusted content.

# Best Practices

- Enforce size, count, type, and ownership limits.
- Use safe generated names and controlled storage paths.
- Keep private files out of public web roots.
- Define cleanup when DB writes fail or records are deleted.

# Common Mistakes

- Treating file extension as proof of content type.
- Exposing storage URLs without access checks.
- Forgetting storage/database operations are not one transaction.

# Debugging

Inspect request `Content-Type`, parser classes, upload size limits, `request.FILES`, storage permissions, and persisted file name.

# Interview Questions

1. What does `FileField` store in the database? A storage-relative file name/reference.
2. Why is MIME type untrusted? The client controls it.
3. Why can storage orphan a file after DB rollback? External storage is outside DB transaction.

# Practice Exercises

1. Add authenticated upload with ownership.
2. Reject oversized and disallowed files.
3. Design private download and cleanup behavior.

# Chapter Summary

File upload is a security-sensitive pipeline from multipart parsing through validation to storage. Enforce limits, validate content, control access, and handle storage/database inconsistency.
