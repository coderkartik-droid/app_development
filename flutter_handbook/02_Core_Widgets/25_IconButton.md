# IconButton

## 📖 Introduction

`IconButton` is a Material button designed around an icon. It supplies interactive behavior and feedback for compact actions such as search, close, or opening a menu.

## 🤔 Why Do We Need It?

An icon can communicate a familiar action in limited space, but a plain `Icon` is only visual content. `IconButton` makes an icon actionable and provides button interaction behavior.

## 🧠 Internal Working

`IconButton` draws its icon inside a Material interaction target. It reacts to focus, hover, and presses according to its environment and style. The `onPressed` callback controls enabled state just like other Material buttons.

Because an icon alone may be ambiguous, provide a tooltip or semantic label when needed so assistive technology and users can understand the action.

## ⚙ Constructor (if applicable)

```dart
IconButton(
  onPressed: () {},
  tooltip: 'Search',
  icon: const Icon(Icons.search),
)
```

## 📋 Properties (only relevant ones)

| Property | Purpose |
|---|---|
| `icon` | The icon displayed by the button |
| `onPressed` | Action callback; `null` disables the button |
| `tooltip` | Short explanation shown on long press or hover and used for accessibility |
| `color` | Optional icon color |
| `iconSize` | Optional icon size |
| `style` | Optional button styling |

## 💻 Complete Runnable Example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const IconButtonApp());

class IconButtonApp extends StatelessWidget {
  const IconButtonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(
          title: const Text('IconButton'),
          actions: [
            Builder(
              builder: (scaffoldContext) {
                return IconButton(
                  tooltip: 'Search',
                  onPressed: () {
                    ScaffoldMessenger.of(scaffoldContext).showSnackBar(
                      const SnackBar(content: Text('Search selected')),
                    );
                  },
                  icon: const Icon(Icons.search),
                );
              },
            ),
          ],
        ),
        body: const Center(child: Text('Tap the search action above.')),
      ),
    );
  }
}
```

## 🔍 Line-by-Line Example Explanation

- `main` launches `IconButtonApp`.
- `MaterialApp` creates the Material app environment.
- `Scaffold` supplies the app bar and body.
- `AppBar.actions` places controls at the end of the app bar.
- `IconButton` wraps the icon in a standard interactive button.
- `tooltip` names the action for users who need more context and for accessibility.
- `onPressed` handles activation and shows confirmation.
- `icon` supplies the magnifying-glass symbol.
- The centered body text tells users where to find the sample action.

## 📱 ASCII Phone Screen Preview

```text
+----------------------+
| IconButton       [🔍]|
|----------------------|
|                      |
| Tap the search action|
| above.               |
|                      |
+----------------------+
```

## 🌳 Widget Tree Diagram

```text
IconButtonApp
└── MaterialApp
    └── Scaffold
        ├── AppBar
        │   ├── Text
        │   └── actions
        │       └── IconButton
        │           └── Icon
        └── Center
            └── Text
```

## ⚙ Flutter Execution Flow

```text
IconButton builds icon and interaction target
        ↓
Tooltip and semantic information identify the action
        ↓
User activates the control
        ↓
onPressed callback runs
```

## ⚠ Common Beginner Mistakes

- Using a bare `Icon` and expecting it to be tappable.
- Omitting a tooltip or semantic meaning for an unfamiliar icon.
- Making the touch target too small through custom styling.
- Choosing an icon whose meaning is unclear to the audience.
- Forgetting that `onPressed: null` disables the button.

## 🚀 Professional Usage

Use `IconButton` for familiar, compact actions, especially in app bars and toolbars. Keep the action discoverable, provide accessible naming, and ensure the touch target remains comfortable even when the icon artwork is small.

## 💡 Best Practices

- Include a concise tooltip for icon-only actions.
- Prefer widely understood symbols and consistent meanings.
- Implement a real callback.
- Keep sufficient contrast and touch area.
- Use a labeled button instead when the icon's meaning is not obvious.

## 🧪 Try It Yourself

Change the icon and tooltip to represent a close action. Update the body text and callback so all three consistently describe the new action.

## 🎯 Challenge

Add two icon actions to an app bar, such as search and help. Give each a meaningful tooltip and distinct callback behavior.

## ❓ 10 Practice Questions

1. What is the purpose of `IconButton`?
2. How is it different from `Icon`?
3. What does `onPressed` control?
4. Why is a tooltip useful?
5. Where are icon buttons commonly placed?
6. What should you do if an icon's meaning is unclear?
7. Why must the touch target be considered separately from icon size?
8. How is an icon button disabled?
9. What information should its callback provide?
10. How can icon-only actions support accessibility?

## 📝 Summary

`IconButton` turns an icon into a Material action control. Add a clear callback and accessible naming, and keep the action understandable and easy to activate.
