# Layouts Cheat Sheet

## Most Important Concepts

- Flutter layout is constraint-based: constraints go down, sizes go up, position is parent-controlled.
- A box chooses a size within its parent's min/max bounds.
- Scrollables need bounded space along their scroll axis.
- Responsive layout should adapt to available constraints, not assume one device size.

## Syntax Reference

```dart
Row(
  children: [
    const Icon(Icons.star),
    const SizedBox(width: 8),
    Expanded(child: Text(title)),
  ],
)
```

## Common Widgets / Classes

| Need | Widget |
|---|---|
| Horizontal/vertical | `Row`, `Column` |
| Layering | `Stack`, `Positioned`, `Align` |
| Flexible space | `Expanded`, `Flexible`, `Spacer` |
| Wrapping | `Wrap` |
| Scroll | `SingleChildScrollView`, `ListView`, `GridView` |
| Sizing | `SizedBox`, `ConstrainedBox`, `AspectRatio`, `FractionallySizedBox` |
| Insets | `Padding`, `SafeArea` |

## Parameters & Properties

- `mainAxisAlignment`: distribution along main direction.
- `crossAxisAlignment`: alignment across main direction.
- `mainAxisSize`: whether a flex layout takes max or min main-axis space.
- `fit`: how a child fills available space in `Stack`.
- `constraints`: min/max width and height bounds.
- `shrinkWrap: true`: sizes a scrollable to content; can cost more layout work.

## Short Example

```dart
LayoutBuilder(
  builder: (context, constraints) {
    final columns = constraints.maxWidth > 700 ? 3 : 1;
    return GridView.builder(
      gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
        crossAxisCount: columns,
      ),
      itemCount: items.length,
      itemBuilder: (context, index) => ItemCard(item: items[index]),
    );
  },
)
```

## Common Mistakes

- `Expanded` outside `Row`, `Column`, or `Flex`.
- Unbounded-height scrollable inside a `Column`.
- Using fixed dimensions where text must scale or wrap.
- Nesting scroll views without a clear scroll owner.
- Using `shrinkWrap` to hide layout errors instead of fixing constraints.

## Quick Comparison

| Widget | Use |
|---|---|
| `Expanded` | Fill remaining flex space |
| `Flexible` | Share space but allow smaller size |
| `Wrap` | Move children to next run when needed |
| `Stack` | Overlap children |
| `LayoutBuilder` | React to parent constraints |

## Interview Revision

- Why overflow? Child requires more than parent constraints allow.
- Why `Expanded`? Gives a flex child a bounded share of remaining space.
- `MediaQuery` measures screen-level context; `LayoutBuilder` measures local parent constraints.

## Best Practices

- Design from constraints and content needs.
- Test narrow, wide, landscape, and large-text cases.
- Prefer `ListView.builder` for long lists.
- Keep one primary scrollable when possible.

## Memory Tricks

- **Constraints down, sizes up, positions down.**
- `Row` main axis is horizontal; `Column` main axis is vertical.
- Expanded fills; Wrap reflows.

## One-Page Summary

Diagnose layout by tracing constraints from parent to child. Bound scrollables, use flex widgets only within flex layouts, wrap variable-length content, and adapt to local constraints. Avoid fixed-size assumptions and unnecessary nested scrolling.
