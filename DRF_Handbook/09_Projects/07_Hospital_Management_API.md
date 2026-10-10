# Project Overview

Build a hospital API for patients, appointments, and clinician access with strict confidentiality and audit requirements.

# Features

- Patient registration and appointment scheduling.
- Clinician access scoped to care relationship.
- Appointment status lifecycle.
- Minimal response fields and audit trail.
- Conflict checks for overlapping appointments.

# Folder Structure

```text
hospital/
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
| POST | `/api/v1/patients/` | Register patient |
| GET | `/api/v1/patients/me/` | Current patient's profile |
| GET/POST | `/api/v1/appointments/` | List/book appointment |
| POST | `/api/v1/appointments/{id}/cancel/` | Cancel eligible booking |

# Database Design

Patient is linked to an authenticated user; clinician is linked to a staff user; Appointment references patient and clinician, with start/end and status. Avoid embedding unnecessary health information in appointment records.

# Models

```python
from django.conf import settings
from django.db import models

class Patient(models.Model):
    user = models.OneToOneField(settings.AUTH_USER_MODEL, on_delete=models.PROTECT)
    display_name = models.CharField(max_length=160)

class Clinician(models.Model):
    user = models.OneToOneField(settings.AUTH_USER_MODEL, on_delete=models.PROTECT)
    active = models.BooleanField(default=True)

class Appointment(models.Model):
    patient = models.ForeignKey(Patient, on_delete=models.PROTECT)
    clinician = models.ForeignKey(Clinician, on_delete=models.PROTECT)
    starts_at = models.DateTimeField()
    ends_at = models.DateTimeField()
    status = models.CharField(max_length=20, default="booked")
```

# Serializers

```python
class AppointmentCreateSerializer(serializers.ModelSerializer):
    class Meta:
        model = Appointment
        fields = ["id", "clinician", "starts_at", "ends_at"]
        read_only_fields = ["id"]

    def validate(self, attrs):
        if attrs["ends_at"] <= attrs["starts_at"]:
            raise serializers.ValidationError("End must be after start.")
        return attrs
```

# Views

```python
class AppointmentViewSet(ModelViewSet):
    permission_classes = [IsAuthenticated]
    serializer_class = AppointmentCreateSerializer

    def get_queryset(self):
        user = self.request.user
        if user.is_staff:
            return Appointment.objects.filter(clinician__user=user)
        return Appointment.objects.filter(patient__user=user)

    def perform_create(self, serializer):
        patient = Patient.objects.get(user=self.request.user)
        serializer.save(patient=patient, status="booked")
```

# URLs

```python
router = SimpleRouter()
router.register("appointments", AppointmentViewSet, basename="appointment")
urlpatterns = router.urls
```

# Authentication

Require a strong, approved identity mechanism and consider MFA for clinician/staff access. Do not use patient-supplied user IDs to select identity.

# Permissions

Patients see their own appointments; clinicians see appointments assigned to them. Staff access must be purpose-limited and audited. `is_staff` alone may be too broad; use explicit clinical roles and care relationships.

# Testing

Test patient isolation, clinician assignment, appointment time validation, scheduling conflicts under concurrent requests, cancellation policy, and audit events. Use synthetic health data only.

# Deployment

Meet applicable privacy and health-data obligations, use encrypted transport/storage, least-privilege infrastructure, strict backups/retention, access auditing, and incident response.

# Complete Source Code

The snippets define core fields and booking behavior. Production scheduling must use a transaction and database-supported conflict prevention appropriate to the database. A simplified overlap query:

```python
def ensure_available(clinician, start, end):
    conflict = Appointment.objects.filter(
        clinician=clinician, status="booked",
        starts_at__lt=end, ends_at__gt=start,
    ).exists()
    if conflict:
        raise serializers.ValidationError("Clinician is not available.")
```

# Line-by-Line Explanation

- Separate Patient and Clinician roles from generic user identity.
- Appointment links participants without storing unnecessary clinical notes.
- Serializer checks chronological validity.
- `get_queryset()` scopes based on role and relationship.
- `perform_create()` derives patient from authenticated user.
- Availability query checks interval overlap; transaction/constraint strategy must protect against concurrent bookings.

**Flow:** Client -> authentication/MFA -> patient/clinician permission -> scoped appointment queryset -> validation/service -> database -> minimal response and audit.

# Best Practices

- Minimize sensitive data and response fields.
- Audit every privileged access and change.
- Enforce care relationship and tenant scope.
- Define retention and breach response.

# Common Mistakes

- Returning patient details to any authenticated clinician.
- Using `is_staff` as the sole clinical policy.
- Assuming an `.exists()` check prevents concurrent double-booking.

# Improvements

Add consent records, clinician schedules, appointment reminders via background tasks, audit export, and emergency-access workflow with enhanced logging.

# Challenge Version

Implement a multi-hospital tenancy model with clinician affiliations, patient consent, conflict-safe scheduling, and auditable emergency access.

# Summary

Hospital APIs handle highly sensitive data and need stricter controls than ordinary CRUD. Least privilege, relationship-based scope, auditability, and concurrency-safe scheduling are foundational.
