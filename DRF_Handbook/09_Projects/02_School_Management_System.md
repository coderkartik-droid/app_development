# Project Overview

Create a school-management backend for schools, staff membership, classes, student enrollment, and class rosters. The design demonstrates relational constraints and tenant-specific query scope.

# Features

- Schools and authenticated memberships.
- Class creation by school administrators.
- Student enrollment into classes.
- Scoped class roster reads.
- Unique membership and enrollment constraints.

# Folder Structure

```text
school/
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
| GET/POST | `/api/v1/classes/` | List/create school classes |
| GET | `/api/v1/classes/{id}/students/` | Read a class roster |
| POST | `/api/v1/enrollments/` | Enroll student |
| DELETE | `/api/v1/enrollments/{id}/` | Remove enrollment |

# Database Design

`School` has users through `SchoolMembership`; `Classroom` belongs to a school; `Student` belongs to a school; `Enrollment` is a join entity with term and enrollment date. Unique constraints prevent duplicate membership and duplicate student/class/term enrollment.

# Models

```python
from django.conf import settings
from django.db import models

class School(models.Model):
    name = models.CharField(max_length=160)

class SchoolMembership(models.Model):
    class Role(models.TextChoices):
        ADMIN = "admin", "Admin"
        TEACHER = "teacher", "Teacher"
    school = models.ForeignKey(School, on_delete=models.CASCADE)
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    role = models.CharField(max_length=20, choices=Role.choices)

    class Meta:
        constraints = [models.UniqueConstraint(fields=["school", "user"], name="one_school_membership")]

class Classroom(models.Model):
    school = models.ForeignKey(School, on_delete=models.CASCADE, related_name="classrooms")
    name = models.CharField(max_length=80)

class Student(models.Model):
    school = models.ForeignKey(School, on_delete=models.PROTECT)
    name = models.CharField(max_length=160)

class Enrollment(models.Model):
    classroom = models.ForeignKey(Classroom, on_delete=models.CASCADE)
    student = models.ForeignKey(Student, on_delete=models.CASCADE)
    term = models.CharField(max_length=24)

    class Meta:
        constraints = [models.UniqueConstraint(
            fields=["classroom", "student", "term"], name="unique_class_student_term")]
```

# Serializers

```python
from rest_framework import serializers
from .models import Classroom, Enrollment

class ClassroomSerializer(serializers.ModelSerializer):
    class Meta:
        model = Classroom
        fields = ["id", "name"]
        read_only_fields = ["id"]

class EnrollmentSerializer(serializers.ModelSerializer):
    class Meta:
        model = Enrollment
        fields = ["id", "classroom", "student", "term"]
        read_only_fields = ["id"]

    def validate(self, attrs):
        if attrs["classroom"].school_id != attrs["student"].school_id:
            raise serializers.ValidationError("Class and student must belong to the same school.")
        return attrs
```

# Views

```python
from rest_framework import viewsets
from rest_framework.permissions import IsAuthenticated
from .models import Classroom, Enrollment, SchoolMembership
from .serializers import ClassroomSerializer, EnrollmentSerializer

class ClassroomViewSet(viewsets.ModelViewSet):
    serializer_class = ClassroomSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return Classroom.objects.filter(
            school__schoolmembership__user=self.request.user
        ).distinct()

    def perform_create(self, serializer):
        membership = SchoolMembership.objects.get(
            user=self.request.user, role=SchoolMembership.Role.ADMIN
        )
        serializer.save(school=membership.school)

class EnrollmentViewSet(viewsets.ModelViewSet):
    serializer_class = EnrollmentSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return Enrollment.objects.filter(
            classroom__school__schoolmembership__user=self.request.user
        ).distinct()
```

# URLs

```python
from rest_framework.routers import SimpleRouter
from .views import ClassroomViewSet, EnrollmentViewSet

router = SimpleRouter()
router.register("classes", ClassroomViewSet, basename="classroom")
router.register("enrollments", EnrollmentViewSet, basename="enrollment")
urlpatterns = router.urls
```

# Authentication

Use the organization's approved authentication mechanism. Each authenticated user must have explicit membership records; a user's global staff flag should not silently imply membership in every school.

# Permissions

The example scopes reads to school membership. For production, implement a role permission that restricts class creation/enrollment to school admins or authorized teachers. Ensure create input cannot select another school.

# Testing

Test cross-school class hiding, admin versus teacher writes, mismatched-school enrollment rejection, duplicate enrollment constraint, and list visibility. Verify all invalid writes leave no row.

# Deployment

Configure custom user early if needed, add app and DRF, migrate constraints, set production database and backups, include versioned router URLs, and enable HTTPS.

# Complete Source Code

The model/serializer/router/view code shown defines the core project. Add imports as grouped above and include `urlpatterns` from the app in project URLs. In production, replace the simplified role lookup with an explicit membership-scoped permission and handle missing membership with a clear denial.

# Line-by-Line Explanation

- Membership links user, school, and role; uniqueness avoids ambiguous duplicate assignments.
- Classroom is tenant-owned.
- Enrollment represents a many-to-many relationship with additional term data.
- The serializer verifies both sides of an enrollment share a school.
- Querysets traverse membership relations and use `distinct()` to remove join duplicates.
- Creation chooses school from server-side membership, never request data.

**Flow:** Client -> authentication -> school membership scope -> router/action -> serializer validation -> ORM constraints -> response. Cross-school student/class combinations fail before persistence.

# Best Practices

- Model enrollment as a first-class entity when it has attributes.
- Use database constraints for uniqueness.
- Keep role and school assignment server-controlled.
- Consider audit history for enrollment changes.

# Common Mistakes

- Modeling enrollment as an untracked direct many-to-many when term history matters.
- Checking school equality only in Flutter.
- Assuming all authenticated users belong to the same school.

# Improvements

Add teacher-to-class assignments, academic year entities, attendance, guardian access, soft withdrawal, audit events, and bulk roster import.

# Challenge Version

Build a class timetable API with conflict detection that prevents one teacher or room from being assigned to overlapping class periods in the same school.

# Summary

This system demonstrates tenant membership and relational validation. The database protects uniqueness; the API scopes every query and derives tenant identity from trusted membership.
