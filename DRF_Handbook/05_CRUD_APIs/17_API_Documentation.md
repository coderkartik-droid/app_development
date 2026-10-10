# Introduction

API documentation describes available routes, methods, parameters, request bodies, response schemas, status codes, and authentication requirements. OpenAPI is a widely used machine-readable description format.

# Why this concept exists

Clients need to understand and test a contract without reading server implementation. Accurate documentation shortens integration time and reduces incompatible assumptions.

# Real-world analogy

API documentation is the instruction manual and parts catalogue for a device: it tells integrators what can be connected and what each operation returns.

# Internal Working

Schema generators inspect URL patterns, views, serializers, permissions, and annotations to produce an OpenAPI schema. A UI such as Swagger UI presents that schema interactively.

# Request Lifecycle

Schema request -> schema view/generator -> inspect routes and metadata -> OpenAPI document -> documentation UI or client generator.

# Syntax

Example OpenAPI operation:

```yaml
paths:
  /api/products/:
    get:
      summary: List products
      responses:
        "200":
          description: Paginated product results
```

# Complete Example 1

```python
# settings.py: using drf-spectacular
INSTALLED_APPS = [
    # existing Django apps...
    "drf_spectacular",
]

REST_FRAMEWORK = {
    "DEFAULT_SCHEMA_CLASS": "drf_spectacular.openapi.AutoSchema",
}

SPECTACULAR_SETTINGS = {
    "TITLE": "Catalogue API",
    "VERSION": "1.0.0",
}
```

Install `drf-spectacular` first. The app enables schema generation, DRF delegates schema inspection to its `AutoSchema`, and title/version identify the published contract. Preserve existing settings instead of replacing them. Explicit annotations are needed where inference cannot understand custom behavior.

# Complete Example 2

```python
# urls.py
from django.urls import path
from drf_spectacular.views import (
    SpectacularAPIView, SpectacularSwaggerView,
)

urlpatterns = [
    path("schema/", SpectacularAPIView.as_view(), name="schema"),
    path("docs/", SpectacularSwaggerView.as_view(url_name="schema"), name="swagger-ui"),
]
```

The schema endpoint returns OpenAPI data; the Swagger view renders an interactive UI pointing to it. Restrict or protect documentation if it reveals internal routes. Verify the schema in CI after endpoint changes.

# Complete Example 3

```python
from drf_spectacular.utils import extend_schema
from rest_framework.generics import ListAPIView

class ProductListView(ListAPIView):
    serializer_class = ProductSerializer
    queryset = Product.objects.all()

    @extend_schema(summary="List catalogue products",
                   responses=ProductSerializer(many=True))
    def get(self, request, *args, **kwargs):
        return super().get(request, *args, **kwargs)
```

The decorator overrides or supplements inferred operation metadata. If pagination is enabled, document the actual paginated envelope rather than claiming a bare array; inspect generated schema output.

# Advanced Example

Document authentication schemes, pagination, filter parameters, error responses, examples, and deprecated fields. Validate the generated schema with a schema-check command and treat breaking schema changes as release events.

# Under the Hood

Schema generation is introspection, not execution of every business path. Dynamic serializers, custom actions, and request-dependent fields may need explicit annotations to avoid inaccurate schemas.

# Performance Notes

Schema generation can inspect many routes; cache or serve a generated schema for large APIs if appropriate. Do not expose private operational detail accidentally.

# Best Practices

- Keep docs generated from code where possible.
- Review actual request and response examples.
- Document constraints, errors, auth, and pagination.
- Run schema validation in continuous integration.

# Common Mistakes

- Publishing a stale schema.
- Documenting inferred behavior without checking it.
- Omitting custom action errors or required permissions.

# Debugging

Generate the schema, inspect warnings and missing operations, compare it with endpoint tests, and resolve ambiguous serializer/action annotations.

# Interview Questions

1. What is OpenAPI? A machine-readable API contract format.
2. Why document custom actions explicitly? Introspection may not infer their semantics and payloads.
3. How can documentation stay current? Generate from code and validate in CI.

# Practice Exercises

1. Publish a schema and interactive UI for the product API.
2. Document filter, ordering, pagination, and error responses.
3. Add a schema validation step to CI.

# Chapter Summary

Good documentation is a versioned, testable contract. Generate it from DRF metadata, explicitly describe dynamic operations, and verify it matches real responses.
