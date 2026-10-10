# Project Overview

Design an attendance API that records a student's presence for a class session. This project focuses on trustworthy attendance events and privacy; it does not implement biometric face recognition.

# Features

- Class session and attendance records.
- Teacher-scoped recording.
- Duplicate prevention per student/session.
- Correction audit trail.
- Attendance reports by date/class.

# Folder Structure

```text
attendance/
  models.py
  serializers.py
  permissions.py
  services.py
  views.py
  urls.py
  tests.py
```

# API Endpoints

| Method | Route | Purpose |
|---|---|---|
| GET/POST | `/api/v1/sessions/` | List/create class sessions |
| POST | `/api/v1/sessions/{id}/attendance/` | Record attendance batch |
| GET | `/api/v1/sessions/{id}/attendance/` | Read roster results |
| PATCH | `/api/v1/attendance/{id}/` | Correct a mark with audit |

# Database Design

`ClassSession` belongs to a classroom and has a start time. `AttendanceRecord` references session and student, records mark, actor, and timestamp. A unique session/student constraint prevents duplicate records.

# Models

```python
from django.conf import settings
from django.db import models

class ClassSession(models.Model):
    classroom = models.ForeignKey("school.Classroom", on_delete=models.CASCADE)
    starts_at = models.DateTimeField()

class AttendanceRecord(models.Model):
    class Mark(models.TextChoices):
        PRESENT = "present", "Present"
        ABSENT = "absent", "Absent"
        LATE = "late", "Late"
    session = models.ForeignKey(ClassSession, on_delete=models.CASCADE)
    student = models.ForeignKey("school.Student", on_delete=models.PROTECT)
    mark = models.CharField(max_length=12, choices=Mark.choices)
    recorded_by = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.PROTECT)
    recorded_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        constraints = [models.UniqueConstraint(
            fields=["session", "student"], name="one_attendance_per_session_student")]
```

# Serializers

```python
class AttendanceInputSerializer(serializers.Serializer):
    student_id = serializers.IntegerField(min_value=1)
    mark = serializers.ChoiceField(choices=AttendanceRecord.Mark.choices)

class AttendanceRecordSerializer(serializers.ModelSerializer):
    class Meta:
        model = AttendanceRecord
        fields = ["id", "student", "mark", "recorded_by", "recorded_at"]
        read_only_fields = fields
```

# Views

```python
from django.db import transaction
from rest_framework import serializers
from rest_framework.permissions import IsAuthenticated
from rest_framework.views import APIView
from rest_framework.response import Response

class SessionAttendanceView(APIView):
    permission_classes = [IsAuthenticated, IsAssignedTeacher]

    def post(self, request, session_id):
        session = get_object_or_404(
            ClassSession.objects.filter(classroom__teachers=request.user),
            pk=session_id,
        )
        serializer = AttendanceInputSerializer(data=request.data, many=True)
        serializer.is_valid(raise_exception=True)
        with transaction.atomic():
            records = record_attendance(session, serializer.validated_data, request.user)
        return Response(AttendanceRecordSerializer(records, many=True).data, status=201)
```

# URLs

```python
urlpatterns = [
    path("sessions/<int:session_id>/attendance/",
         SessionAttendanceView.as_view(), name="session-attendance"),
]
```

# Authentication

Require authenticated staff/teacher identity. If face matching exists in a separate system, the API should accept a verified event or reference from that system, not raw biometric templates without a clear legal/security basis.

# Permissions

`IsAssignedTeacher` should verify the caller teaches the session's classroom. The queryset lookup also scopes the session. Check every student belongs to the same classroom/school before saving.

# Testing

Test assigned teacher success, unrelated teacher denial, foreign-school student rejection, duplicate records, invalid mark, atomic batch rollback, and audit actor/time.

# Deployment

Use strict access logging without biometric data, encrypt sensitive data in transit/at rest, apply retention policy, back up the database, and monitor attendance writes.

# Complete Source Code

The snippets define the schema and endpoint boundary. A domain service should enforce membership and duplicates:

```python
def record_attendance(session, rows, actor):
    student_ids = [row["student_id"] for row in rows]
    students = Student.objects.filter(
        pk__in=student_ids, school=session.classroom.school
    )
    by_id = {student.pk: student for student in students}
    if len(by_id) != len(set(student_ids)):
        raise serializers.ValidationError("Every student must belong to this school.")
    return [
        AttendanceRecord.objects.create(
            session=session, student=by_id[row["student_id"]],
            mark=row["mark"], recorded_by=actor
        )
        for row in rows
    ]
```

# Line-by-Line Explanation

- The model defines controlled attendance states.
- Foreign keys tie records to a class session and student.
- Unique constraint blocks duplicate marks even under concurrent requests.
- Input serializer accepts student IDs and allowed marks only.
- Endpoint scopes session to assigned teacher before processing.
- Service checks school membership and creates actor-attributed rows.
- Atomic transaction prevents partial batch persistence.

**Flow:** Client -> authentication -> teacher permission -> session scope -> input validation -> membership check -> transaction/DB -> response. Any invalid row aborts the batch.

# Best Practices

- Minimize personal/biometric data.
- Make corrections auditable rather than silently overwriting history.
- Enforce unique and school-membership constraints.
- Define batch atomicity and size limit.

# Common Mistakes

- Trusting face-match claims directly from client.
- Recording any student ID without school/class validation.
- Storing biometric templates without strict necessity and governance.

# Improvements

Add attendance correction events, parent notifications, reports, time-zone-aware sessions, and signed integration events from a verified device service.

# Challenge Version

Add offline attendance capture with signed device identity, idempotency keys, conflict resolution, and a tamper-evident audit trail.

# Summary

Attendance is a sensitive domain. This project models trustworthy teacher-attributed records and explicitly avoids treating biometric recognition as a simple CRUD field.
