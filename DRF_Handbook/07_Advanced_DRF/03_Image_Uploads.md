# Introduction

Image uploads are file uploads with additional content validation and image-processing requirements, such as dimensions, orientation, and safe derivatives.

# Why this concept exists

Avatars and product images need predictable visual content and must not allow arbitrary dangerous or resource-exhausting files.

# Real-world analogy

An image upload is like a photo ID submission: confirm it is actually an image, meets size rules, and is stored where only appropriate people can view it.

# Internal Working

The request parser yields an uploaded file. An image field validates that the data can be decoded as an image (Pillow is commonly required); the storage backend saves the content. Image decoding itself can consume significant CPU and memory.

# Authentication Flow (where applicable)

Client -> authentication -> owner permission -> multipart parser -> image validation -> storage/model -> response containing controlled metadata.

# Syntax

```python
image = serializers.ImageField()
```

# Complete Example 1

```python
# models.py
from django.conf import settings
from django.db import models

class Avatar(models.Model):
    owner = models.OneToOneField(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    image = models.ImageField(upload_to="avatars/")
```

`ImageField` stores a path/reference via Django storage and validates image-like uploads when processed through Django forms/serializers. Install Pillow and apply migrations. Configure private/public access intentionally; `upload_to` is not an access policy.

# Complete Example 2

```python
from rest_framework import generics, serializers
from rest_framework.parsers import MultiPartParser, FormParser
from rest_framework.permissions import IsAuthenticated
from .models import Avatar

class AvatarSerializer(serializers.ModelSerializer):
    class Meta:
        model = Avatar
        fields = ["image"]

class AvatarUploadView(generics.CreateAPIView):
    serializer_class = AvatarSerializer
    parser_classes = [MultiPartParser, FormParser]
    permission_classes = [IsAuthenticated]

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)
```

The parser accepts multipart uploads; ImageField checks decodability; the view requires identity and sets owner. A one-to-one record may already exist, so production should implement update-or-create behavior and size/dimension constraints. Client MIME headers alone are insufficient.

# Complete Example 3

```python
response = APIClient().post(
    "/api/avatar/",
    {"image": SimpleUploadedFile("avatar.png", valid_png_bytes,
                                 content_type="image/png")},
    format="multipart",
)
```

`valid_png_bytes` must be actual valid image bytes; a filename and MIME label do not create a valid image. Assert accepted response and stored ownership. Test corrupt images and decompression-bomb limits.

# Advanced Example

Generate normalized derivatives (size, orientation, format) asynchronously after quarantine/scanning. Strip metadata such as GPS coordinates when privacy requires it, and serve through controlled CDN/storage policy.

# Under the Hood

Image validation decodes image headers/content; processing may allocate memory proportional to dimensions. Thumbnail generation and transcoding are CPU-intensive and belong in bounded workers for large workloads.

# Performance Notes

Limit byte size and pixel dimensions, process asynchronously, generate multiple sizes once, and use CDN caching only for content whose visibility permits it.

# Security Notes

Defend against polyglot files, decompression bombs, malicious metadata, path traversal, and public exposure. Keep libraries patched.

# Best Practices

- Validate actual image data and dimensions.
- Normalize generated filenames.
- Store originals privately when sensitive.
- Test corrupted and extreme-dimension images.

# Common Mistakes

- Trusting extension or MIME type.
- Returning a public URL for a private avatar.
- Doing large image transformations inside a request.

# Debugging

Check Pillow installation, multipart parsing, image decoder errors, file limits, storage configuration, and dimensions.

# Interview Questions

1. What extra validation does ImageField provide?
2. Why can a valid image still be dangerous? Extreme dimensions or malformed content can consume resources.
3. Why strip image metadata?

# Practice Exercises

1. Build authenticated avatar upload/update.
2. Add dimension and byte-size rules.
3. Design asynchronous thumbnail creation and failure cleanup.

# Chapter Summary

Image handling adds decoding and processing risk to ordinary file uploads. Validate actual content, bound resource use, control privacy, and move expensive transformations out of request workers.
