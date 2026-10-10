# Introduction

These exercises reinforce the end-to-end skills needed to connect Flutter screens to DRF services.

# Why this concept exists

Integration knowledge is practical: URL, parser, auth, response shape, and UI state must all agree.

# Real-world analogy

Practice is a rehearsal of the entire delivery chain, not just writing the address on the parcel.

# Internal Working

For each exercise, trace Flutter state -> repository -> request -> DRF policy/serializer -> database -> response -> mapping/UI and test failure paths.

# Data Flow

```text
Flutter -> HTTP -> DRF auth/permission -> serializer/DB -> response -> UI
```

# Complete Example 1

Flutter model:

```dart
class Student {
  const Student({required this.id, required this.name});
  final int id;
  final String name;

  factory Student.fromJson(Map<String, dynamic> json) =>
      Student(id: json['id'] as int, name: json['name'] as String);
}
```

This defines a strict contract mapping. Exercise: implement a DRF serializer with matching `id` and `name`, then write a Flutter parsing test using representative JSON and a backend API test for the same fields.

# Complete Example 2

DRF list:

```python
class StudentListView(ListAPIView):
    serializer_class = StudentSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return Student.objects.filter(school=self.request.user.school).order_by("name", "pk")
```

The endpoint requires identity and restricts results to the caller's school. Exercise: configure the Flutter bearer header, parse pagination, and test that a student from another school is absent.

# Complete Example 3

Flutter request:

```dart
final response = await client.post(
  Uri.https('api.example.com', '/api/students/'),
  headers: {'Authorization': 'Bearer $access', 'Content-Type': 'application/json'},
  body: jsonEncode({'name': name}),
);
```

This sends JSON with credentials. Exercise: complete status/error mapping, implement a matching create serializer, assign school server-side, and verify invalid input does not create a row.

# Advanced Example

Build a complete school-management flow: JWT login, student list/search/pagination, create/update, document upload, offline read cache, role-specific teacher/admin policies, and contract tests.

# Under the Hood

Test client mapping independently, DRF endpoint behavior through APIClient, authentication through real headers, and full device networking on emulator/device.

# Performance Notes

Include pagination and debounce in search exercises; record request/query counts.

# Security Notes

Use synthetic data and credentials. Test school isolation and never expose student personal information unnecessarily.

# Best Practices

- Keep exercises reproducible.
- Test one boundary at a time and then end to end.
- Include negative cases.

# Common Mistakes

- Testing Flutter parsing against hand-written JSON that differs from DRF.
- Using `force_authenticate` as the only auth test.
- Omitting tenant access assertions.

# Debugging

Compare exact serialized DRF response with Flutter model expectations; inspect auth, status, and error envelope.

# Interview Questions

1. Why test the same contract at client and server?
2. How can school scoping be enforced?
3. Why debounce search?

# Practice Exercises

1. Implement paginated student search.
2. Add teacher/admin permissions.
3. Upload and authorize a student document.

# Chapter Summary

End-to-end practice should verify contract, identity, authorization, persistence, UI state, and failure behavior together.
