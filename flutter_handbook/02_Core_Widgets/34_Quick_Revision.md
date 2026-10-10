# Quick Revision

## 📖 Introduction

This chapter reviews the core ideas from chapters 1–30: widgets describe the UI, layout widgets arrange it, Material widgets provide familiar app structure and controls, and feedback widgets communicate changing outcomes.

## 🤔 Why Do We Need It?

Knowing individual widget names is not enough to build a clear interface. A quick revision helps connect each widget's role to the problem it solves, so you can choose a small, suitable widget tree instead of piling on layout wrappers.

## 🧠 Internal Working

Flutter builds a tree of immutable widget descriptions. Parent widgets pass layout constraints down; children choose sizes within those constraints; layout results are painted. When inputs or state change, Flutter rebuilds the affected descriptions and updates the rendered interface.

Choose widgets by responsibility:

| Need | Common choice |
|---|---|
| Describe a reusable UI piece | `StatelessWidget` or `StatefulWidget` |
| Page structure and Material environment | `MaterialApp`, `Scaffold`, `AppBar` |
| Show or style words | `Text`, `RichText` |
| Show symbols or image assets | `Icon`, `Image` |
| Space or position content | `Padding`, `SizedBox`, `Center` |
| Group related material content | `Card`, `ListTile`, `Divider` |
| Offer an action | Material button variants or `FloatingActionButton` |
| Show ongoing work or an outcome | Progress indicators, `SnackBar`, `AlertDialog`, bottom sheet |

## ⚙ Constructor (if applicable)

Quick composition example:

```dart
Scaffold(
  appBar: AppBar(title: const Text('Overview')),
  body: const Center(
    child: Padding(
      padding: EdgeInsets.all(24),
      child: Text('Choose one clear action.'),
    ),
  ),
)
```

## 📋 Properties (only relevant ones)

| Concept | Remember |
|---|---|
| `child` and `children` | A widget accepts one child or a list of children depending on its layout role |
| Parent constraints | A requested size cannot ignore its parent's limits |
| `const` | Use for configurations that never change |
| `onPressed` | A non-null callback enables a Material button |
| `value` on progress indicators | Null is indeterminate; a fraction represents known progress |
| `BuildContext` | Identifies a widget's place in the tree and provides inherited services |

## 💻 Complete Runnable Example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const RevisionApp());

class RevisionApp extends StatelessWidget {
  const RevisionApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('Widget Revision')),
        body: Center(
          child: Card(
            margin: const EdgeInsets.all(24),
            child: Padding(
              padding: const EdgeInsets.all(20),
              child: Column(
                mainAxisSize: MainAxisSize.min,
                children: [
                  const Icon(Icons.lightbulb_outline, size: 36),
                  const SizedBox(height: 12),
                  const Text(
                    'Build with purpose',
                    style: TextStyle(fontSize: 20),
                  ),
                  const SizedBox(height: 12),
                  ElevatedButton(
                    onPressed: () {},
                    child: const Text('Got it'),
                  ),
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

- `main` starts `RevisionApp`.
- `MaterialApp` sets up the Material app environment.
- `Scaffold` provides the page structure and top app bar.
- `Center` positions the card in the body area.
- `Card` groups a related icon, message, and action on one surface.
- `Padding` keeps that content away from the card's edges.
- `Column` arranges the group vertically.
- `MainAxisSize.min` makes the column use only the height it needs.
- `Icon` adds a visual cue.
- `SizedBox` creates clear gaps between elements.
- `Text` communicates the main message.
- `ElevatedButton` provides a recognizable action control.
- The sample callback is intentionally empty; in a real screen, replace it with the action this button represents.

## 📱 ASCII Phone Screen Preview

```text
+----------------------+
| Widget Revision      |
|----------------------|
|                      |
|    +------------+    |
|    |    (bulb)  |    |
|    | Build with |    |
|    | purpose    |    |
|    |  [Got it]  |    |
|    +------------+    |
|                      |
+----------------------+
```

## 🌳 Widget Tree Diagram

```text
RevisionApp
└── MaterialApp
    └── Scaffold
        ├── AppBar
        │   └── Text
        └── Center
            └── Card
                └── Padding
                    └── Column
                        ├── Icon
                        ├── SizedBox
                        ├── Text
                        ├── SizedBox
                        └── ElevatedButton
                            └── Text
```

## ⚙ Flutter Execution Flow

```text
runApp installs the root widget
        ↓
MaterialApp and Scaffold create the app/page environment
        ↓
Parents pass constraints; layout widgets position children
        ↓
Widgets paint text, icons, surfaces, and controls
        ↓
User interaction invokes callbacks and may update the tree
```

## ⚠ Common Beginner Mistakes

- Choosing a widget because its name is familiar rather than because it matches the layout need.
- Expecting a widget to exceed the constraints given by its parent.
- Using `Container` for every gap instead of focused widgets such as `Padding` and `SizedBox`.
- Confusing a visible icon with an interactive `IconButton`.
- Using transient feedback for information that must remain available.
- Leaving visible buttons without meaningful behavior.

## 🚀 Professional Usage

Start from the user task and map it to widget responsibilities. Keep each component focused, extract repeated UI into named widgets, and centralize shared visual styles when the app grows. Confirm the result on narrow screens, with large text, and in relevant themes.

## 💡 Best Practices

- Explain the user need before choosing a widget.
- Keep the widget tree readable and avoid unnecessary wrappers.
- Treat `BuildContext` as location-dependent.
- Use layout constraints instead of screen-coordinate positioning.
- Use meaningful labels and accessible descriptions.
- Show loading and feedback only when they match real application state.
- Review behavior as well as appearance.

## 🧪 Try It Yourself

Replace the centered card with a `ListTile` containing a leading icon, title, subtitle, and trailing chevron. Decide whether the row should be interactive and provide a matching callback if it is.

## 🎯 Challenge

Build a small dashboard with an app bar, one summary card, two list rows, and one clear primary action. Use widgets from this section, maintain consistent spacing, and ensure the page can scroll if it becomes taller than the screen.

## ❓ 10 Practice Questions

1. What does a widget describe in Flutter?
2. What information does `BuildContext` represent?
3. Which widgets commonly create a Material page structure?
4. How do `Padding` and `SizedBox` differ in intent?
5. Which widget positions one child in the center of its available area?
6. Why is an `IconButton` different from an `Icon`?
7. What does a null progress value mean?
8. What is the role of a `ScaffoldMessenger`?
9. How do parent constraints affect a requested widget size?
10. What should you check beyond a layout's appearance?

## 📝 Summary

Core widgets are building blocks with distinct responsibilities. Understand the user need, choose the simplest suitable widgets, respect parent constraints, and make interactions, accessibility, and feedback part of the design.
