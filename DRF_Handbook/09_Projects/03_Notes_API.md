# Project Overview

Build a private notes API where users create, list, edit, archive, and delete their own notes. Notes demonstrate owner isolation and practical search.

# Features

- Authenticated CRUD.
- Owner-only list/detail access.
- Archive/unarchive state.
- Search title/body and stable pagination.
- Content length and field validation.

# Folder Structure

```text
notes/
  models.py
  serializers.py
  views.py
  urls.py
  tests.py
```

# API Endpoints

| Method | Route | Purpose |
|---|---|---|
| GET/POST | `/api/v1/notes/` | List/create notes |
| GET/PUT/PATCH/DELETE | `/api/v1/notes/{id}/` | Retrieve/update/delete |
| POST | `/api/v1/notes/{id}/archive/` | Archive a note |

# Database Design

Each note belongs to one user. Store title, body, archive status, and timestamps. Index owner and update time for common list queries. Hard delete versus retention should follow product policy.

# Models

```python
from django.conf import settings
from django.db import models

class Note(models.Model):
    owner = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE,
                              related_name="notes")
    title = models.CharField(max_length=160)
    body = models.TextField(max_length=20000)
    archived = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        indexes = [models.Index(fields=["owner", "archived", "-updated_at"])]
```

# Serializers

```python
from rest_framework import serializers
from .models import Note

class NoteSerializer(serializers.ModelSerializer):
    class Meta:
        model = Note
        fields = ["id", "title", "body", "archived", "created_at", "updated_at"]
        read_only_fields = ["id", "archived", "created_at", "updated_at"]

    def validate_title(self, value):
        if not value.strip():
            raise serializers.ValidationError("Title cannot be blank.")
        return value.strip()
```

# Views

```python
from rest_framework import filters, status, viewsets
from rest_framework.decorators import action
from rest_framework.permissions import IsAuthenticated
from rest_framework.response import Response
from .models import Note
from .serializers import NoteSerializer

class NoteViewSet(viewsets.ModelViewSet):
    serializer_class = NoteSerializer
    permission_classes = [IsAuthenticated]
    filter_backends = [filters.SearchFilter]
    search_fields = ["title", "body"]

    def get_queryset(self):
        return Note.objects.filter(owner=self.request.user).order_by("-updated_at", "pk")

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)

    @action(detail=True, methods=["post"])
    def archive(self, request, pk=None):
        note = self.get_object()
        note.archived = True
        note.save(update_fields=["archived", "updated_at"])
        return Response(self.get_serializer(note).data, status=status.HTTP_200_OK)
```

# URLs

```python
router = SimpleRouter()
router.register("notes", NoteViewSet, basename="note")
urlpatterns = router.urls
```

# Authentication

Configure session auth for same-site browser use or JWT for mobile clients. Never accept an `owner` field from a note create/update request.

# Permissions

`IsAuthenticated` requires login; `get_queryset()` makes all standard actions owner-scoped. The archive action calls `get_object()`, preserving that scope. Add throttling for search if note bodies are large.

# Testing

Test create assigns owner, another user cannot list/retrieve/update/delete, title validation, archive action, search scope, and pagination. Use `force_authenticate` for policy tests and a real configured authenticator test separately.

# Deployment

Install DRF, configure authentication and pagination, add app URLs under `/api/v1/`, migrate, serve only over HTTPS, and back up private note data.

# Complete Source Code

The model, serializer, ViewSet, and router above are the complete feature core. Add project imports and standard app configuration. Configure a page-number paginator globally; do not expose note content in logs or analytics.

# Line-by-Line Explanation

- Owner foreign key defines ownership and cascade policy.
- Title/body constrain record size; timestamps support ordering.
- Composite index supports owner/archive/time access.
- Serializer makes archival status and timestamps server-controlled.
- Search fields are explicit.
- Queryset scope protects list and detail endpoints.
- Custom action changes archive state through the scoped object helper.

**Flow:** Flutter/client -> auth -> owner queryset -> ViewSet -> serializer -> DB -> JSON. Archive follows the same identity and object scope before state mutation.

# Best Practices

- Keep private content out of search logs.
- Prefer archive if users expect recovery.
- Add optimistic concurrency if edits from multiple devices can conflict.
- Paginate large note collections.

# Common Mistakes

- Filtering by an owner ID supplied by client.
- Applying object permission only and forgetting list scope.
- Making archived status freely writable.

# Improvements

Add tags, pinned notes, encrypted local drafts, collaboration with explicit membership, and export functionality with background tasks.

# Challenge Version

Implement collaborative notes with owner/editor/viewer roles, revision history, conflict detection, and audit events.

# Summary

The Notes API demonstrates a compact private resource with safe ownership, search, validation, and an action endpoint. Queryset scope is the key isolation boundary.
