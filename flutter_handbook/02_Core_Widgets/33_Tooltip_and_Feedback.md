# Tooltip and Feedback

## 📖 Introduction

This chapter brings together two kinds of user guidance: a `Tooltip` explains a control when requested, and feedback widgets such as `SnackBar` confirm the result of an action.

## 🤔 Why Do We Need It?

A compact icon may need a short explanation, and a completed action may need confirmation. If the interface gives neither, users may not know what a control does or whether their action worked.

Tooltips clarify the meaning of a control; feedback reports a change or result. Neither should hide information that must remain visible or interrupt users unnecessarily.

## 🧠 Internal Working

`Tooltip` wraps a child and associates a short message with it. It can show the message in response to supported pointer or touch interactions and contributes context for assistive technologies.

`SnackBar` is presented by `ScaffoldMessenger`. The messenger manages its display and duration. The callback for the control must first complete or confirm the relevant action; the app then decides what feedback to show.

## ⚙ Constructor (if applicable)

```dart
Tooltip(
  message: 'Refresh items',
  child: IconButton(
    tooltip: 'Refresh items',
    onPressed: () {},
    icon: const Icon(Icons.refresh),
  ),
)
```

For a button that supports a tooltip, setting its built-in `tooltip` property is generally enough; avoid wrapping it in a duplicate tooltip.

## 📋 Properties (only relevant ones)

| Widget/property | Purpose |
|---|---|
| `Tooltip.message` | Brief explanation of a child |
| `IconButton.tooltip` | Tooltip and accessible description for an icon action |
| `SnackBar.content` | Message displayed as feedback |
| `SnackBar.duration` | How long the feedback remains visible |
| `SnackBar.action` | Optional action associated with the message |
| `ScaffoldMessenger.showSnackBar` | Presents a snack bar in the scaffold environment |

## 💻 Complete Runnable Example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const TooltipFeedbackApp());

class TooltipFeedbackApp extends StatelessWidget {
  const TooltipFeedbackApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('Tooltip and Feedback')),
        body: Builder(
          builder: (pageContext) {
            return Center(
              child: IconButton(
                tooltip: 'Refresh items',
                onPressed: () {
                  ScaffoldMessenger.of(pageContext).showSnackBar(
                    const SnackBar(
                      content: Text('Items refreshed'),
                      duration: Duration(seconds: 2),
                    ),
                  );
                },
                icon: const Icon(Icons.refresh),
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

- `main` starts the sample app.
- `MaterialApp` creates the Material environment for the button and feedback.
- `Scaffold` supplies an app bar, page body, and messenger host.
- `Builder` provides a `pageContext` below the scaffold.
- `Center` places the compact action in the body.
- `IconButton` makes the refresh icon a real interactive control.
- `tooltip` explains the icon action when the user requests more context.
- `onPressed` runs the app's response to the action.
- `ScaffoldMessenger.of(pageContext)` finds the messenger associated with the scaffold.
- `SnackBar` confirms that the example action was selected.
- `duration` keeps the transient message visible for two seconds.
- `Icon` supplies the familiar refresh symbol.

## 📱 ASCII Phone Screen Preview

```text
+----------------------+
| Tooltip and Feedback |
|----------------------|
|                      |
|          [↻]         |
|                      |
|  Items refreshed    |
+----------------------+
```

## 🌳 Widget Tree Diagram

```text
TooltipFeedbackApp
└── MaterialApp
    └── Scaffold
        ├── AppBar
        └── Builder
            └── Center
                └── IconButton
                    └── Icon

After activation:
ScaffoldMessenger
└── SnackBar
    └── Text
```

## ⚙ Flutter Execution Flow

```text
User encounters compact icon control
        ↓
Tooltip supplies a short explanation when requested
        ↓
User activates the control
        ↓
Application performs its action
        ↓
ScaffoldMessenger presents appropriate feedback
```

## ⚠ Common Beginner Mistakes

- Relying on a tooltip to communicate an essential instruction that should be visible.
- Providing a tooltip that does not match the action.
- Showing success feedback before the operation has actually succeeded.
- Using a snack bar for a decision that requires a clear response.
- Showing repeated, unclear messages that compete with one another.
- Using a context above the scaffold when looking up its `ScaffoldMessenger`.

## 🚀 Professional Usage

Pair a tooltip with an icon-only control when its meaning may not be obvious. Show feedback after an action reaches a meaningful result, and select the feedback pattern according to urgency: inline information for persistent context, a snack bar for brief non-blocking confirmation, and a dialog for a required decision.

## 💡 Best Practices

- Make tooltip text concise and action-specific.
- Prefer a button's built-in tooltip when available.
- Keep essential labels and instructions visible.
- Match feedback to the actual outcome, including errors.
- Use context from below the relevant scaffold to access its messenger.
- Offer a snack bar action only when it performs a real, understandable operation.

## 🧪 Try It Yourself

Change the example so the action is a `TextButton` labeled “Refresh.” Decide whether a tooltip is still useful when the control already has a visible label.

## 🎯 Challenge

Create a compact toolbar with an icon-only help action and a refresh action. Give the help action a tooltip, make refresh perform a clear sample operation, and show feedback only after it completes.

## ❓ 10 Practice Questions

1. What problem does a tooltip solve?
2. How does a tooltip differ from a snack bar?
3. Which Material button property can provide an icon button's tooltip?
4. Which object displays a snack bar?
5. Why should a success message wait until success is confirmed?
6. When should important guidance be visible instead of hidden in a tooltip?
7. When is a dialog more suitable than a snack bar?
8. What does the snack bar duration control?
9. Why does the example use `Builder`?
10. How should the app respond to an operation that fails?

## 📝 Summary

Tooltips explain controls; feedback messages communicate outcomes. Use each for its own purpose, keep transient messages accurate and concise, and make essential information visible and accessible.
