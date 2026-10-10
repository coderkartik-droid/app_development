# OutlinedButton

## 📖 Introduction

`OutlinedButton` is a Material button with an outlined shape and no filled surface by default. It provides more visual emphasis than a text button while usually remaining secondary to a filled primary action.

## 🤔 Why Do We Need It?

Interfaces often need an action that is visible and easy to recognize but should not compete with the primary filled button. An outline communicates “this is a button” while keeping the visual weight restrained.

## 🧠 Internal Working

`OutlinedButton` uses Material's button interaction system for presses, focus, and hover. A non-null `onPressed` enables it; null disables it. Its appearance can be customized with `ButtonStyle`, including shape, side, padding, and state-dependent colors.

## ⚙ Constructor (if applicable)

```dart
OutlinedButton(
  onPressed: () {},
  child: const Text('Choose another'),
)
```

## 📋 Properties (only relevant ones)

| Property | Purpose |
|---|---|
| `onPressed` | Action callback; `null` disables the button |
| `child` | Usually the button label |
| `style` | Optional button appearance and shape |
| `onLongPress` | Optional long-press callback |
| `key` | Helps Flutter identify the widget in the tree |

## 💻 Complete Runnable Example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const OutlinedButtonApp());

class OutlinedButtonApp extends StatelessWidget {
  const OutlinedButtonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('OutlinedButton')),
        body: Builder(
          builder: (scaffoldContext) {
            return Center(
              child: OutlinedButton(
                onPressed: () {
                  ScaffoldMessenger.of(scaffoldContext).showSnackBar(
                    const SnackBar(content: Text('Another option selected')),
                  );
                },
                child: const Text('Choose another'),
              ),
            );
          },
        ),
      ),
    );
  }
}
```

## 🔍 Line-by-Line Example Explanation

- `main` starts the app.
- `MaterialApp` establishes the Material environment.
- `Scaffold` provides the page and the messenger used for feedback.
- `Center` places the control in the body.
- `OutlinedButton` creates a button with a visible outline.
- Its non-null callback enables the button.
- When tapped, the callback displays a short confirmation using `ScaffoldMessenger`.
- `Text` supplies a clear action label.

## 📱 ASCII Phone Screen Preview

```text
+----------------------+
| OutlinedButton       |
|----------------------|
|                      |
|  +----------------+  |
|  | Choose another |  |
|  +----------------+  |
|                      |
+----------------------+
```

## 🌳 Widget Tree Diagram

```text
OutlinedButtonApp
└── MaterialApp
    └── Scaffold
        ├── AppBar
        └── Center
            └── OutlinedButton
                └── Text
```

## ⚙ Flutter Execution Flow

```text
Button receives label, style, and callback
        ↓
Material draws its outline and interaction states
        ↓
User taps the enabled button
        ↓
onPressed performs the app-defined action
```

## ⚠ Common Beginner Mistakes

- Giving the outline too little contrast against the background.
- Using outlined and filled buttons with no clear hierarchy.
- Setting `onPressed` to null and expecting an active button.
- Making an outlined button represent an action that should be unmistakably primary.
- Changing its style inconsistently across the app.

## 🚀 Professional Usage

Use `OutlinedButton` for a secondary action that needs a clear button boundary. It commonly pairs with a filled primary button, such as “Save” beside “Preview.” Make sure its border and label remain legible in every theme.

## 💡 Best Practices

- Pair it with stronger or weaker variants according to the action hierarchy.
- Check contrast in light and dark themes.
- Keep labels short and action-oriented.
- Implement the callback rather than shipping a visual-only control.
- Centralize repeated style choices in a theme or shared component.

## 🧪 Try It Yourself

Place an `OutlinedButton` next to a `TextButton` and an `ElevatedButton`. Rank them by visual emphasis and decide which action should be primary.

## 🎯 Challenge

Create a row of two actions for a media item: a filled primary action and an outlined secondary action. Make both callbacks respond with distinct feedback.

## ❓ 10 Practice Questions

1. What distinguishes an `OutlinedButton` visually?
2. What action emphasis commonly suits this variant?
3. How is the button enabled or disabled?
4. Which property customizes the outline and shape?
5. Why must the border contrast with its background?
6. How can it pair with an elevated button?
7. What does a null `onPressed` indicate?
8. Why centralize repeated button styles?
9. Does the button implement the app's action for you?
10. What should be tested across themes?

## 📝 Summary

`OutlinedButton` offers a clearly bounded action with moderate emphasis. It works well for secondary actions, provided its border and label remain legible and its callback is implemented.
