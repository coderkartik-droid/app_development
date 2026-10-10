# Divider

## 📖 Introduction

`Divider` draws a thin horizontal line that separates content. It is commonly used between items in a vertical list or between sections.

## 🤔 Why Do We Need It?

When items are crowded together, it can be difficult to tell where one group ends and another begins. A subtle divider creates a visual boundary without introducing another large heading or card.

## 🧠 Internal Working

`Divider` is a thin box that paints a line across the width allowed by its parent. Its height includes the space reserved around the line, while `thickness` controls the visible line thickness. Parent constraints determine the final width.

For vertical separators, use `VerticalDivider` in an appropriate horizontal layout.

## ⚙ Constructor (if applicable)

```dart
const Divider(
  height: 1,
  thickness: 1,
  indent: 16,
  endIndent: 16,
)
```

## 📋 Properties (only relevant ones)

| Property | Purpose |
|---|---|
| `height` | Total vertical space occupied by the divider |
| `thickness` | Thickness of the painted line |
| `indent` | Empty space before the line begins |
| `endIndent` | Empty space after the line ends |
| `color` | Line color |

## 💻 Complete Runnable Example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const DividerApp());

class DividerApp extends StatelessWidget {
  const DividerApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('Divider')),
        body: ListView(
          children: const [
            ListTile(
              leading: Icon(Icons.inbox),
              title: Text('Inbox'),
            ),
            Divider(height: 1, indent: 16, endIndent: 16),
            ListTile(
              leading: Icon(Icons.archive),
              title: Text('Archive'),
            ),
          ],
        ),
      ),
    );
  }
}
```

## 🔍 Line-by-Line Example Explanation

- The import makes Material widgets available.
- `main` starts the app with `DividerApp`.
- `MaterialApp` and `Scaffold` provide the Material screen and body.
- `AppBar` supplies the screen title.
- `ListView` allows the rows to scroll when necessary.
- The first `ListTile` represents the inbox row.
- `Divider` separates the two rows and leaves 16 logical pixels clear at each end.
- The second `ListTile` represents the archive row.
- The list's children are constant, so the list can be declared `const`.

## 📱 ASCII Phone Screen Preview

```text
+----------------------+
| Divider              |
|----------------------|
|  [inbox] Inbox       |
|    ----------------  |
|  [box] Archive       |
+----------------------+
```

## 🌳 Widget Tree Diagram

```text
DividerApp
└── MaterialApp
    └── Scaffold
        ├── AppBar
        └── ListView
            ├── ListTile
            ├── Divider
            └── ListTile
```

## ⚙ Flutter Execution Flow

```text
ListView lays out each child in sequence
        ↓
Divider receives the list's available width
        ↓
Indent and endIndent reserve space at the ends
        ↓
The horizontal line is painted between rows
```

## ⚠ Common Beginner Mistakes

- Expecting `height` to be the line thickness; use `thickness` for the painted line.
- Assuming the divider will always span the screen; its parent controls its width.
- Using dividers between every element when spacing alone would be clearer.
- Using a horizontal divider where a `VerticalDivider` is needed.
- Choosing a low-contrast color that disappears against the surface.

## 🚀 Professional Usage

Use dividers sparingly to separate related rows or sections. Prefer the theme's default color unless the design system calls for a specific contrast. A divider is a visual cue and should not be the only way to convey semantic grouping.

## 💡 Best Practices

- Use a subtle line that remains visible on the current surface.
- Set `height` and `thickness` intentionally; they have different jobs.
- Align divider indents with the content it separates.
- Consider whitespace or headings as alternatives to repeated lines.
- Use `VerticalDivider` for vertical separation in a horizontal layout.

## 🧪 Try It Yourself

Change `indent` and `endIndent` to 0, then to 32. Observe how the line aligns with the row's content.

## 🎯 Challenge

Create a settings page with two clearly named groups and several rows in each. Use dividers only where they improve scanning, rather than between every piece of text.

## ❓ 10 Practice Questions

1. What does `Divider` paint?
2. What does `height` control?
3. What does `thickness` control?
4. What do `indent` and `endIndent` do?
5. Which parent constraint determines the divider's width?
6. Which widget is intended for a vertical separator?
7. Why should dividers be used with restraint?
8. Should line color have sufficient contrast?
9. Can a divider alone communicate semantic grouping?
10. What alternatives can separate sections?

## 📝 Summary

`Divider` is a thin horizontal visual separator. Its total occupied height differs from its line thickness, and its width comes from its parent. Use it as one of several tools for clear grouping.
