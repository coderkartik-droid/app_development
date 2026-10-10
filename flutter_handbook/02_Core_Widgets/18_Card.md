# Card

## 📖 Introduction

`Card` is a Material surface for grouping related content into a visually distinct area. It provides a surface shape and elevation behavior that can match the app's Material theme.

## 🤔 Why Do We Need It?

Related information is easier to scan when it is visually grouped. A `Card` provides that grouping without manually assembling a background, rounded shape, and shadow for each instance.

A card is a visual container, not automatically an interactive control. Add a button or other interaction explicitly when the user should act on it.

## 🧠 Internal Working

`Card` paints a Material surface behind its child. The theme and card properties determine its color, shape, clipping behavior, and elevation. Its child determines the card's content and much of its size.

Elevation is a visual depth cue. It does not change the order in which widgets are laid out, and it should not be used as a substitute for meaningful hierarchy or accessibility.

## ⚙ Constructor (if applicable)

```dart
Card(
  elevation: 2,
  child: const Padding(
    padding: EdgeInsets.all(16),
    child: Text('Grouped information'),
  ),
)
```

## 📋 Properties (only relevant ones)

| Property | Purpose |
|---|---|
| `child` | Content displayed on the card surface |
| `color` | Surface color |
| `elevation` | Visual depth of the surface |
| `shape` | Surface outline and corners |
| `margin` | Space outside the card |
| `clipBehavior` | Whether and how content is clipped to the card shape |
| `key` | Helps Flutter identify the widget in the tree |

## 💻 Complete Runnable Example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const CardApp());

class CardApp extends StatelessWidget {
  const CardApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('Card')),
        body: Center(
          child: Card(
            elevation: 3,
            margin: const EdgeInsets.all(20),
            child: Padding(
              padding: const EdgeInsets.all(16),
              child: Column(
                mainAxisSize: MainAxisSize.min,
                crossAxisAlignment: CrossAxisAlignment.start,
                children: const [
                  Text(
                    'Daily reminder',
                    style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
                  ),
                  SizedBox(height: 8),
                  Text('Take a short break and stretch.'),
                ],
              ),
            ),
          ),
        ),
      ),
    );
  }
}
```

## 🔍 Line-by-Line Example Explanation

- `main` starts the application with `CardApp`.
- `MaterialApp` establishes the Material theme and app structure.
- `Scaffold` and `AppBar` create the page and title.
- `Center` places the sample card in the body.
- `Card` draws a Material surface with a modest elevation and outside margin.
- `Padding` gives the card's content space from its edges.
- `Column` groups the title and supporting text vertically.
- `MainAxisSize.min` prevents the column from claiming more height than its content needs.
- `CrossAxisAlignment.start` aligns both text lines to the same leading edge.
- The first `Text` styles the card heading.
- `SizedBox` creates a small vertical gap.
- The second `Text` supplies supporting information.

## 📱 ASCII Phone Screen Preview

```text
+----------------------+
| Card                 |
|----------------------|
|                      |
|   +--------------+   |
|   | Daily        |   |
|   | reminder     |   |
|   |              |   |
|   | Take a short |   |
|   | break...     |   |
|   +--------------+   |
|                      |
+----------------------+
```

## 🌳 Widget Tree Diagram

```text
CardApp
└── MaterialApp
    └── Scaffold
        ├── AppBar
        └── Center
            └── Card
                └── Padding
                    └── Column
                        ├── Text (heading)
                        ├── SizedBox
                        └── Text (details)
```

## ⚙ Flutter Execution Flow

```text
Card receives parent constraints
        ↓
Child content is measured inside the card
        ↓
Card resolves its surface size and shape
        ↓
Surface, elevation, and child are painted
```

## ⚠ Common Beginner Mistakes

- Treating a `Card` as clickable without adding an interaction.
- Forgetting internal padding, leaving text close to the card edge.
- Using excessive elevation and making every card compete for attention.
- Expecting card margin to create space inside its surface.
- Placing a very large, unbounded list inside a card instead of managing scrolling deliberately.

## 🚀 Professional Usage

Use cards to group related content, not simply to decorate every section. Choose elevation and shape from the app's visual system, and make an interactive card's semantics and feedback clear with an appropriate interaction widget.

## 💡 Best Practices

- Keep each card focused on one related content group.
- Add readable internal padding.
- Use elevation sparingly and consistently.
- Use a dedicated button or gesture-aware Material component when the card is interactive.
- Consider `ListTile` for a compact title/subtitle/leading/trailing row.

## 🧪 Try It Yourself

Set `elevation` to `0` and compare the visual depth. Then add a subtle `shape` using a rounded rectangle border.

## 🎯 Challenge

Create two cards for two related tasks. Each card should have a heading, description, and clear action. Keep their spacing and typography consistent.

## ❓ 10 Practice Questions

1. What visual role does `Card` provide?
2. Does a card become interactive automatically?
3. What does elevation communicate?
4. What is the difference between card margin and child padding?
5. Which property controls the card's outline?
6. What does `clipBehavior` affect?
7. Why should elevation be used sparingly?
8. When might `ListTile` be a better fit?
9. How can a card's child influence its size?
10. What should be added when a card represents an action?

## 📝 Summary

`Card` provides a themed Material surface for related content. It supplies visual grouping, not automatic interaction. Use clear internal spacing, restrained elevation, and an explicit interaction when users can act on the card.
