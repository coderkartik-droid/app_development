# TextButton

## 📖 Introduction

`TextButton` is a low-emphasis Material button whose label is not placed on a filled surface by default. It is suited to secondary actions and actions in compact areas such as dialogs.

## 🤔 Why Do We Need It?

Not every action should visually compete with the primary action. A text button keeps a real, accessible button interaction while using a lighter visual treatment.

## 🧠 Internal Working

`TextButton` participates in Material button interactions, including enabled/disabled behavior and pressed, hover, and focus states. Its `onPressed` callback enables it when non-null and disables it when null. The app supplies the action and can customize the button through a `ButtonStyle`.

## ⚙ Constructor (if applicable)

```dart
TextButton(
  onPressed: () {},
  child: const Text('Cancel'),
)
```

## 📋 Properties (only relevant ones)

| Property | Purpose |
|---|---|
| `onPressed` | Action callback; `null` disables the button |
| `child` | Usually the action label |
| `style` | Optional styling for color, padding, shape, and other button states |
| `onLongPress` | Optional long-press callback |
| `key` | Helps Flutter identify the widget in the tree |

## 💻 Complete Runnable Example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const TextButtonApp());

class TextButtonApp extends StatelessWidget {
  const TextButtonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('TextButton')),
        body: Builder(
          builder: (scaffoldContext) {
            return Center(
              child: TextButton(
                onPressed: () {
                  ScaffoldMessenger.of(scaffoldContext).showSnackBar(
                    const SnackBar(content: Text('Help selected')),
                  );
                },
                child: const Text('Get help'),
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

- `main` launches `TextButtonApp`.
- `MaterialApp` supplies the Material app environment.
- `Scaffold` provides the page structure and messenger.
- `Center` positions the sample action.
- `TextButton` creates a low-emphasis button with standard interaction behavior.
- Its non-null `onPressed` callback makes it enabled.
- Pressing it asks `ScaffoldMessenger` to show a short confirmation.
- The `Text` child is the label users see.

## 📱 ASCII Phone Screen Preview

```text
+----------------------+
| TextButton           |
|----------------------|
|                      |
|       Get help       |
|                      |
+----------------------+
```

## 🌳 Widget Tree Diagram

```text
TextButtonApp
└── MaterialApp
    └── Scaffold
        ├── AppBar
        └── Center
            └── TextButton
                └── Text
```

## ⚙ Flutter Execution Flow

```text
TextButton is built from its label and callback
        ↓
Material handles focus, hover, and pressed states
        ↓
User activates the button
        ↓
The app's onPressed callback runs
```

## ⚠ Common Beginner Mistakes

- Using a text button for the most important action when it needs stronger prominence.
- Forgetting that `onPressed: null` disables it.
- Styling it so faintly that users cannot recognize it as actionable.
- Using a button label that describes a topic rather than an action.
- Creating an empty callback in production code.

## 🚀 Professional Usage

Use text buttons for secondary actions, dialog choices, or compact actions that should not dominate the screen. Pair them with a clear primary action when both are present, and preserve adequate touch targets.

## 💡 Best Practices

- Match visual emphasis to the action's importance.
- Use concise verbs such as “Cancel,” “Learn more,” or “Retry.”
- Keep the label visible and distinguishable from ordinary body text.
- Implement the behavior and any needed feedback.
- Use consistent styling across comparable actions.

## 🧪 Try It Yourself

Put a `TextButton` and an `ElevatedButton` in a `Row`. Compare their visual emphasis and consider which action should be primary.

## 🎯 Challenge

Build a confirmation area with a primary “Delete” or “Confirm” action and a secondary “Cancel” text button. Make the visual hierarchy and behavior clear.

## ❓ 10 Practice Questions

1. What is the visual emphasis of a `TextButton`?
2. Which actions commonly suit this button style?
3. How does `onPressed` affect enabled state?
4. Does a text button have normal Material interaction behavior?
5. Why should a text button remain visually recognizable?
6. When might an elevated button be more appropriate?
7. What makes a button label actionable?
8. Why are touch targets important even for low-emphasis actions?
9. What does `style` customize?
10. How can two actions show a clear hierarchy?

## 📝 Summary

`TextButton` is a functional, low-emphasis Material action. Use it for secondary actions, give it a clear label and adequate touch area, and keep the primary action visually distinct.
