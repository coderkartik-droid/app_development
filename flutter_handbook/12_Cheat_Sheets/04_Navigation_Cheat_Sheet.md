# Navigation Cheat Sheet

## Most Important Concepts

- A route represents a screen in the navigation stack.
- `push` adds a route; `pop` removes the current route.
- Pass arguments explicitly and return results with the route future.
- Deep links, restoration, and nested navigation require deliberate routing design.

## Syntax Reference

```dart
final result = await Navigator.of(context).push<String>(
  MaterialPageRoute(builder: (_) => const DetailPage()),
);
```

```dart
Navigator.of(context).pop('done');
```

## Common Widgets / Classes

- `Navigator`, `Route`, `MaterialPageRoute`
- `Navigator.push`, `pushReplacement`, `pop`, `maybePop`
- `NavigationBar`, `BottomNavigationBar`, `Drawer`
- Declarative routers (e.g. Router-based packages) for deep links and complex route graphs

## Frequently Used Methods

| Method | Effect |
|---|---|
| `push(route)` | Add route; return future result |
| `pop(result)` | Remove top route and optionally return value |
| `pushReplacement(route)` | Replace current route |
| `pushAndRemoveUntil(...)` | Push and remove matching earlier routes |
| `maybePop()` | Attempt a pop, respecting route policy |

## Parameters & Properties

- `builder`: creates destination page.
- Route `settings`: optional route name/arguments metadata.
- `maintainState`: whether an inactive route retains state (route-specific).
- `PopScope`: control/observe back navigation behavior.

## Short Example

```dart
onPressed: () async {
  final selected = await Navigator.of(context).push<String>(
    MaterialPageRoute(builder: (_) => const PickerPage()),
  );
  if (!context.mounted || selected == null) return;
  setState(() => choice = selected);
}
```

## Common Mistakes

- Calling `setState` after awaiting navigation without checking `context.mounted`.
- Pushing duplicate screens repeatedly.
- Using `pushReplacement` when back navigation should remain available.
- Treating route names as a complete deep-link solution.

## Interview Revision

- Navigator is commonly described as a stack of routes.
- `push` returns a `Future<T?>` completed when the route is popped.
- Imperative Navigator is fine for simple flows; declarative routing helps complex URLs, nested navigation, and deep links.

## Best Practices

- Define route ownership and back behavior.
- Return typed results rather than reaching into prior screens.
- Keep authentication/authorization decisions separate from navigation visibility.
- Test deep links and platform back behavior when supported.

## Memory Tricks

- Push = go forward; pop = return.
- `pushReplacement` means the old route is no longer the back destination.
- Route result travels back through the `push` future.

## One-Page Summary

Navigation changes the active route stack. Use typed route results, check mounted context after awaits, and choose a declarative router when URLs, nested navigation, or restoration make imperative pushes hard to manage.
