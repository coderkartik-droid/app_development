# Introduction

This chapter reviews DRF view families, routers, hooks, lifecycle, and optimization for interviews and design discussions.

# Why this concept exists

Choosing a view abstraction is a design decision. Strong answers explain what behavior is reused, what policy remains explicit, and how requests reach actions.

# Real-world analogy

Selecting a view class is like choosing a service workflow: custom desk, standard counter, or a full resource department with a generated directory.

# Internal Working

```text
route -> FBV/APIView/generic/ViewSet dispatch
      -> policy -> handler/mixin/action -> serializer/query -> response
```

# Request Lifecycle

Routers map HTTP methods to actions for ViewSets; APIView maps methods to handlers. DRF initializes the request, checks policies, executes logic, catches API exceptions, and finalizes the response.

# Syntax

```python
router.register("items", ItemViewSet, basename="item")
```

# Complete Example 1

Interview prompt: “How do you safely create a user-owned object with a generic view?”

```python
class ItemCreateView(generics.CreateAPIView):
    serializer_class = ItemSerializer
    permission_classes = [IsAuthenticated]

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)
```

**Line-by-line:** Generic create view provides POST flow; serializer validates accepted fields; authenticated permission identifies caller; `perform_create` injects owner from server context. **Flow:** router -> view -> permission -> parser/serializer -> owner assignment -> ORM -> response. **Internal:** Save hook runs after validation. **Professional tip:** For reads and updates, also scope queryset by owner.

# Complete Example 2

Interview prompt: “How expose only list and detail?”

```python
class ItemReadOnlyViewSet(viewsets.ReadOnlyModelViewSet):
    queryset = Item.objects.filter(is_public=True)
    serializer_class = ItemSerializer
```

Router provides list/retrieve only. Queryset filters public items; read-only class does not itself make access public. State the permission policy separately.

# Complete Example 3

Interview prompt: “How reduce nested serializer queries?”

```python
def get_queryset(self):
    return Parent.objects.prefetch_related("children").select_related("owner")
```

Use query loading that matches relation cardinality and serializer access. Verify with query-count tests.

# Advanced Example

Be ready to explain:

- FBV: concise endpoint with `@api_view`.
- APIView: explicit class handlers and policies.
- GenericAPIView + mixins: custom composition of common CRUD.
- Concrete generic views: standard operation combinations.
- ViewSet: actions grouped by resource.
- Router: URLs/action maps, not authorization.

# Under the Hood

`APIView.as_view` produces Django-callable dispatch. Routers configure ViewSet action maps. Generic mixins call helper methods such as `get_queryset`, `get_object`, `get_serializer`, and save hooks.

# Performance Notes

Mention pagination, relation loading, serializer cost, and measured query counts.

# Best Practices

- Choose minimal abstraction that matches behavior.
- Scope querysets and set permissions explicitly.
- Use hooks instead of duplicating framework handlers.
- Test route/action map and negative access.

# Common Mistakes

- Claiming ModelViewSet is always best.
- Saying router secures endpoints.
- Confusing `perform_create` with serializer validation.
- Ignoring 405 and object-level behavior.

# Debugging

Trace route/action map, method, policy, queryset, serializer, save hook, and response. Identify the first stage not reached.

# Interview Questions

1. APIView vs GenericAPIView? API dispatch foundation vs additional generic resource helpers.
2. What does ListCreateAPIView combine? GenericAPIView, list mixin, and create mixin.
3. Why `get_queryset`? Request-dependent and secure filtering.
4. What does router do? Generates routes and method/action mappings.
5. When use `@action`? Resource-specific operation beyond standard CRUD.
6. How restrict ModelViewSet? Compose GenericViewSet with selected mixins.
7. Authentication vs permissions? Identity vs authorization decision.
8. How avoid N+1? `select_related`/`prefetch_related` aligned with serializer.

# Practice Exercises

1. Explain when to choose each view family.
2. Draw a ViewSet/router create lifecycle.
3. Diagnose a cross-user data leak and a 405 error.

# Chapter Summary

Interview-ready view design means explaining dispatch, router mapping, hooks, policy, queryset scope, and serializer/data flow. No abstraction replaces explicit authorization and tests.
