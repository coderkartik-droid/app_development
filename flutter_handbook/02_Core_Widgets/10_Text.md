# Text

## 📖 Introduction

`Text` displays a string using Flutter's text layout and painting system. It is the basic widget for headings, labels, messages, and readable content.

## 🤔 Why Do We Need It?

Text communicates information and actions. A dedicated widget handles font metrics, wrapping, alignment, overflow, direction, and accessibility more reliably than manually drawing characters.

## 🧠 Internal Working

Flutter uses a paragraph layout engine to measure glyphs, apply style, wrap lines, and paint them. The available width comes from the parent constraints. A `Text` widget may occupy multiple lines when wrapping is allowed.

## ⚙ Constructor (if applicable)

```dart
Text(
  'Hello',
  style: const TextStyle(fontSize: 20),
  textAlign: TextAlign.center,
)
```

## 📋 Properties (only relevant ones)

| Property | Purpose |
|---|---|
| `data` | String displayed |
| `style` | Font and decoration |
| `textAlign` | Horizontal alignment |
| `maxLines` | Maximum line count |
| `overflow` | Behavior when text does not fit |
| `softWrap` | Whether wrapping is allowed |
| `textDirection` | Reading direction when needed |

## 💻 Complete Runnable Example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const TextApp());

class TextApp extends StatelessWidget {
  const TextApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('Text')),
        body: const Center(
          child: Padding(
            padding: EdgeInsets.all(24),
            child: Text(
              'Readable text adapts to the available space.',
              textAlign: TextAlign.center,
              style: TextStyle(fontSize: 22, fontWeight: FontWeight.bold),
            ),
          ),
        ),
      ),
    );
  }
}
```

## 🔍 Line-by-Line Example Explanation

- The app bar uses a short `Text` title.
- `Center` positions the content.
- `Padding` keeps it away from screen edges.
- The message is a constant string.
- `textAlign` centers lines within the text box.
- `TextStyle` increases size and weight.

## 📱 ASCII Phone Screen Preview

```text
+----------------------+
| Text                 |
|----------------------|
|  Readable text adapts|
|  to the available    |
|       space.         |
+----------------------+
```

## 🌳 Widget Tree Diagram

```text
Scaffold
├── AppBar → Text
└── Center
    └── Padding
        └── Text
```

## ⚙ Flutter Execution Flow

```text
Text receives string and constraints → measures glyphs → wraps lines
                                     → applies style → paints paragraph
```

## ⚠ Common Beginner Mistakes

- Assuming `textAlign` positions the widget itself.
- Omitting overflow handling in constrained rows.
- Hard-coding text that should be localized.
- Using tiny text or low-contrast colors.
- Forgetting semantics when text is visually hidden.

## 🚀 Professional Usage

Use theme text styles for consistent hierarchy and localization-ready strings. Design for larger accessibility text and test translated strings that may be longer.

## 💡 Best Practices

- Prefer theme styles over repeated literal styles.
- Set meaningful `maxLines` and `overflow` where space is limited.
- Keep user-facing strings localizable.
- Maintain readable contrast and line length.

## 🧪 Try It Yourself

Add `maxLines: 2` and `overflow: TextOverflow.ellipsis`.

## 🎯 Challenge

Create a responsive message that displays a title and subtitle with different theme text styles.

## ❓ 10 Practice Questions

1. What does Text display?
2. What controls font appearance?
3. What does `textAlign` align?
4. Why can text wrap?
5. What does `maxLines` limit?
6. What is ellipsis overflow?
7. Why localize strings?
8. Why use theme styles?
9. What determines available width?
10. How should accessibility text be tested?

## 📝 Summary

`Text` handles readable string layout. Use styles, constraints, overflow, localization, and accessibility considerations to make messages robust.
