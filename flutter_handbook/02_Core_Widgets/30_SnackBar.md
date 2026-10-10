# SnackBar

## 📖 Introduction

`SnackBar` is a brief message displayed near the bottom of a screen, typically after an action. It can confirm an event and optionally offer a related action such as undo.

## 🤔 Why Do We Need It?

Users need lightweight feedback when an action succeeds or when a temporary issue needs attention. A snack bar provides that feedback without navigating away from the current screen or interrupting the user with a dialog.

## 🧠 Internal Working

A `SnackBar` is shown by a `ScaffoldMessenger`, which manages its display in relation to the app's scaffold. The messenger queues snack bars and removes them after their duration unless they are dismissed or replaced. The message is not a substitute for durable information that users must be able to revisit.

Use dialogs for decisions that require the user's attention; use snack bars for brief, non-blocking feedback.

## ⚙ Constructor (if applicable)

```dart
SnackBar(
  content: const Text('Saved'),
  duration: const Duration(seconds: 3),
)
```

## 📋 Properties (only relevant ones)

| Property | Purpose |
|---|---|
| `content` | Required widget containing the message |
| `duration` | How long the snack bar is shown |
| `action` | Optional labeled action, such as undo |
| `behavior` | Controls presentation style such as floating behavior |
| `backgroundColor` | Optional background color |
| `onVisible` | Callback invoked when the snack bar becomes visible |

## 💻 Complete Runnable Example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const SnackBarApp());

class SnackBarApp extends StatelessWidget {
  const SnackBarApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('SnackBar')),
        body: Builder(
          builder: (scaffoldContext) {
            return Center(
              child: ElevatedButton(
                onPressed: () {
                  ScaffoldMessenger.of(scaffoldContext).showSnackBar(
                    const SnackBar(
                      content: Text('Changes saved'),
                      duration: Duration(seconds: 3),
                    ),
                  );
                },
                child: const Text('Save'),
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

- `main` starts `SnackBarApp`.
- `MaterialApp` supplies the `ScaffoldMessenger` infrastructure.
- `Scaffold` creates the page and app bar.
- `Builder` creates a new context below the scaffold in the widget tree.
- `scaffoldContext` can therefore find the messenger associated with this page.
- `Center` places the save action in the body.
- `ElevatedButton` gives the user a clear action to trigger.
- `onPressed` looks up the messenger and calls `showSnackBar`.
- `SnackBar` contains a concise confirmation and stays visible for three seconds.
- `Text` supplies both the button label and the feedback message.

## 📱 ASCII Phone Screen Preview

```text
+----------------------+
| SnackBar             |
|----------------------|
|                      |
|        [Save]        |
|                      |
|  Changes saved       |
+----------------------+
```

## 🌳 Widget Tree Diagram

```text
SnackBarApp
└── MaterialApp
    └── Scaffold
        ├── AppBar
        └── Builder
            └── Center
                └── ElevatedButton
                    └── Text

ScaffoldMessenger (shown on button press)
└── SnackBar
    └── Text
```

## ⚙ Flutter Execution Flow

```text
User presses Save
        ↓
Button callback finds the page's ScaffoldMessenger
        ↓
showSnackBar queues and presents the message
        ↓
Message is dismissed or expires after its duration
```

## ⚠ Common Beginner Mistakes

- Calling `ScaffoldMessenger.of(context)` with a context above the `ScaffoldMessenger`.
- Showing a snack bar before an action has actually succeeded.
- Putting a long explanation in a message that disappears quickly.
- Using a snack bar for a critical decision that requires a dialog.
- Showing repeated messages too rapidly without considering how they queue.

## 🚀 Professional Usage

Use snack bars for short-lived confirmation, errors that do not require a blocking decision, or reversible actions. Place durable outcomes in the screen itself, and ensure any snack bar action is clearly labeled and genuinely works.

## 💡 Best Practices

- Keep the message brief and specific.
- Trigger success feedback only after success is confirmed.
- Use an action such as “Undo” only when the operation can truly be reversed.
- Choose a duration that gives users enough time to read and act.
- Use dialogs or inline errors for information that must not disappear unnoticed.

## 🧪 Try It Yourself

Add an `action` labeled `Undo` and implement a callback that displays a second confirmation. In a real app, connect it to an actual reversible operation.

## 🎯 Challenge

Create a list item with a delete action. Show a snack bar that offers “Undo,” and outline how the app would restore the item if the action is selected.

## ❓ 10 Practice Questions

1. What kind of feedback is a snack bar suited for?
2. Which object displays a snack bar?
3. Why does the example use `Builder`?
4. What does `duration` control?
5. What is the purpose of `action`?
6. When should a dialog be used instead?
7. Why must success feedback follow confirmed success?
8. What happens to a snack bar after its duration?
9. Which information should not exist only in a snack bar?
10. What is an appropriate use for an “Undo” action?

## 📝 Summary

`SnackBar` gives concise, temporary feedback through `ScaffoldMessenger`. Use it for non-blocking updates, show it from a context that can find the messenger, and reserve critical or durable information for more persistent UI.
