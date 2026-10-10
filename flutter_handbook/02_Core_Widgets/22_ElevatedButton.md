# ElevatedButton

## 📖 Introduction

`ElevatedButton` is a Material button with a filled surface and visual elevation. It is typically used for an important action that should stand out from surrounding content.

## 🤔 Why Do We Need It?

Users need to recognize which action is available and how to trigger it. A button supplies familiar interaction, touch behavior, focus handling, and visual feedback rather than making styled text pretend to be a control.

## 🧠 Internal Working

An `ElevatedButton` builds on Material's button interaction and styling system. Its `onPressed` callback determines whether it is enabled: a non-null callback enables it, and `null` disables it. Flutter provides interaction states such as pressed, hovered, and focused through the Material styling mechanism.

The button does not perform an action automatically. The application must implement its callback.

## ⚙ Constructor (if applicable)

```dart
ElevatedButton(
  onPressed: () {},
  child: const Text('Save'),
)
```

## 📋 Properties (only relevant ones)

| Property | Purpose |
|---|---|
| `onPressed` | Action callback; `null` disables the button |
| `child` | Usually the button label or a composed icon-and-label widget |
| `style` | Optional `ButtonStyle` for visual and sizing choices |
| `onLongPress` | Optional callback for a long press |
| `key` | Helps Flutter identify the widget in the tree |

## 💻 Complete Runnable Example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const ElevatedButtonApp());

class ElevatedButtonApp extends StatelessWidget {
  const ElevatedButtonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('ElevatedButton')),
        body: Builder(
          builder: (scaffoldContext) {
            return Center(
              child: ElevatedButton(
                onPressed: () {
                  ScaffoldMessenger.of(scaffoldContext).showSnackBar(
                    const SnackBar(content: Text('Changes saved')),
                  );
                },
                child: const Text('Save changes'),
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

- `main` starts the application.
- `MaterialApp` creates the Material environment needed by the button and messenger.
- `Scaffold` provides the app bar and page body.
- `Center` places the button in the middle of the body.
- `ElevatedButton` creates the interactive filled button.
- The non-null `onPressed` callback means the button is enabled.
- `ScaffoldMessenger.of(context)` finds the messenger associated with the surrounding app.
- `showSnackBar` gives visible confirmation after the button is pressed.
- `child` supplies the button's readable label.

## 📱 ASCII Phone Screen Preview

```text
+----------------------+
| ElevatedButton       |
|----------------------|
|                      |
|    +--------------+  |
|    | Save changes |  |
|    +--------------+  |
|                      |
+----------------------+
```

## 🌳 Widget Tree Diagram

```text
ElevatedButtonApp
└── MaterialApp
    └── Scaffold
        ├── AppBar
        └── Center
            └── ElevatedButton
                └── Text
```

## ⚙ Flutter Execution Flow

```text
Button is built with callback
        ↓
Material paints its enabled state and interaction feedback
        ↓
User presses the button
        ↓
onPressed runs and ScaffoldMessenger displays confirmation
```

## ⚠ Common Beginner Mistakes

- Leaving `onPressed` empty in real application code, so the visible control does nothing.
- Setting `onPressed: null` while expecting the button to remain enabled.
- Using the strongest visual button style for every action.
- Using an unclear label such as “OK” when a specific action can be named.
- Disabling a button without explaining why it is unavailable.

## 🚀 Professional Usage

Use `ElevatedButton` for a prominent action in a page or form. Keep one clear primary action where possible, and use less prominent button variants for secondary actions. Provide meaningful completion or error feedback after the callback runs.

## 💡 Best Practices

- Use action-oriented, specific labels.
- Implement the callback and handle the real operation.
- Use `null` only when the button should truly be disabled.
- Preserve focus, touch, and accessibility behavior by using a button widget.
- Keep a consistent button hierarchy through app-wide styles or themes.

## 🧪 Try It Yourself

Change the callback to `null` and observe the disabled appearance. Then restore it and change the label to a specific action relevant to a form.

## 🎯 Challenge

Create a two-action form footer with one primary elevated action and one secondary action. Ensure each has a distinct, working callback and clear labels.

## ❓ 10 Practice Questions

1. What visual treatment does `ElevatedButton` provide?
2. What does a non-null `onPressed` mean?
3. How is a button disabled?
4. Does Flutter decide what the action should do?
5. What kind of content usually belongs in `child`?
6. When should this variant be used?
7. Why should not every button be elevated?
8. What is a useful alternative to a vague action label?
9. Why is a real button preferable to clickable styled text?
10. What feedback might follow a successful action?

## 📝 Summary

`ElevatedButton` is a prominent Material action control. Its callback determines whether it is enabled, but the app must implement the action. Choose it for important actions and use clear labels and meaningful feedback.
