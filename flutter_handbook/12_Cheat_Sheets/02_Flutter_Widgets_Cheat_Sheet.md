# Flutter Widgets Cheat Sheet

## Most Important Concepts

- Flutter UI is a tree of immutable widget descriptions.
- `build(context)` describes UI; it should be fast and free of side effects.
- `StatelessWidget` depends only on inputs; `StatefulWidget` has mutable `State`.
- Constraints flow down, sizes flow up, parent sets position.

## Syntax Reference

```dart
class Greeting extends StatelessWidget {
  const Greeting({super.key, required this.name});
  final String name;

  @override
  Widget build(BuildContext context) => Text('Hello, $name');
}
```

## Common Widgets / Classes

| Purpose | Widgets |
|---|---|
| App structure | `MaterialApp`, `Scaffold`, `AppBar`, `SafeArea` |
| Text/input | `Text`, `TextField`, `TextFormField` |
| Layout | `Row`, `Column`, `Stack`, `Wrap`, `Expanded`, `Padding` |
| Lists | `ListView`, `ListView.builder`, `GridView.builder` |
| Actions | `FilledButton`, `IconButton`, `GestureDetector`, `InkWell` |
| State | `StatefulWidget`, `ValueListenableBuilder`, `StreamBuilder` |

## Frequently Used Methods

- `setState(() { ... })`: notify Flutter local state changed.
- `Navigator.of(context).push(...)`: open a route.
- `Theme.of(context)`: access inherited theme.
- `MediaQuery.sizeOf(context)`: read available screen metrics.
- `FocusScope.of(context).unfocus()`: dismiss keyboard focus.

## Parameters & Properties

- `child`: one child; `children`: list of children.
- `padding` adds space inside a boundary; `margin` is outside in widgets that support it.
- `key` helps preserve element identity across rebuilds.
- `mainAxisAlignment` and `crossAxisAlignment` control flex layout.
- `fit`, `maxLines`, `overflow`, and `constraints` control sizing/content behavior.

## Short Example

```dart
Scaffold(
  appBar: AppBar(title: const Text('Tasks')),
  body: ListView.builder(
    itemCount: tasks.length,
    itemBuilder: (context, index) => ListTile(
      key: ValueKey(tasks[index].id),
      title: Text(tasks[index].title),
    ),
  ),
)
```

## Common Mistakes

- Calling network/database writes in `build`.
- Putting a `ListView` in an unbounded `Column` without `Expanded` or constraints.
- Using list index as key for reorderable stateful rows.
- Forgetting to dispose controllers, focus nodes, and animation controllers.
- Overusing `setState` at a high tree level.

## Interview Revision

- Widgets are immutable configurations; elements retain identity/state.
- `build` may run frequently; it should be deterministic and cheap.
- `Expanded` fills flex space; `Flexible` can take less than all of it.
- Keys help Flutter match old and new widgets during reconciliation.

## Best Practices

- Extract reusable widgets when it improves clarity.
- Use `const` constructors for static subtrees.
- Keep state near the widgets that need it.
- Respect accessibility, text scaling, and responsive constraints.

## Memory Tricks

- **Down constraints, up size, parent position.**
- `child` = one; `children` = many.
- Widget describes; element owns identity; render object lays out/paints.

## One-Page Summary

Compose small widgets into a tree. Let constraints determine layout, keep builds pure, give dynamic list rows stable keys, and place state where it is needed. Use built-in Material widgets and semantics before inventing custom controls.
