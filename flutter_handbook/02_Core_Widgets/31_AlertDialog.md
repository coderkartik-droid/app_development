# AlertDialog

## 📖 Introduction

`AlertDialog` is a Material dialog that temporarily appears above the current screen to present important information or ask the user to make a decision.

## 🤔 Why Do We Need It?

Some choices need the user's attention before the app continues, such as confirming a destructive action or choosing whether to discard edits. A dialog brings that decision into focus. For routine success messages, a less interruptive `SnackBar` is usually more appropriate.

## 🧠 Internal Working

`AlertDialog` describes the dialog's content and actions, but it must be presented in the navigation overlay, commonly using `showDialog`. That function returns a `Future` which completes with the value supplied to `Navigator.pop`. A `BuildContext` from inside the dialog builder can be used to close the dialog.

The dialog does not decide what a selection means. The calling code interprets the returned result and performs the chosen action.

## ⚙ Constructor (if applicable)

```dart
AlertDialog(
  title: const Text('Discard changes?'),
  content: const Text('Your unsaved edits will be lost.'),
  actions: [
    TextButton(onPressed: () {}, child: const Text('Keep editing')),
    TextButton(onPressed: () {}, child: const Text('Discard')),
  ],
)
```

## 📋 Properties (only relevant ones)

| Property | Purpose |
|---|---|
| `title` | Short heading describing the decision |
| `content` | Explanation or other main dialog content |
| `actions` | Widgets for the user's available choices |
| `icon` | Optional visual context for the dialog |
| `actionsAlignment` | Alignment of the action widgets |
| `scrollable` | Allows the title and content area to scroll when needed |

## 💻 Complete Runnable Example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const AlertDialogApp());

class AlertDialogApp extends StatelessWidget {
  const AlertDialogApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('AlertDialog')),
        body: Builder(
          builder: (pageContext) {
            return Center(
              child: ElevatedButton(
                onPressed: () async {
                  final shouldDelete = await showDialog<bool>(
                    context: pageContext,
                    builder: (dialogContext) {
                      return AlertDialog(
                        title: const Text('Delete note?'),
                        content: const Text(
                          'This example asks for confirmation before deleting.',
                        ),
                        actions: [
                          TextButton(
                            onPressed: () => Navigator.pop(dialogContext, false),
                            child: const Text('Cancel'),
                          ),
                          TextButton(
                            onPressed: () => Navigator.pop(dialogContext, true),
                            child: const Text('Delete'),
                          ),
                        ],
                      );
                    },
                  );

                  if (!pageContext.mounted || shouldDelete != true) return;
                  ScaffoldMessenger.of(pageContext).showSnackBar(
                    const SnackBar(content: Text('Delete confirmed')),
                  );
                },
                child: const Text('Delete note'),
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

- `main` starts the application with `AlertDialogApp`.
- `MaterialApp` supplies the Material app and navigation environment.
- `Scaffold` creates the page and its app bar.
- `Builder` provides `pageContext`, which is below the scaffold and can access its messenger.
- The `ElevatedButton` starts the confirmation flow.
- `showDialog<bool>` presents a dialog whose result can be a boolean.
- `dialogContext` is local to the dialog builder and is used to close that dialog.
- `AlertDialog` displays a short title, explanatory content, and two choices.
- The Cancel action pops the dialog with `false`; Delete pops it with `true`.
- `await` pauses the callback until the dialog closes.
- `pageContext.mounted` ensures the page still exists before it is used after the asynchronous wait.
- The confirmation message is shown only when the result is `true`.

## 📱 ASCII Phone Screen Preview

```text
+----------------------+
| AlertDialog          |
|----------------------|
|      +----------+    |
|      | Delete   |    |
|      | note?    |    |
|      | This asks|    |
|      | before   |    |
|      | deleting.|    |
|      | Cancel  Delete|
|      +----------+    |
+----------------------+
```

## 🌳 Widget Tree Diagram

```text
AlertDialogApp
└── MaterialApp
    └── Scaffold
        ├── AppBar
        └── Builder
            └── Center
                └── ElevatedButton

Overlay (when the dialog is open)
└── AlertDialog
    ├── title: Text
    ├── content: Text
    └── actions
        ├── TextButton (Cancel)
        └── TextButton (Delete)
```

## ⚙ Flutter Execution Flow

```text
User starts an action
        ↓
showDialog adds the dialog to the navigation overlay
        ↓
User chooses an action; Navigator.pop returns a result
        ↓
The awaiting callback checks the result and continues if appropriate
```

## ⚠ Common Beginner Mistakes

- Showing a dialog for trivial feedback that would be better as a `SnackBar`.
- Using vague action labels such as “OK” when a specific decision is needed.
- Performing an action before checking the dialog result.
- Using a page context after an asynchronous gap without checking that it is still mounted.
- Forgetting that dialogs can become cramped with long content or large text.

## 🚀 Professional Usage

Use an alert dialog when the user must notice and resolve a focused question. Explain consequences plainly, make each action distinct, and handle the returned result in the calling flow. For lengthy forms or complex workflows, navigate to a dedicated screen instead.

## 💡 Best Practices

- Keep the title and content concise.
- Label actions with clear outcomes, especially for destructive choices.
- Do not default the user into a destructive action.
- Respect large text and allow long content to scroll.
- Use the dialog result rather than relying on hidden mutable state.
- Check a context's mounted state after awaiting before using it.

## 🧪 Try It Yourself

Add a third choice that closes the dialog with `null`. Verify that the page shows its confirmation only when the result is `true`.

## 🎯 Challenge

Create a confirmation dialog for discarding an edited draft. Return a result for both choices and ensure the draft is discarded only after the user confirms.

## ❓ 10 Practice Questions

1. What kind of interaction is an `AlertDialog` suited for?
2. Which function commonly presents an `AlertDialog`?
3. What does the `showDialog` future return?
4. Why does the builder receive a `dialogContext`?
5. How does `Navigator.pop` return a result?
6. Why should the caller inspect the result?
7. When is a `SnackBar` a better choice?
8. Why should destructive actions be labeled specifically?
9. What does checking `mounted` protect against after an `await`?
10. When might a separate screen be better than a dialog?

## 📝 Summary

`AlertDialog` presents a focused decision above the current screen. Use `showDialog` to display it, return a clear result when the user chooses, and perform the requested action only after checking that result.
