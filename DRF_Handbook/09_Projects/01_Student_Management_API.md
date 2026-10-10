# Project Overview

Build a school-scoped student API for listing, enrolling, viewing, and updating student records. Django handles persistence; DRF exposes a versioned JSON contract.

# Features

- Authenticated staff access.
- School-based data isolation.
- Student enrollment and read/update endpoints.
- Searchable, paginated directory.
- Validation, tests, and deployment notes.

# Folder Structure

```text
students/
  models.py
  serializers.py
  permissions.py
  views.py
  urls.py
  tests.py
```

# API Endpoints

| Method | Path | Purpose |
|---|---|---|
| GET | `/api/v1/students/` | List students in caller's school |
| POST | `/api/v1/students/` | Enroll a student |
| GET | `/api/v1/students/{id}/` | Retrieve a student |
| PATCH | `/api/v1/students/{id}/` | Update editable fields |
| DELETE | `/api/v1/students/{id}/` | Remove a record, restricted to staff |

# Database Design

`School` owns students; `Student` contains a school foreign key, admission number, names, grade, and active state. Admission number is unique within a school, enforced by a database constraint.

# Models

```python
class School(models.Model):
    name = models.CharField(max_length=160)

class Student(models.Model):
    school = models.ForeignKey(School, on_delete=models.PROTECT, related_name="students")
    admission_number = models.CharField(max_length=32)
    first_name = models.CharField(max_length=80)
    last_name = models.CharField(max_length=80)
    grade = models.CharField(max_length=20)
    active = models.BooleanField(default=True)

    class Meta:
        constraints = [
            models.UniqueConstraint(fields=["school", "admission_number"],
                                    name="unique_school_admission_number")
        ]
        ordering = ["last_name", "first_name", "pk"]
```

# Serializers

```python
from rest_framework import serializers
from .models import Student

class StudentSerializer(serializers.ModelSerializer):
    class Meta:
        model = Student
        fields = ["id", "admission_number", "first_name", "last_name", "grade", "active"]
        read_only_fields = ["id"]
```

# Views

```python
from rest_framework import filters, viewsets
from rest_framework.permissions import IsAuthenticated
from .models import Student
from .serializers import StudentSerializer

class StudentViewSet(viewsets.ModelViewSet):
    serializer_class = StudentSerializer
    permission_classes = [IsAuthenticated]
    filter_backends = [filters.SearchFilter, filters.OrderingFilter]
    search_fields = ["first_name", "last_name", "admission_number"]
    ordering_fields = ["last_name", "grade"]

    def get_queryset(self):
        return Student.objects.filter(school=self.request.user.school).order_by(
            "last_name", "first_name", "pk"
        )

    def perform_create(self, serializer):
        serializer.save(school=self.request.user.school)
```

# URLs

```python
from rest_framework.routers import SimpleRouter
from .views import StudentViewSet

router = SimpleRouter()
router.register("students", StudentViewSet, basename="student")
urlpatterns = router.urls
```

# Authentication

Configure a project-approved session or JWT authenticator. Every request must resolve a user linked to a school; never accept school ownership from the Flutter/client payload.

# Permissions

`IsAuthenticated` blocks anonymous access. The school-filtered queryset enforces tenant scope on list and detail operations. Add a staff/registrar permission for creation, deletion, and sensitive edits if students have accounts.

# Testing

Test list/detail scope for two schools, duplicate admission numbers, valid enrollment, invalid grades, anonymous access, and staff-only operations. Assert rejected writes do not modify the database.

# Deployment

Add the app to `INSTALLED_APPS`, configure `AUTH_USER_MODEL` with a school relation before initial migrations, include routes under `/api/v1/`, run migrations, enable HTTPS, and configure production database backups.

# Complete Source Code

The model, serializer, ViewSet, and router above form the app's core source. The project must provide the configured user model relation `user.school`, DRF, and a paginator. Generate migrations with `makemigrations` and apply them; migration files are generated artifacts.

# Line-by-Line Explanation

- `School` stores the institution identity.
- `Student.school` prevents orphaning a student when a school is removed.
- The uniqueness constraint protects enrollment numbers under concurrency.
- Serializer fields explicitly exclude internal school linkage.
- `get_queryset()` applies tenant scope before retrieval.
- `perform_create()` assigns the school from authenticated context.
- Search/ordering fields are allowlisted to bound API behavior.

**Request flow:** Client -> auth -> school permission/scope -> router/ViewSet -> serializer -> ORM/database -> JSON response. For create, trusted school assignment happens immediately before save.

# Best Practices

- Treat student data as sensitive personal information.
- Scope every query by school.
- Use soft deactivation when retention rules require records to remain.
- Add pagination and audit trails for access to sensitive records.

# Common Mistakes

- Trusting a submitted `school_id`.
- Using a globally unique admission number when schools issue their own.
- Returning unnecessary personal data.
- Assuming authentication alone provides school isolation.

# Improvements

Add class enrollment history, guardian contacts with restricted visibility, audit events, CSV import with transactional validation, and role-based registrar/teacher access.

# Challenge Version

Add attendance and grade resources. Teachers may access only assigned classes; administrators manage enrollment; all changes include actor and timestamp audit records.

# Summary

This project demonstrates multi-tenant CRUD with school-scoped access, database uniqueness, explicit fields, and practical search. The critical invariant is that school ownership comes from the authenticated user, not client input.
