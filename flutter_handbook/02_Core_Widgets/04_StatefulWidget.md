# StatefulWidget

## 📖 Introduction

`StatefulWidget` is used when a widget owns data that can change while the widget is visible. The widget configuration remains immutable, while a separate `State` object stores mutable values.

## 🤔 Why Do We Need It?

Interactive interfaces need memory: a counter needs its number, a form needs entered values, and an animation needs progress. Stateful widgets provide that memory and a controlled way to request a rebuild.

## 🧠 Internal Working

Flutter creates a `State` object when the widget is inserted. Calling `setState` marks that state as needing a rebuild. Flutter then calls `build` again and updates the affected descendants. State can survive rebuilds because it belongs to the `State` object, not a temporary widget configuration.

## ⚙ Constructor (if applicable)

```dart
const Counter({super.key});
```

The widget constructor receives stable configuration. Mutable fields belong in the associated `State` class.

## 📋 Properties (only relevant ones)

| Member | Purpose |
|---|---|
| `createState` | Connects the widget to its state object |
| `setState` | Reports that mutable state changed |
| `mounted` | Indicates whether state is still in the tree |
| `initState` | Runs once when state is inserted |
| `dispose` | Releases controllers and subscriptions |

## 💻 Complete Runnable Example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const CounterApp());

class CounterApp extends StatelessWidget {
  const CounterApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(home: CounterPage());
  }
}

class CounterPage extends StatefulWidget {
  const CounterPage({super.key});

  @override
  State<CounterPage> createState() => _CounterPageState();
}

class _CounterPageState extends State<CounterPage> {
  int count = 0;

  void increment() {
    setState(() {
      count++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Counter')),
      body: Center(child: Text('Count: $count')),
      floatingActionButton: FloatingActionButton(
        onPressed: increment,
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

## 🔍 Line-by-Line Example Explanation

- `CounterPage` declares that its subtree owns changing state.
- `createState` creates `_CounterPageState`.
- `count` is mutable state and starts at zero.
- `increment` changes it inside `setState`.
- `setState` tells Flutter to call `build` again.
- The interpolated `count` displays the current value.
- The floating action button calls `increment` when tapped.

## 📱 ASCII Phone Screen Preview

```text
+----------------------+
| Counter              |
|----------------------|
|                      |
|       Count: 2       |
|                  (+) |
+----------------------+
```

## 🌳 Widget Tree Diagram

```text
CounterApp
└── MaterialApp
    └── CounterPage
        └── Scaffold
            ├── AppBar
            ├── Center → Text
            └── FloatingActionButton → Icon
```

## ⚙ Flutter Execution Flow

```text
createState → initial build → tap button → setState
            → build again → updated Text → repaint
```

## ⚠ Common Beginner Mistakes

- Changing state without calling `setState`.
- Calling `setState` after `dispose`.
- Creating controllers repeatedly inside `build`.
- Forgetting to dispose controllers or subscriptions.
- Using state when the value belongs higher in the application.

## 🚀 Professional Usage

Keep local ephemeral state close to the widget that owns it. For shared or long-lived application state, use an established state-management approach rather than passing unrelated values through many layers.

## 💡 Best Practices

- Keep `setState` callbacks small.
- Move expensive calculations out of `build`.
- Dispose resources in `dispose`.
- Lift state upward only when multiple widgets need it.

## 🧪 Try It Yourself

Add a reset button that sets `count` to zero inside `setState`.

## 🎯 Challenge

Build a stateful switch page that displays “Enabled” or “Disabled” and changes the label when a `Switch` is toggled.

## ❓ 10 Practice Questions

1. Where does mutable state live?
2. What does `createState` return?
3. Why is `setState` required?
4. What happens after `setState`?
5. When does `initState` run?
6. Why is `dispose` important?
7. What does `mounted` indicate?
8. Should controllers be created in `build`?
9. When should state be lifted?
10. What is immutable about a `StatefulWidget`?

## 📝 Summary

A stateful widget combines immutable configuration with a persistent `State` object. Change state inside `setState`, rebuild declaratively, and release resources when the state is disposed.
