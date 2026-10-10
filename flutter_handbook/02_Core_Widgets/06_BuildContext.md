# BuildContext

## 📖 Introduction

`BuildContext` is a handle to a widget's location in the widget tree. It lets code find information provided by ancestors, such as themes, media size, localization, and navigators.

## 🤔 Why Do We Need It?

A widget often needs information without receiving every value manually. Context connects a widget to the nearest relevant ancestor and allows Flutter APIs to work with the correct part of the tree.

## 🧠 Internal Working

The context is associated with an element, not a global screen coordinate. Two contexts on the same screen can see different ancestors. A context is valid only while its widget is mounted, so asynchronous code must check `mounted` before using it afterward.

## ⚙ Constructor (if applicable)

You do not construct `BuildContext` yourself. Flutter supplies it to `build`, callbacks, and builder functions:

```dart
Widget build(BuildContext context) { ... }
```

## 📋 Properties (only relevant ones)

| API | Purpose |
|---|---|
| `Theme.of(context)` | Finds the nearest theme |
| `MediaQuery.sizeOf(context)` | Reads available size |
| `ScaffoldMessenger.of(context)` | Finds a snackbar host |
| `Navigator.of(context)` | Finds navigation state |
| `context.mounted` | Checks that context is still active |

## 💻 Complete Runnable Example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const ContextApp());

class ContextApp extends StatelessWidget {
  const ContextApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('BuildContext')),
        body: const ContextInfo(),
      ),
    );
  }
}

class ContextInfo extends StatelessWidget {
  const ContextInfo({super.key});

  @override
  Widget build(BuildContext context) {
    final size = MediaQuery.sizeOf(context);
    final color = Theme.of(context).colorScheme.primary;
    return Center(
      child: Text('Width: ${size.width.toStringAsFixed(0)}\nColor: $color'),
    );
  }
}
```

## 🔍 Line-by-Line Example Explanation

- Flutter supplies `context` to `ContextInfo.build`.
- `MediaQuery.sizeOf` finds screen information above this widget.
- `Theme.of` finds the nearest `Theme`.
- `Center` positions the result.
- The string displays values read from the current tree location.

## 📱 ASCII Phone Screen Preview

```text
+----------------------+
| BuildContext         |
|----------------------|
| Width: 390           |
| Color: blue          |
+----------------------+
```

Values vary by device and theme.

## 🌳 Widget Tree Diagram

```text
ContextApp
└── MaterialApp
    └── Scaffold
        └── ContextInfo
            └── Center → Text
```

`MaterialApp` supplies inherited services that descendants can find through context.

## ⚙ Flutter Execution Flow

```text
build receives context → lookup nearest ancestor → use inherited value
                        → return widgets → render
```

## ⚠ Common Beginner Mistakes

- Storing a context and using it after its widget is removed.
- Using a context above the ancestor you want to find.
- Calling `ScaffoldMessenger.of` before a messenger exists in that branch.
- Using context after `await` without checking `mounted`.
- Treating context as a global application object.

## 🚀 Professional Usage

Use context for tree-scoped dependencies and navigation. Keep APIs explicit when a value is business data rather than inherited UI configuration.

## 💡 Best Practices

- Use the closest appropriate context.
- Prefer `MediaQuery.sizeOf` when only size is needed.
- Check `context.mounted` after asynchronous gaps.
- Avoid passing context deep into non-UI services.

## 🧪 Try It Yourself

Display `MediaQuery.sizeOf(context).height` below the width.

## 🎯 Challenge

Add a button that shows a `SnackBar` with `ScaffoldMessenger.of(context)`.

## ❓ 10 Practice Questions

1. What does `BuildContext` represent?
2. Who supplies it?
3. Why can two contexts see different ancestors?
4. How do you read a theme?
5. How do you check whether context is mounted?
6. Why should context not be stored globally?
7. What does `Navigator.of` find?
8. What can happen if context is used after `await`?
9. Where should business services receive data?
10. Why does tree position matter?

## 📝 Summary

`BuildContext` identifies a widget's tree location and enables scoped lookups. Use the nearest valid context, respect its lifetime, and avoid treating it as global state.
