# Build Method

## 📖 Introduction

The `build` method describes the widget subtree Flutter should display. It returns a `Widget` and receives a `BuildContext`, which identifies the widget's location in the tree.

## 🤔 Why Do We Need It?

A declarative method is easier to reason about than manually changing individual views. You describe what the UI should look like for current inputs; Flutter handles updating the screen.

## 🧠 Internal Working

Flutter may call `build` many times, including after parent updates, `setState`, theme changes, or inherited data changes. The method should be deterministic for its inputs. It should construct widgets, not perform one-time setup or network work.

## ⚙ Constructor (if applicable)

`build` is not a constructor. Its required signature is:

```dart
@override
Widget build(BuildContext context) {
  return const Text('Hello');
}
```

## 📋 Properties (only relevant ones)

| Item | Meaning |
|---|---|
| Return value | The subtree to display |
| `context` | Tree location and inherited access |
| `@override` | Confirms the framework method is implemented |

## 💻 Complete Runnable Example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const BuildApp());

class BuildApp extends StatelessWidget {
  const BuildApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('Build Method')),
        body: const BuildMessage(),
      ),
    );
  }
}

class BuildMessage extends StatelessWidget {
  const BuildMessage({super.key});

  @override
  Widget build(BuildContext context) {
    final theme = Theme.of(context);
    return Center(
      child: Text('Built with ${theme.colorScheme.primary} theme color'),
    );
  }
}
```

## 🔍 Line-by-Line Example Explanation

- `BuildApp.build` returns the app's widget tree.
- `BuildMessage.build` receives its location as `context`.
- `Theme.of(context)` reads the nearest inherited theme.
- `Center` positions the resulting message.
- `Text` uses the value calculated from current inherited data.

## 📱 ASCII Phone Screen Preview

```text
+----------------------+
| Build Method         |
|----------------------|
| Built with blue      |
| theme color          |
+----------------------+
```

The exact color name is theme-dependent.

## 🌳 Widget Tree Diagram

```text
BuildApp
└── MaterialApp
    └── Scaffold
        ├── AppBar → Text
        └── BuildMessage
            └── Center → Text
```

## ⚙ Flutter Execution Flow

```text
framework schedules build → build(context) returns widgets
                           → widgets are reconciled → layout and paint
```

## ⚠ Common Beginner Mistakes

- Performing API calls on every build.
- Calling `setState` synchronously from `build`.
- Returning `null` instead of a widget.
- Creating unnecessary controllers in the method.
- Assuming build runs only once.

## 🚀 Professional Usage

Keep `build` short by extracting meaningful sections into widgets or helper methods. Use it as a pure description of current UI state and inherited configuration.

## 💡 Best Practices

- Avoid side effects in `build`.
- Use `const` where possible.
- Read context-dependent values at the appropriate tree location.
- Profile before optimizing rebuilds.

## 🧪 Try It Yourself

Change the returned message and add a `TextStyle` using `theme.textTheme.headlineSmall`.

## 🎯 Challenge

Create a `Greeting` widget whose `build` method reads `Theme.of(context)` and displays text using the theme's headline style.

## ❓ 10 Practice Questions

1. What does `build` return?
2. Why can it run repeatedly?
3. What is a build side effect?
4. What does `context` provide?
5. Why should network calls not be in build?
6. What does `@override` mean?
7. Can build return a primitive string?
8. How can a long build method be simplified?
9. When can `const` help?
10. What triggers a rebuild?

## 📝 Summary

`build` declaratively returns a widget subtree. Keep it fast, repeatable, and free of side effects because Flutter may call it whenever the UI's inputs change.
