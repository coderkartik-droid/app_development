# Introduction

Multipart form data is an HTTP encoding that carries fields and binary files in one request, commonly used for uploads.

# Why this concept exists

JSON is suitable for structured text but is inefficient or awkward for transmitting binary files together with form fields.

# Real-world analogy

Multipart is a parcel with labeled compartments: each field or file has its own boundary and content headers.

# Internal Working

The request declares `multipart/form-data` with a boundary. Django/DRF parsers split parts into `request.data` and uploaded file objects in `request.FILES`.

# Authentication Flow (where applicable)

Client sends multipart request -> authentication/policy -> MultiPartParser -> serializer validation -> storage/database -> response.

# Syntax

```python
parser_classes = [MultiPartParser, FormParser]
```

# Complete Example 1

```python
from rest_framework.parsers import MultiPartParser, FormParser
from rest_framework.generics import CreateAPIView

class UploadView(CreateAPIView):
    parser_classes = [MultiPartParser, FormParser]
    serializer_class = UploadSerializer
```

`MultiPartParser` handles multipart boundaries; `FormParser` supports URL-encoded form fields. DRF selects a parser based on request content type when accessing the body. Do not manually set a fixed multipart Content-Type in browser clients because the browser must add the boundary.

# Complete Example 2

```python
class UploadSerializer(serializers.Serializer):
    title = serializers.CharField(max_length=120)
    file = serializers.FileField()
```

The serializer validates text and uploaded file together. The view must use the parser and apply authentication, size/type constraints, and persistence. Use `request.data`, not `request.body`, for parsed input.

# Complete Example 3

```python
response = APIClient().post(
    "/api/uploads/",
    {"title": "Invoice", "file": SimpleUploadedFile("invoice.txt", b"demo")},
    format="multipart",
)
assert response.status_code == 201
```

`format="multipart"` builds the request with a valid boundary in DRF tests. Assert uploaded content and ownership as well as status. Also cover malformed body and size limits.

# Advanced Example

For forms containing nested lists, HTML multipart has naming conventions and limitations. Define a clear field representation and test actual browser/client encoding; for large files consider direct storage uploads.

# Under the Hood

The parser consumes the request stream into form values and uploaded-file objects. Upload handlers determine memory versus temporary-disk behavior; proxy and Django limits should align.

# Performance Notes

Large bodies consume network, disk, and worker resources. Set limits at reverse proxy and application layers and use streaming/direct uploads when appropriate.

# Security Notes

Authenticate uploads, cap field counts and file sizes, validate content, generate names, and prevent temporary-file exhaustion.

# Best Practices

- Let clients generate multipart boundaries correctly.
- Declare parser support explicitly when useful.
- Test with the same encoding as production clients.
- Separate metadata validation from file content policy.

# Common Mistakes

- Sending JSON while expecting an uploaded file.
- Manually setting an incorrect multipart boundary.
- Reading the whole request body into memory.

# Debugging

Inspect `Content-Type`, boundary, parser classes, `request.data`, `request.FILES`, and proxy limits.

# Interview Questions

1. What does MultiPartParser produce?
2. Why should browser code not manually set multipart Content-Type?
3. Where can upload size limits be enforced?

# Practice Exercises

1. Upload one file plus metadata.
2. Test URL-encoded versus multipart parsers.
3. Configure and verify file-size limits.

# Chapter Summary

Multipart carries fields and files through a boundary-based encoding. Correct parser selection, client boundary handling, resource limits, and validation are essential for reliable uploads.
