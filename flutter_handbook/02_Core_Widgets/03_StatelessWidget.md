# StatelessWidget

## 📖 Introduction

`StatelessWidget` is the right starting point for a widget whose output depends only on its constructor values and information available while building. It does not own mutable state that changes during its lifetime.

## 🤔 Why Do We Need It?

Many UI pieces are naturally descriptive: a logo, welcome message, or reusable card. A stateless widget keeps those pieces simple and predictable. It can still rebuild when its parent supplies new values.

## 🧠 Internal Working

Flutter may call `build` whenever the widget is inserted or its configuration changes. The widget object is immutable. Rebuilding creates a new description of its descendants; it does not mean every rendered object is necessarily discarded.

## ⚙ Constructor (if applicable)

```dart
const Greeting({super.key, required this.name});
```

Use `required` for values the widget cannot work without, and make the constructor `const` when possible.

## 📋 Properties (only relevant ones)

| Property | Meaning |
|---|---|
| `key` | Helps Flutter identify this widget |
| Custom `final` fields | Immutable input used by `build` |
| `context` in `build` | Location and inherited information |

## 💻 Complete Runnable Example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const StatelessApp());

class StatelessApp extends StatelessWidget {
  const StatelessApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(home: GreetingPage(name: 'Asha'));
  }
}

class GreetingPage extends StatelessWidget {
  final String name;

  const GreetingPage({super.key, required this.name});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: const AppBar(title: Text('Stateless Widget')),
      body: Center(child: Text('Welcome, $name!')),
    );
  }
}
```

## 🔍 Line-by-Line Example Explanation

- `StatelessApp` is the root stateless widget.
- Its `build` method returns a constant `MaterialApp`.
- `GreetingPage` receives `name` through a `final` field.
- `required this.name` ensures callers provide that value.
- `Scaffold` supplies a page and app bar.
- String interpolation inserts the immutable input into the greeting.

## 📱 ASCII Phone Screen Preview

```text
+----------------------+
| Stateless Widget     |
|----------------------|
|                      |
|     Welcome, Asha!   |
|                      |
+----------------------+
```

## 🌳 Widget Tree Diagram

```text
StatelessApp
└── MaterialApp
    └── GreetingPage
        └── Scaffold
            ├── AppBar
            └── Center
                └── Text
```

## ⚙ Flutter Execution Flow

```text
constructor values → build(context) → returned descendants
                   → layout → paint
```

When `name` changes, the parent supplies a new `GreetingPage` configuration and Flutter rebuilds the affected branch.

## ⚠ Common Beginner Mistakes

- Declaring changing fields without `final`.
- Expecting a stateless widget to update itself with `setState`.
- Doing expensive work on every `build` call.
- Forgetting `required` for essential inputs.
- Mutating a list passed to a widget instead of providing a new value.

## 🚀 Professional Usage

Use stateless widgets for presentation, design-system components, and pure sections of a screen. Keeping them free of lifecycle side effects improves testing and reuse.

## 💡 Best Practices

- Prefer `const` constructors.
- Store inputs in `final` fields.
- Keep `build` declarative and fast.
- Give custom widgets focused names and responsibilities.

## 🧪 Try It Yourself

Add a `subtitle` field and display it below the greeting with a `Column`.

## 🎯 Challenge

Create a reusable `StatusLabel` widget with required `label` and `color` fields, then use it twice with different values.

## ❓ 10 Practice Questions

1. What kind of widget is stateless?
2. Why are custom fields usually `final`?
3. Can a stateless widget rebuild?
4. What does `required` do?
5. What is the purpose of `key`?
6. Where should display input be stored?
7. Why should `build` remain fast?
8. Can a stateless widget receive changing data?
9. What replaces `setState` for this widget?
10. When is a stateless widget preferable to a stateful one?

## 📝 Summary

`StatelessWidget` describes UI from immutable inputs. It is simple, reusable, and ideal for presentation that does not own changing local state.
