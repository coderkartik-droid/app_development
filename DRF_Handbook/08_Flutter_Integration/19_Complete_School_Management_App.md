# Introduction

This chapter designs a school-management Flutter app backed by DRF for authenticated student and class workflows.

# Why this concept exists

It demonstrates a realistic integration in which multiple roles access related, sensitive data through a mobile client.

# Real-world analogy

The Flutter app is a school office's mobile portal; DRF is the central records system that verifies staff identity and school membership before sharing information.

# Internal Working

Flutter login obtains tokens; API calls carry access token; DRF resolves user and school/role, scopes records, validates writes, persists through ORM, and returns paginated JSON.

# Data Flow

```text
Flutter login -> DRF token endpoint -> access/refresh
Flutter student screen -> bearer GET -> auth -> school permission/queryset
-> serializer -> DB -> paginated JSON -> model mapping -> UI
```

# Complete Example 1

Flutter model and GET method:

```dart
class Student {
  const Student({required this.id, required this.name, required this.grade});
  final int id;
  final String name;
  final String grade;

  factory Student.fromJson(Map<String, dynamic> json) => Student(
        id: json['id'] as int,
        name: json['name'] as String,
        grade: json['grade'] as String,
      );
}

Future<List<Student>> fetchStudents(http.Client client, String access) async {
  final response = await client.get(
    Uri.https('api.example.com', '/api/v1/students/'),
    headers: {'Authorization': 'Bearer $access', 'Accept': 'application/json'},
  );
  if (response.statusCode != 200) throw ApiException.fromResponse(response);
  final page = jsonDecode(response.body) as Map<String, dynamic>;
  return (page['results'] as List)
      .map((row) => Student.fromJson(row as Map<String, dynamic>))
      .toList();
}
```

The model maps three documented fields; the request uses HTTPS and bearer access; status is checked before page parsing; `results` maps to typed students. Add timeout, pagination metadata, and safe token provider rather than passing token through UI layers.

# Complete Example 2

DRF models/serializer:

```python
class Student(models.Model):
    school = models.ForeignKey(School, on_delete=models.CASCADE)
    name = models.CharField(max_length=120)
    grade = models.CharField(max_length=20)

class StudentSerializer(serializers.ModelSerializer):
    class Meta:
        model = Student
        fields = ["id", "name", "grade"]
        read_only_fields = ["id"]
```

The model ties each student to a school; serializer exposes only app-required fields and keeps ID read-only. In production, evaluate personal-data minimization and relationships/retention rules. Apply migrations before endpoint use.

# Complete Example 3

DRF endpoint and route:

```python
class StudentListView(ListAPIView):
    serializer_class = StudentSerializer
    permission_classes = [IsAuthenticated]
    pagination_class = PageNumberPagination

    def get_queryset(self):
        return Student.objects.filter(school=self.request.user.school).order_by("name", "pk")

urlpatterns = [
    path("students/", StudentListView.as_view(), name="student-list"),
]
```

The view requires an authenticated user and scopes records to their school. This assumes each user has a trusted school relation. The route is included under `/api/v1/`; test cross-school denial and configure page size.

# Advanced Example

Add roles: teachers may view assigned classes and submit attendance; administrators may manage enrollment; students may view their own profile. Use server-side policy, audit sensitive access, minimize student data, and use transactions for attendance batches.

# Under the Hood

Flutter model mapping runs after JSON decode. DRF authentication sets the user; permission and queryset scope enforce school access; pagination evaluates bounded query; serializer renders response.

# Performance Notes

Paginate student lists, index school/grade fields, select related class data, debounce search, and cache only non-sensitive public data.

# Security Notes

Student records are sensitive. Use HTTPS, short-lived tokens, secure refresh storage, strict school scope, minimal fields, audit logs, and privacy-aware retention.

# Best Practices

- Separate Flutter auth/service/repository/state layers.
- Keep role assignment server-side.
- Test school isolation and each role.
- Version and document response contracts.

# Common Mistakes

- Sending school ID from client and trusting it.
- Returning all student fields.
- Using one global permission for every role/action.

# Debugging

Trace token/user, school relation, queryset, serializer shape, pagination envelope, and Flutter parser fields.

# Interview Questions

1. How is tenant isolation enforced?
2. Why should Flutter not select the school scope?
3. What additional safeguards apply to student data?

# Practice Exercises

1. Add teacher-assigned class scoping.
2. Implement attendance creation with atomic batch semantics.
3. Add Flutter search/pagination and secure logout.

# Chapter Summary

The school app demonstrates secure cross-platform integration: Flutter handles user experience and transport; DRF validates identity, roles, school scope, and data access for every request.
