# State Management Cheat Sheet

## Most Important Concepts

- State is information that can change and affects UI.
- Local state belongs near its consumer; shared state needs a clear owner.
- Separate UI state (loading/error/selection) from domain/data state.
- State-management tools organize updates; they do not replace good data modeling.

## Quick Comparison

| Approach | Good fit | Watch for |
|---|---|---|
| `setState` | Small, local screen state | State shared across distant widgets |
| `ValueNotifier` | One small observable value | Many coordinated domain objects |
| Provider / ChangeNotifier | Shared app state with simple listening | Broad notifications, lifecycle ownership |
| BLoC/Cubit | Explicit event/state flows, larger feature boundaries | Extra structure for tiny screens |
| Riverpod | Composable, testable dependency/state graph | Learning its provider model |

## Syntax Reference

```dart
setState(() => count++);
```

```dart
final name = ValueNotifier<String>('Ada');
ValueListenableBuilder<String>(
  valueListenable: name,
  builder: (_, value, __) => Text(value),
);
```

## Common Classes / Methods

- `StatefulWidget`, `State`, `setState`
- `ValueNotifier`, `ValueListenableBuilder`
- `ChangeNotifier`, `notifyListeners`, `ListenableBuilder`
- `Stream`, `StreamBuilder`, `FutureBuilder`
- State package concepts: provider, notifier/cubit, event, state, selector

## Parameters & Properties

- `listen`: rebuild when dependency changes (library-specific).
- `select`/selectors: listen to a subset of state where supported.
- `initialData`: initial builder value for async UI.
- `mounted`: whether a `State` remains in the tree.

## Short Example

```dart
class CounterState extends ChangeNotifier {
  int value = 0;
  void increment() {
    value++;
    notifyListeners();
  }
}
```

## Common Mistakes

- Mutating state without notifying listeners.
- Putting all app data in one global object.
- Calling `setState` after disposal.
- Confusing a `Future` result with persistent state.
- Rebuilding large subtrees for tiny state changes.

## Interview Revision

- State management controls ownership, updates, and observation.
- Immutable state makes transitions easier to reason about and test.
- Derived values should generally be calculated from source state, not duplicated.
- Loading, success, empty, and error are distinct UI states.

## Best Practices

- Start with the simplest approach that fits the sharing scope.
- Keep side effects in controllers/repositories, not `build`.
- Make state transitions explicit and testable.
- Scope rebuilds to widgets that actually depend on the changed value.

## Memory Tricks

- **Local state local; shared state shared deliberately.**
- Data owner changes state; listeners rebuild.
- One source of truth beats duplicated totals.

## One-Page Summary

Choose state management by ownership and complexity. Keep state near consumers, expose shared state through a clear boundary, model async states explicitly, and avoid duplicate derived values. Libraries help coordinate state but do not eliminate lifecycle or testing responsibilities.
