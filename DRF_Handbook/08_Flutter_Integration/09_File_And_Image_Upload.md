# Introduction

Flutter can upload files and images to DRF using multipart form data. Both client and server must handle file selection, encoding, limits, validation, and storage policy.

# Why this concept exists

Users need to attach receipts, profile photos, and documents to server-side records.

# Real-world analogy

The client packages a document as a labeled parcel; the server checks the label and contents before storing it in a secure facility.

# Internal Working

Flutter's multipart request creates boundaries and file parts. DRF `MultiPartParser` parses fields/files; a serializer validates; Django storage writes bytes and the database stores a reference.

# Data Flow

Flutter picker -> multipart request -> auth/permission -> DRF parser/serializer
-> storage/database -> JSON metadata response -> Flutter UI.

# Complete Example 1

Flutter upload:

```dart
final request = http.MultipartRequest(
  'POST',
  Uri.https('api.example.com', '/api/attachments/'),
);
request.headers['Authorization'] = 'Bearer $accessToken';
request.files.add(await http.MultipartFile.fromPath('file', pickedFile.path));
final streamed = await request.send().timeout(const Duration(seconds: 60));
final response = await http.Response.fromStream(streamed);
if (response.statusCode != 201) throw ApiException.fromResponse(response);
```

The request automatically sets multipart boundary; auth header identifies the caller; file bytes stream from selected path; timeout bounds waiting; response is collected and checked. For large files, report progress and use resumable/direct-storage upload patterns. Avoid logging paths or tokens.

# Complete Example 2

DRF endpoint:

```python
class AttachmentUploadView(CreateAPIView):
    serializer_class = AttachmentSerializer
    parser_classes = [MultiPartParser, FormParser]
    permission_classes = [IsAuthenticated]

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)
```

The parser handles multipart; authentication and permission gate access; ownership is server assigned. Serializer should validate actual type and size, and storage should be private for sensitive files. Add proxy-level size limits.

# Complete Example 3

DRF model and serializer:

```python
class Attachment(models.Model):
    owner = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    file = models.FileField(upload_to="private-uploads/")

class AttachmentSerializer(serializers.ModelSerializer):
    class Meta:
        model = Attachment
        fields = ["id", "file"]
        read_only_fields = ["id"]
```

The file field stores a storage key, not necessarily file bytes in the database. The upload path is not authorization. Use controlled names, private storage, and a permission-checked download endpoint.

# Advanced Example

For high-volume images, upload to quarantine, scan/normalize asynchronously, strip metadata if needed, and publish only approved derivatives. Direct signed uploads should be short-lived and bound to an authenticated user and expected object key.

# Under the Hood

Small Django uploads may stay in memory; larger ones use temporary files based on upload handlers. Storage and DB transactions are separate, so failures can leave orphan objects.

# Performance Notes

Enforce byte and dimension limits at edge and app layers. Use streaming/resumable methods for large files and background processing for image transformations.

# Security Notes

Client filenames, extensions, MIME types, and image claims are untrusted. Prevent path traversal, malware, decompression bombs, public exposure, and cross-user downloads.

# Best Practices

- Use HTTPS and authenticated ownership.
- Validate content, size, count, and dimensions.
- Keep private uploads private and authorize downloads.
- Define orphan cleanup and retention.

# Common Mistakes

- Manually setting multipart boundary header.
- Trusting filename extension.
- Serving uploaded content directly from a public directory.

# Debugging

Check picker permission/path, multipart field name, parser, request size, serializer errors, storage write, and returned URL access policy.

# Interview Questions

1. Why use multipart rather than JSON for file bytes?
2. What does DRF MultiPartParser do?
3. Why is storage not transactionally tied to database?

# Practice Exercises

1. Upload an authenticated profile image with progress.
2. Reject corrupt or oversized content.
3. Implement owner-scoped download and cleanup.

# Chapter Summary

Uploads cross client, parser, validation, storage, and access-control boundaries. Validate actual content, bound resource use, and keep private files protected.
