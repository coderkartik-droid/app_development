# Introduction

Provider is a Flutter package commonly used to expose state objects to descendant widgets. For API features, a `ChangeNotifier` can own loading, data, and error state while calling a repository.

# Why this concept exists

A page-level `FutureBuilder` is convenient for one request. Shared or interactive API state often needs refresh, mutation, and observation across multiple widgets.

# Real-world analogy

Provider is a notice board in a shared office. One coordinator updates the board, and interested screens read the latest status.

# Internal Working

`ChangeNotifier` notifies listeners when state changes. `ChangeNotifierProvider` creates and disposes the notifier; `Consumer` or `context.watch` rebuilds dependents. Network I/O remains asynchronous and belongs in a repository/service rather than `build`.

# Syntax

```dart
ChangeNotifierProvider(
  create: (_) => PostsModel(repository)..load(),
  child: const PostsPage(),
)
```

# Complete Example 1

This runnable sample uses Provider with a fake repository so it needs only the `provider` package and no internet.

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

class Post {
  const Post(this.title);
  final String title;
}

class PostsRepository {
  Future<List<Post>> fetch() async {
    await Future<void>.delayed(const Duration(milliseconds: 500));
    return const [Post('Learn Provider'), Post('Load API data')];
  }
}

class PostsModel extends ChangeNotifier {
  PostsModel(this.repository);
  final PostsRepository repository;
  List<Post> posts = [];
  bool loading = false;
  String? error;

  Future<void> load() async {
    loading = true;
    error = null;
    notifyListeners();
    try {
      posts = await repository.fetch();
    } catch (e) {
      error = 'Could not load posts';
    } finally {
      loading = false;
      notifyListeners();
    }
  }
}

void main() => runApp(
      ChangeNotifierProvider(
        create: (_) => PostsModel(PostsRepository())..load(),
        child: const ProviderApiApp(),
      ),
    );

class ProviderApiApp extends StatelessWidget {
  const ProviderApiApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: ProviderPostsPage());
}

class ProviderPostsPage extends StatelessWidget {
  const ProviderPostsPage({super.key});

  @override
  Widget build(BuildContext context) {
    final model = context.watch<PostsModel>();
    return Scaffold(
      appBar: AppBar(title: const Text('Provider API state')),
      body: model.loading
          ? const Center(child: CircularProgressIndicator())
          : model.error != null
              ? Center(child: Text(model.error!))
              : ListView(
                  children: model.posts.map((post) => ListTile(title: Text(post.title))).toList(),
                ),
      floatingActionButton: FloatingActionButton(
        onPressed: model.load,
        child: const Icon(Icons.refresh),
      ),
    );
  }
}
```

### Line-by-line explanation

- `PostsRepository` simulates asynchronous API work.
- `PostsModel` stores data/loading/error and exposes `load`.
- `notifyListeners` informs Provider consumers before and after the request.
- `finally` resets loading regardless of success/failure.
- `ChangeNotifierProvider` creates and owns the model, then starts loading.
- `context.watch` subscribes the page so it rebuilds when notified.
- The body chooses loading, error, or list UI.
- The FAB calls the model's refresh method.

### Widget tree

```text
ChangeNotifierProvider<PostsModel>
└── MaterialApp → ProviderPostsPage
                  └→ Scaffold → loading/error/ListView + refresh FAB
```

### Execution flow

```text
Provider creates model -> load() -> notify loading
                         -> repository Future -> notify result
                         -> context.watch rebuilds page
```

### ASCII UI preview

```text
+---------------------------+
|   Provider API state      |
+---------------------------+
| • Learn Provider          |
| • Load API data           |
|                       [↻] |
+---------------------------+
```

### Example explanation

The notifier owns presentation state; repository owns data retrieval. Provider disposes the created notifier when its subtree is removed. In this simplified example, errors are shown but the original exception is not retained; production diagnostics should preserve a sanitized cause.

If `notifyListeners` is omitted, the UI may not update. If `load()` is called in `build`, every rebuild can trigger another request.

**Common mistakes:** Use `context.read` for an event-triggered method call; use `watch` when the widget must react to state changes.

# Complete Example 2

Call an action without subscribing:

```dart
context.read<PostsModel>().load();
```

This is suitable inside callbacks. Calling `watch` inside an event callback is usually unnecessary.

# Complete Example 3

Use `Selector` to listen to a smaller piece of state:

```dart
Selector<PostsModel, bool>(
  selector: (_, model) => model.loading,
  builder: (_, loading, __) => loading
      ? const CircularProgressIndicator()
      : const SizedBox.shrink(),
)
```

The selected value limits rebuilds when unrelated model properties change.

# Advanced Example

Represent API state as a sealed state type or use a dedicated state-management approach when boolean combinations become invalid. Add cancellation/stale request protection and keep repositories injectable.

# Under the Hood

Provider uses inherited widget dependencies to expose objects. `ChangeNotifier` listener callbacks schedule dependent widget rebuilds. It does not automatically cache HTTP responses or cancel network requests.

# Performance Notes

- Keep notifier scope local to feature.
- Use `Selector`/`Consumer` to narrow rebuilds.
- Avoid notifying for unchanged state.
- Deduplicate refreshes if needed.

# Best Practices

- Keep network calls out of build.
- Separate notifier and repository.
- Use read for actions, watch/select for rendering.
- Dispose resources owned by notifier.
- Test notifier with fake repository.

# Common Mistakes

- Recreating providers in a rebuild-prone location.
- Calling load repeatedly from build.
- Using one global notifier for unrelated APIs.
- Swallowing errors with no diagnostics.
- Rebuilding large trees unnecessarily.

# Debugging

- Confirm provider is an ancestor of consumer.
- Check `notifyListeners` calls and disposal.
- Watch request count in logs.
- Test loading/error state transitions.

# Interview Questions

1. What does Provider do? Exposes dependencies/state through widget tree.
2. What does ChangeNotifier do? Notifies listeners when state changes.
3. Difference between `watch` and `read`? Subscribe/rebuild versus access without subscription.
4. Does Provider cache API responses? No, application/repository must define that.

# Try It Yourself

1. Replace fake repository with an HTTP repository.
2. Add retry and empty states.
3. Use Selector for the loading indicator.
4. Write notifier tests.

# Challenge Project

Build a Provider-powered searchable list with loading, error, empty, refresh, and fake repository tests.

# Quick Revision Sheet

- Provider exposes state/dependencies.
- ChangeNotifier notifies UI.
- Repository performs API/data-source work.
- `watch` renders state; `read` triggers actions.
- Keep requests out of build and test with fakes.
