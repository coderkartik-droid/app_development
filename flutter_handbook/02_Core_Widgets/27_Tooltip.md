# Tooltip

## 📖 Introduction

`Tooltip` displays a short explanatory message associated with a child. It can appear after a user hovers over or presses and holds the child, helping explain controls whose purpose may not be obvious.

## 🤔 Why Do We Need It?

An icon-only control can save space but may be ambiguous. A tooltip adds context without permanently taking up room. It can also contribute an accessible label, though important instructions should not be hidden only in a tooltip.

## 🧠 Internal Working

`Tooltip` wraps one child and associates it with a message. Flutter's interaction system decides when to show the message for supported input modes such as hover and touch. A tooltip is transient; it is not a replacement for visible text when the information is essential.

## ⚙ Constructor (if applicable)

```dart
Tooltip(
  message: 'Search',
  child: IconButton(
    onPressed: () {},
    icon: const Icon(Icons.search),
  ),
)
```

Material controls such as `IconButton` also offer a `tooltip` property, which is often the simpler choice for a single control.

## 📋 Properties (only relevant ones)

| Property | Purpose |
|---|---|
| `message` | Required short text shown in the tooltip |
| `child` | The single widget associated with the message |
| `preferBelow` | Preferred placement relative to the child |
| `waitDuration` | Delay before showing on hover |
| `showDuration` | How long the tooltip remains visible |
| `key` | Helps Flutter identify the widget in the tree |

## 💻 Complete Runnable Example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const TooltipApp());

class TooltipApp extends StatelessWidget {
  const TooltipApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('Tooltip')),
        body: const Center(
          child: Tooltip(
            message: 'Search for a place',
            child: Icon(Icons.search, size: 40),
          ),
        ),
      ),
    );
  }
}
```

## 🔍 Line-by-Line Example Explanation

- `main` starts `TooltipApp`.
- `MaterialApp` and `Scaffold` create a Material page.
- `AppBar` identifies the screen.
- `Center` places the tooltip's child in the body.
- `Tooltip` associates a short message with its child.
- `message` describes what the search symbol means.
- `Icon` is the one child wrapped by the tooltip.
- In a real interface, use a button as the child when the icon performs an action; this example focuses only on showing the tooltip behavior.

## 📱 ASCII Phone Screen Preview

```text
+----------------------+
| Tooltip              |
|----------------------|
|                      |
|     Search for a     |
|       place          |
|         (🔍)         |
|                      |
+----------------------+
```

## 🌳 Widget Tree Diagram

```text
TooltipApp
└── MaterialApp
    └── Scaffold
        ├── AppBar
        └── Center
            └── Tooltip
                └── Icon
```

## ⚙ Flutter Execution Flow

```text
Tooltip wraps its child and message
        ↓
Input hovers over or presses and holds the target
        ↓
Tooltip timing and placement are applied
        ↓
Short explanatory message appears temporarily
```

## ⚠ Common Beginner Mistakes

- Putting essential instructions only in a tooltip.
- Writing a long paragraph where a short label is expected.
- Wrapping decorative content when no explanation is useful.
- Using `Tooltip` around a non-interactive icon when the actual UI needs a button.
- Assuming every input method triggers tooltips in exactly the same way.

## 🚀 Professional Usage

Use tooltips to clarify compact controls, especially icon-only actions. Keep messages brief and complementary to visible UI. For interactive controls, prefer built-in tooltip properties when available so semantics and behavior remain together.

## 💡 Best Practices

- Explain the action in a few words.
- Keep necessary information visible without requiring a tooltip.
- Use an interactive widget for an actionable child.
- Avoid redundant tooltips on controls whose label is already clear.
- Check tooltip behavior on touch, keyboard, and pointer-based devices.

## 🧪 Try It Yourself

Wrap an `IconButton` in `Tooltip` and compare it with setting `tooltip` directly on `IconButton`. Prefer the more direct version for that case.

## 🎯 Challenge

Create a toolbar with three icon-only actions. Add concise explanations, make each action a real button, and check that the messages accurately describe their callbacks.

## ❓ 10 Practice Questions

1. What does `Tooltip` associate with a child?
2. What is a tooltip useful for?
3. Why should essential instructions not exist only in tooltips?
4. What is the required `message`?
5. How many children does a `Tooltip` directly wrap?
6. What does `waitDuration` affect?
7. When can a control's built-in tooltip property be simpler?
8. Should an actionable icon be a plain `Icon`?
9. Why should tooltip text be short?
10. Which input types should tooltip behavior be checked against?

## 📝 Summary

`Tooltip` provides brief, temporary context for a child, often an icon control. Keep it concise, do not hide essential information in it, and use a real interactive control when the child represents an action.
