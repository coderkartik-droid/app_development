# Project Overview

Build a blog API with public published posts, author-owned drafts, comments, search, and moderation.

# Features

- Public published post feed.
- Author-only draft management.
- Publishing action with validation.
- Comments and moderation status.
- Search and pagination.

# Folder Structure

```text
blog/
  models.py
  serializers.py
  permissions.py
  views.py
  urls.py
  tests.py
```

# API Endpoints

| Method | Route | Purpose |
|---|---|---|
| GET | `/api/v1/posts/` | Public published posts |
| POST | `/api/v1/posts/` | Create draft |
| GET/PATCH/DELETE | `/api/v1/posts/{id}/` | Author management |
| POST | `/api/v1/posts/{id}/publish/` | Publish own post |
| GET/POST | `/api/v1/posts/{id}/comments/` | Read/add comments |

# Database Design

Post belongs to an author and stores title, slug, body, status, and timestamps. Comment belongs to post and author, with moderation state. Slugs are unique; only published content is public.

# Models

```python
from django.conf import settings
from django.db import models

class Post(models.Model):
    class Status(models.TextChoices):
        DRAFT = "draft", "Draft"
        PUBLISHED = "published", "Published"
    author = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    title = models.CharField(max_length=180)
    slug = models.SlugField(unique=True)
    body = models.TextField()
    status = models.CharField(max_length=12, choices=Status.choices, default=Status.DRAFT)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

class Comment(models.Model):
    post = models.ForeignKey(Post, on_delete=models.CASCADE, related_name="comments")
    author = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    body = models.TextField(max_length=3000)
    approved = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)
```

# Serializers

```python
class PostSerializer(serializers.ModelSerializer):
    class Meta:
        model = Post
        fields = ["id", "author", "title", "slug", "body", "status", "created_at"]
        read_only_fields = ["id", "author", "status", "created_at"]
```

# Views

```python
from rest_framework import filters, viewsets
from rest_framework.decorators import action
from rest_framework.permissions import AllowAny, IsAuthenticated
from rest_framework.response import Response

class PostViewSet(viewsets.ModelViewSet):
    serializer_class = PostSerializer
    filter_backends = [filters.SearchFilter]
    search_fields = ["title", "body"]

    def get_permissions(self):
        if self.action in {"list", "retrieve"}:
            return [AllowAny()]
        return [IsAuthenticated()]

    def get_queryset(self):
        qs = Post.objects.select_related("author").order_by("-created_at", "pk")
        if self.request.user.is_authenticated:
            return qs.filter(status=Post.Status.PUBLISHED) | qs.filter(author=self.request.user)
        return qs.filter(status=Post.Status.PUBLISHED)

    def perform_create(self, serializer):
        serializer.save(author=self.request.user)

    @action(detail=True, methods=["post"])
    def publish(self, request, pk=None):
        post = self.get_object()
        post.status = Post.Status.PUBLISHED
        post.save(update_fields=["status", "updated_at"])
        return Response(self.get_serializer(post).data)
```

# URLs

```python
router = SimpleRouter()
router.register("posts", PostViewSet, basename="post")
urlpatterns = router.urls
```

# Authentication

Public published reads allow anonymous users; draft creation/editing requires authentication. Use the project's session/JWT mechanism and secure author assignment.

# Permissions

The example queryset combines published posts and the current author's posts. Add an object permission to prevent editing another author's published content; as written, authentication alone does not enforce that write rule. Ensure the `publish` action is owner/staff-only.

# Testing

Test anonymous published feed, drafts hidden from strangers, author draft access, non-author update denial, publish authorization, comment moderation, search, and pagination.

# Deployment

Use HTTPS, configure database and media storage, enable pagination/throttling, add schema docs, and deploy migrations with backups. Cache only public published representations.

# Complete Source Code

The model, serializer, ViewSet, and router form a project skeleton. Production must add `IsAuthorOrReadOnly`/staff policy, comment serializers/views, pagination, and a safe slug generation strategy. Do not deploy the simplified write permission as-is.

# Line-by-Line Explanation

- Status choices distinguish draft from published.
- Author and status are read-only in serializer input.
- Permission selection makes listing public and writes authenticated.
- Queryset includes public posts plus the caller's own drafts.
- `perform_create` assigns author.
- Publish action changes domain state only after object retrieval.

**Flow:** Flutter/browser -> auth if required -> visibility queryset -> serializer/action -> database -> rendered post JSON. A public list never includes another author's draft.

# Best Practices

- Keep draft visibility private.
- Sanitize rendered rich text at the presentation boundary.
- Moderate comments and rate-limit posting.
- Use slug uniqueness and stable pagination.

# Common Mistakes

- Making every authenticated user able to edit every post.
- Exposing drafts through an overly broad queryset.
- Rendering untrusted HTML without sanitization.

# Improvements

Add tags, scheduled publishing, revision history, moderation queues, author profiles, and cache invalidation after publish.

# Challenge Version

Build editorial workflow with author, editor, and publisher roles, approval history, scheduled releases, and immutable audit records.

# Summary

The blog project illustrates public/private visibility, draft ownership, and publishing workflows. Queryset visibility and write permissions must be designed separately and tested by role.
