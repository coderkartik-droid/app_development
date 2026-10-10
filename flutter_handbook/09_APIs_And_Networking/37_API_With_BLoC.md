# Introduction

BLoC (Business Logic Component) separates events, business logic, and emitted states. For API features, a BLoC receives an event, calls a repository, and emits loading, success, or failure state.

# Why this concept exists

As API screens grow, explicit event/state flow can make behavior easier to reason about and test, especially with retries, filters, and concurrent requests.

# Real-world analogy

A control room receives a request ticket (event), performs the work, and broadcasts status updates (states) to screens.

# Internal Working

UI dispatches events. A BLoC handles them asynchronously and emits states through a stream. `BlocBuilder` rebuilds for relevant state changes; `BlocListener` handles one-off effects such as navigation or snackbars. Repositories perform I/O.

# Syntax

```dart
on<LoadItems>((event, emit) async {
  emit(const ItemsLoading());
  final items = await repository.load();
  emit(ItemsLoaded(items));
});
```

# Complete Example 1

This runnable-style example uses `flutter_bloc` with a fake repository. Add `flutter_bloc` to dependencies. The API boundary is simulated for clarity.

```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';

class Item {
  const Item(this.title);
  final String title;
}

class ItemRepository {
  Future<List<Item>> load() async {
    await Future<void>.delayed(const Duration(milliseconds: 400));
    return const [Item('First item'), Item('Second item')];
  }
}

sealed class ItemsState {
  const ItemsState();
}

class ItemsInitial extends ItemsState {
  const ItemsInitial();
}

class ItemsLoading extends ItemsState {
  const ItemsLoading();
}

class ItemsLoaded extends ItemsState {
  const ItemsLoaded(this.items);
  final List<Item> items;
}

class ItemsFailure extends ItemsState {
  const ItemsFailure(this.message);
  final String message;
}

sealed class ItemsEvent {
  const ItemsEvent();
}

class LoadItems extends ItemsEvent {
  const LoadItems();
}

class ItemsBloc extends Bloc<ItemsEvent, ItemsState> {
  ItemsBloc(this.repository) : super(const ItemsInitial()) {
    on<LoadItems>((event, emit) async {
      emit(const ItemsLoading());
      try {
        emit(ItemsLoaded(await repository.load()));
      } catch (error) {
        emit(const ItemsFailure('Unable to load items'));
      }
    });
  }

  final ItemRepository repository;
}

void main() => runApp(
      RepositoryProvider(
        create: (_) => ItemRepository(),
        child: const BlocApiApp(),
      ),
    );

class BlocApiApp extends StatelessWidget {
  const BlocApiApp({super.key});

  @override
  Widget build(BuildContext context) => MaterialApp(
        home: BlocProvider(
          create: (context) => ItemsBloc(context.read<ItemRepository>())
            ..add(const LoadItems()),
          child: const ItemsPage(),
        ),
      );
}

class ItemsPage extends StatelessWidget {
  const ItemsPage({super.key});

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('BLoC API flow')),
        body: BlocBuilder<ItemsBloc, ItemsState>(
          builder: (context, state) => switch (state) {
            ItemsInitial() || ItemsLoading() =>
              const Center(child: CircularProgressIndicator()),
            ItemsLoaded(:final items) => ListView(
                children: items.map((item) => ListTile(title: Text(item.title))).toList(),
              ),
            ItemsFailure(:final message) => Center(child: Text(message)),
          },
        ),
        floatingActionButton: FloatingActionButton(
          onPressed: () => context.read<ItemsBloc>().add(const LoadItems()),
          child: const Icon(Icons.refresh),
        ),
      );
}
```

### Line-by-line explanation

- `ItemRepository` simulates asynchronous data retrieval.
- Sealed `ItemsState` variants make loading, loaded, and failure states explicit.
- `LoadItems` is an event representing user intent.
- `ItemsBloc` registers a handler, emits loading, awaits repository, then emits success or failure.
- `RepositoryProvider` provides the repository; `BlocProvider` creates and owns the BLoC.
- The initial load event is dispatched after construction.
- `BlocBuilder` maps each state to a widget tree.
- Refresh dispatches another event.

