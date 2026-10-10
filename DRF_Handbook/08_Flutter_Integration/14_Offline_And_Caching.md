# Introduction

Offline support lets Flutter display previously available data without a network connection and synchronize changes later. Caching stores data temporarily; offline-first adds explicit local authority and sync behavior.

# Why this concept exists

Mobile networks are intermittent, but users still expect previously loaded content and drafts to remain available.

# Real-world analogy

An offline notebook lets a field worker record changes, then reconcile them with the central office once connected.

# Internal Working

Repository reads cache/local database, reports freshness, and optionally fetches DRF data. Offline writes become pending operations; synchronization must handle retries, conflicts, duplicate delivery, and account separation.

# Data Flow

Flutter UI -> repository -> local cache/database -> immediate state
-> network sync -> DRF auth/validation/database -> conflict/result -> local reconciliation.

# Complete Example 1

```dart
Future<List<Product>> listProducts() async {
  final cached = await localStore.readProducts();
  if (cached.isNotEmpty) {
    refreshInBackground();
    return cached;
  }
  final fresh = await api.listProducts();
  await localStore.replaceProducts(fresh);
  return fresh;
}
```

The repository returns cached data quickly and refreshes when appropriate; a cache miss fetches then saves. Add explicit stale indicators, error policy, and synchronization safety. The omitted background method must catch/report failures rather than creating an unhandled Future.

# Complete Example 2

```python
class UpdateProfileView(APIView):
    permission_classes = [IsAuthenticated]

    def patch(self, request):
        serializer = ProfileSerializer(request.user.profile, data=request.data, partial=True)
        serializer.is_valid(raise_exception=True)
        serializer.save()
        return Response(serializer.data)
```

The server validates and applies a profile patch to the authenticated user's profile. An offline client must not assume its local edit succeeded until reconciliation; add conflict/version policy for concurrent edits.

# Complete Example 3

```dart
class PendingChange {
  const PendingChange({
    required this.idempotencyKey,
    required this.payload,
    required this.createdAt,
  });
  final String idempotencyKey;
  final Map<String, Object?> payload;
  final DateTime createdAt;
}
```

The pending record persists an operation identity and payload for retry. Store this in an appropriate local database, scope it to the signed-in user, and avoid storing sensitive fields unnecessarily. The server must honor idempotency keys for safe duplicate retries.

# Advanced Example

Define conflict resolution by domain: last-write-wins only when acceptable; otherwise use version numbers/ETags and surface conflicts. Use a sync queue with bounded retry/backoff and observable pending/failed states.

# Under the Hood

HTTP cache headers can help network caches, but app-level offline persistence is separate. Local and server databases are independent; synchronization is a distributed consistency problem.

# Performance Notes

Limit cache size, paginate sync, batch changes, and avoid repeatedly downloading unchanged data using ETags or modification timestamps where supported.

# Security Notes

Encrypt sensitive local data when required, clear user-scoped caches on logout, and never treat cached authorization as current server permission.

# Best Practices

- Show freshness and sync state.
- Make retries idempotent.
- Scope local data by user/account.
- Define conflict and deletion semantics.

# Common Mistakes

- Calling a cache “offline-first” without queued writes.
- Silently overwriting server changes.
- Replaying one user's pending operations after account switch.

# Debugging

Inspect local cache timestamps, queue state, server response/version, retry history, and current account scope.

# Interview Questions

1. Difference between caching and offline-first?
2. Why use idempotency keys?
3. How can clients detect conflicts?

# Practice Exercises

1. Add stale-while-revalidate product reads.
2. Queue an offline profile update safely.
3. Test account switching and conflict behavior.

# Chapter Summary

Offline integration is a synchronization design, not simply a local cache. Define freshness, retries, idempotency, conflicts, and account isolation explicitly.
