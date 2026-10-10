# Introduction

Loading and progress UI communicates that asynchronous API work is underway and indicates when the user can act again.

# Why this concept exists

HTTP operations take time. Without feedback, users may tap repeatedly, assume the app is frozen, or lose confidence.

# Real-world analogy

A queue display shows that a request is being processed and how much of a long upload is complete.

# Internal Working

Flutter state changes from idle to loading, then success or error. The widget rebuilds to reflect that state. Upload progress can be observed from streamed byte counts; ordinary request Futures only report completion.

# Data Flow

User action -> loading state -> request -> DRF processing -> response -> success/error state -> UI rebuild.

# Complete Example 1

```dart
class SaveButtonState extends State<SaveButton> {
  bool _loading = false;

  Future<void> save() async {
    setState(() => _loading = true);
    try {
      await widget.repository.save();
    } finally {
      if (mounted) setState(() => _loading = false);
    }
  }

  @override
  Widget build(BuildContext context) => ElevatedButton(
        onPressed: _loading ? null : save,
        child: _loading
            ? const CircularProgressIndicator()
            : const Text('Save'),
      );
}
```

The state disables duplicate taps, shows progress, and clears loading in `finally`. `mounted` prevents updating a disposed widget. Catch errors separately to display them; this snippet guarantees loading reset but does not silently swallow the thrown failure.

# Complete Example 2

```python
class ExportView(APIView):
    permission_classes = [IsAuthenticated]

    def post(self, request):
        job = create_export_job(owner=request.user)
        return Response({"job_id": job.pk, "state": "queued"}, status=202)
```

DRF returns 202 because work continues asynchronously. Flutter should show queued status and poll or subscribe through a documented mechanism, not keep an HTTP request open while a long export runs.

# Complete Example 3

```dart
StreamedResponse upload = await request.send();
```

`package:http`'s streamed upload response alone does not provide a universal upload progress callback through this API usage. For progress, use a client/package that exposes upload byte progress or implement a streaming transport with measured bytes; keep backend upload limits and cancellation aligned.

# Advanced Example

Represent request state as a sealed/result type (`idle`, `loading`, `success`, `failure`) and keep it in a controller/BLoC/Provider. Prevent stale responses from replacing newer search results by using request IDs or cancellation.

# Under the Hood

`setState` schedules a rebuild; it does not block the event loop. Network I/O is asynchronous, but JSON decoding/large transformations run on the UI isolate unless moved elsewhere.

# Performance Notes

Use lightweight progress indicators, avoid rebuilding large trees, and move expensive decoding to an isolate for genuinely large payloads.

# Security Notes

Never render server-provided error HTML. Disable duplicate sensitive actions while requests are in flight and avoid exposing progress endpoints across users.

# Best Practices

- Show loading, success, empty, and failure states.
- Disable duplicate submissions.
- Reset state in `finally`.
- Use operation status for long-running backend work.

# Common Mistakes

- Setting loading false only on success.
- Calling setState after disposal.
- Showing an indefinite spinner for a queued job.

# Debugging

Inspect state transitions, widget lifecycle, future completion, duplicate calls, and task status responses.

# Interview Questions

1. Why use `finally` for loading state?
2. What does 202 mean to a progress UI?
3. When can JSON parsing affect UI responsiveness?

# Practice Exercises

1. Add loading and validation error UI to a form.
2. Show upload progress using a progress-capable client.
3. Build a queued-job status screen.

# Chapter Summary

Progress UI is a state contract between asynchronous transport and the user. Handle lifecycle and errors, prevent duplicate writes, and represent long-running jobs as tracked operations.