### Widget tree

```text
RepositoryProvider<ItemRepository>
└── MaterialApp
    └── BlocProvider<ItemsBloc>
        └── ItemsPage
            └── Scaffold → BlocBuilder(state-specific body) + FAB
```

### Execution flow

```text
UI dispatches LoadItems -> BLoC emits Loading
                         -> repository Future
                         -> emits Loaded or Failure
                         -> BlocBuilder renders state
```

### ASCII UI preview

```text
+---------------------------+
|       BLoC API flow       |
+---------------------------+
| • First item              |
| • Second item             |
|                       [↻] |
+---------------------------+
```

### Example explanation

Each mutually exclusive state is represented by a type, reducing invalid combinations such as `loading=true` and `error!=null` simultaneously. The BLoC catches repository failures and emits a user-facing failure state; production logging should retain sanitized cause.

If the BLoC is not closed, its stream/listeners can leak; `BlocProvider` closes instances it creates. If the event handler starts inside the builder, repeated builds can duplicate requests.

**Common mistakes:** Do not use `BlocBuilder` for one-off effects such as navigation; use `BlocListener` or `BlocConsumer`.

# Complete Example 2

Trigger an event from a button:

```dart
context.read<ItemsBloc>().add(const LoadItems());
```

Events describe intent; the BLoC owns the asynchronous response flow.

# Complete Example 3

Use `BlocListener` for a snackbar on a failure state:

```dart
BlocListener<ItemsBloc, ItemsState>(
  listener: (context, state) {
    if (state is ItemsFailure) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text(state.message)),
      );
    }
  },
  child: const ItemsView(),
)
```

One-off side effects should not be repeated whenever a state rebuilds.

# Advanced Example

Handle concurrent search events deliberately with event transformers or request IDs. Cancel/ignore stale results, preserve prior content during refresh if desired, and represent pagination append state separately.

# Under the Hood

BLoC exposes a stream of states. Event handlers await repository work and emit new states; builders subscribe through Flutter widgets and rebuild when selected state changes.

# Performance Notes

- Use `buildWhen` or selectors to narrow rebuilds.
- Avoid recreating BLoCs on every build.
- Keep state payloads immutable.
- Deduplicate or cancel concurrent requests.

# Best Practices

- Use events for user/system intent.
- Use sealed states for distinct outcomes.
- Keep I/O in repositories.
- Keep one-off effects in listeners.
- Test event-to-state sequences with fake repositories.

# Common Mistakes

- Doing HTTP in `build`.
- Putting every transient UI detail into global state.
- Emitting mutable lists.
- Ignoring concurrency/stale response behavior.
- Using builders for repeated side effects.

# Debugging

- Observe event and state sequence.
- Check provider scope and BLoC lifecycle.
- Test repository failure and empty results.
- Verify concurrency transformer behavior.

# Interview Questions

1. What is the BLoC flow? Event -> logic/repository -> state -> UI.
2. What does `BlocBuilder` do? Rebuilds UI for states.
3. What does `BlocListener` do? Handles one-off side effects.
4. Where should HTTP live? In API client/repository, invoked by BLoC.
5. How test BLoC? Inject fake repository and assert emitted state sequence.

# Try It Yourself

1. Add retry event and empty state.
2. Add search event with stale response protection.
3. Move error snackbar into BlocListener.
4. Write BLoC tests.

# Challenge Project

Build a paginated API list with BLoC states for initial loading, loaded, refresh, append, and failure, using a fake repository for tests.

# Quick Revision Sheet

- Events express intent; states describe UI result.
- BLoC calls repositories asynchronously.
- BlocBuilder renders; BlocListener handles side effects.
- Provider owns/ closes created BLoC.
- Test event-state sequences and concurrency.
