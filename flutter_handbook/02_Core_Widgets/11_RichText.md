# RichText

## 📖 Introduction

`RichText` displays one paragraph containing multiple `TextSpan` objects, allowing different portions of a sentence to use different styles or gesture recognizers.

## 🤔 Why Do We Need It?

A single `Text` style applies broadly. Rich text is useful for emphasis, inline links, legal copy, highlighted search results, or mixed typography without splitting a sentence into awkward separate widgets.

## 🧠 Internal Working

`RichText` builds one paragraph from a span tree. A parent span can provide inherited style, while child spans override selected properties. Gesture recognizers must be managed carefully when they are attached to spans.

## ⚙ Constructor (if applicable)

```dart
RichText(
  text: const TextSpan(
    text: 'Hello ',
    children: <TextSpan>[TextSpan(text: 'world')],
  ),
)
```

## 📋 Properties (only relevant ones)

| Property | Purpose |
|---|---|
| `text` | Root `InlineSpan` |
| `style` | Default paragraph style |
| `textAlign` | Paragraph alignment |
| `maxLines` | Maximum lines |
| `overflow` | Overflow behavior |
| `textScaler` | Text scaling behavior |

## 💻 Complete Runnable Example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const RichTextApp());

class RichTextApp extends StatelessWidget {
  const RichTextApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('RichText')),
        body: Center(
          child: RichText(
            textAlign: TextAlign.center,
            text: TextSpan(
              style: DefaultTextStyle.of(context).style,
              children: const <TextSpan>[
                TextSpan(text: 'Read the '),
                TextSpan(
                  text: 'important',
                  style: TextStyle(fontWeight: FontWeight.bold),
                ),
                TextSpan(text: ' message.'),
              ],
            ),
          ),
        ),
      ),
    );
  }
}
```

## 🔍 Line-by-Line Example Explanation

- `RichText` receives one root span tree.
- `DefaultTextStyle.of(context).style` supplies a readable inherited base style.
- The first span contains normal text.
- The second span overrides weight to bold.
- The final span completes the sentence.
- `textAlign` centers the complete paragraph.

## 📱 ASCII Phone Screen Preview

```text
+----------------------+
| RichText             |
|----------------------|
| Read the IMPORTANT   |
|      message.        |
+----------------------+
```

## 🌳 Widget Tree Diagram

```text
Scaffold
├── AppBar → Text
└── Center
    └── RichText
        └── TextSpan
            ├── TextSpan
            ├── TextSpan (bold)
            └── TextSpan
```

## ⚙ Flutter Execution Flow

```text
span tree → paragraph layout → inherited/override styles applied
          → wrapping and alignment → paint text runs
```

## ⚠ Common Beginner Mistakes

- Forgetting a readable root style.
- Using rich spans only to place unrelated blocks vertically.
- Styling a link without a visible focus or semantic indication.
- Creating gesture recognizers without disposing them.
- Making contrast depend only on color.

## 🚀 Professional Usage

Use `RichText` for genuinely inline content. For tappable links, consider `Text.rich` or a well-designed accessible link component, and ensure keyboard and screen-reader behavior is supported.

## 💡 Best Practices

- Keep span trees understandable.
- Use theme typography as the base.
- Provide non-color cues for emphasis or links.
- Dispose long-lived `GestureRecognizer` objects.
- Localize complete sentences where possible.

## 🧪 Try It Yourself

Add an italic span and change the root alignment to left.

## 🎯 Challenge

Create a sentence with a bold product name, an italic version number, and a visibly underlined “Learn more” span.

## ❓ 10 Practice Questions

1. What does RichText display?
2. What is a TextSpan?
3. How does a child span differ from its parent?
4. What does the root style provide?
5. When is RichText preferable to several Text widgets?
6. Why should links not rely only on color?
7. What resource may need disposal?
8. What does `textAlign` affect?
9. Why should complete sentences be localized?
10. What accessibility concerns apply to rich text?

## 📝 Summary

`RichText` lays out one paragraph made of styled spans. Use it for inline variation, preserve readability and accessibility, and manage gesture resources responsibly.
