# Quick Revision: State Management in Flutter

## 📖 Introduction

State management is the heart of Flutter app architecture.

Every app keeps some data in memory, and that data changes over time. The challenge is to keep the UI in sync with that data without writing messy code.

This quick revision chapter collects the most important ideas in one place:

- what state is
- when to use local state or app state
- what `setState` does
- how `Cubit`, `Bloc`, and `Provider` fit together
- how to choose the right approach

This is your compact survival guide for Flutter state management.

---

## 🤔 Why Do We Need It?

Without a clear state strategy, Flutter apps become difficult to maintain.

Common problems:
- widgets rebuild too much
- logic is scattered across components
- state is duplicated
- user actions cause inconsistent UI
- debugging becomes painful

A proper state strategy creates structure, predictability, and easier testing.

---

## 🌍 Real-Life Analogy

Think of a state manager as the manager of a busy office.

- A task is assigned to a widget.
- The widget needs the latest information.
- The manager delivers the current status and updates it when things change.

If the office has no manager, people keep guessing the latest facts.

Flutter state management is the same idea: it keeps everyone aligned with the latest information.

---

## 🧠 Internal Working

At the core, Flutter state management is about one question:

> "Who owns the data, and who listens to it?"

### Local state
The widget owns it and updates itself.

```dart
setState(() {
  count = count + 1;
});
```

### Shared state
A separate object owns the data. Multiple widgets depend on it.

```dart
class CounterCubit extends Cubit<int> {
  CounterCubit() : super(0);

  void increment() => emit(state + 1);
}
```

### Event-driven state
The app turns user actions into events and processes them into a final state.

```dart
on<LoginSubmitted>((event, emit) async {
  emit(LoginLoading());
  final result = await repo.login();
  emit(LoginSuccess(result));
});
```

This pattern scales better as an app becomes more complex.

---

## ⚙ Syntax

### `setState`
```dart
setState(() {
  isOpen = !isOpen;
});
```

### `Cubit`
```dart
class ThemeCubit extends Cubit<bool> {
  ThemeCubit() : super(false);

  void toggle() => emit(!state);
}
```

### `Bloc`
```dart
class CounterBloc extends Bloc<CounterEvent, int> {
  CounterBloc() : super(0) {
    on<Increment>((event, emit) => emit(state + 1));
  }
}
```

### `Provider`
```dart
ChangeNotifierProvider(
  create: (_) => CounterModel(),
  child: MyWidget(),
);
```

These are not competing in a vacuum. They solve different layers of the same challenge.

---

## 📋 Important Methods / Properties

| Concept | Meaning |
| :--- | :--- |
| `state` | the current value in the state manager |
| `setState` | rebuild the current widget when local state changes |
| `emit` | send a new state from a cubit |
| `notifyListeners` | tell listeners a `ChangeNotifier` updated |
| `BlocBuilder` | rebuild UI from bloc state |
| `Consumer` | read provider state in a widget |
| `Provider` | inject dependencies and shared values |

### Memory rule
The longer the app lives, the more important it becomes to separate state from UI.

---

## 💻 Complete Runnable Example

```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';

void main() {
  runApp(const MyApp());
}

class CounterCubit extends Cubit<int> {
  CounterCubit() : super(0);

  void increment() => emit(state + 1);
  void decrement() => emit(state - 1);
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: BlocProvider(
        create: (_) => CounterCubit(),
        child: const CounterPage(),
      ),
    );
  }
}

class CounterPage extends StatelessWidget {
  const CounterPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Quick Revision')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            BlocBuilder<CounterCubit, int>(
              builder: (context, value) => Text('$value', style: const TextStyle(fontSize: 40)),
            ),
            const SizedBox(height: 20),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                FloatingActionButton(
                  onPressed: () => context.read<CounterCubit>().decrement(),
                  child: const Icon(Icons.remove),
                ),
                const SizedBox(width: 20),
                FloatingActionButton(
                  onPressed: () => context.read<CounterCubit>().increment(),
                  child: const Icon(Icons.add),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}
```

This is a clean example of state management in action. The cubit owns the state, and the UI listens to it.

---

## 🔍 Line-by-Line Example Explanation

### `class CounterCubit extends Cubit<int>`
This says the cubit holds an integer state.

### `CounterCubit() : super(0);`
The initial value is `0`.

### `void increment() => emit(state + 1);`
When the user taps +, the cubit emits a new state.

### `BlocProvider(create: (_) => CounterCubit())`
This injects the cubit into the widget tree.

### `BlocBuilder<CounterCubit, int>`
This listens to state changes and rebuilds the view.

### `context.read<CounterCubit>()`
This reads the cubit from the current context.

This pattern is the foundation of clean, predictable Flutter state management.

---

## 📱 ASCII Phone Screen Preview

```text
┌────────────────────────────┐
authored by : revision       │
│                            │
│       Quick Revision        │
│                            │
│             3              │
│                            │
│        [-]     [+]          │
│                            │
│                            │
└────────────────────────────┘
```

---

## 🌳 Widget Tree

```text
MaterialApp
└── BlocProvider
    └── CounterPage
        └── Scaffold
            ├── AppBar
            └── Center
                └── Column
                    ├── BlocBuilder
                    └── Row
                        ├── FloatingActionButton(-)
                        └── FloatingActionButton(+)
```

---

## ⚙ Flutter Execution Flow

```text
User taps +
   |
   v
Button calls increment()
   |
   v
Cubit emits new state
   |
   v
BlocBuilder rebuilds UI
   |
   v
New value visible on screen
```

That is the base lifecycle of most modern Flutter state management patterns.

---

## ⚠ Common Beginner Mistakes

- Using `setState` for app-wide state
- Forgetting to keep logic separate from UI
- Mixing local and shared state in the same widget
- Overusing `Bloc` for every app
- Not testing state transitions

---

## 🚀 Professional Usage

Professional Flutter teams usually follow a layered approach:

- `setState` for tiny local UI behavior
- `Cubit` for feature-specific state
- `Bloc` for validated, complex flows
- `Provider` or `Riverpod` for shared and scalable state

This system scales very well because it keeps responsibilities clear.

---

## 💡 Best Practices

- Keep state close to where it is needed.
- Avoid storing all app data in one giant widget.
- Separate business rules from UI rendering.
- Use immutable state where possible.
- Choose the simplest pattern that solves the problem.

---

## 🧪 Try It Yourself

Answer these in one sentence each:

1. What is the difference between local and shared state?
2. Which pattern is best for a single-widget toggle?
3. Which pattern is best for a login workflow with errors and loading?
4. Why is emitting state better than mutating raw values in place?

---

## 🎯 Challenge

Build a quick comparison chart for these app scenarios:

- theme switch
- cart count
- login page
- API data fetch
- chat screen
- multi-step form

For each one, write the likely best choice and why.

---

## ❓ Practice Questions (Minimum 10)

1. What is state in Flutter?
2. When should `setState` be used?
3. What is the main role of a cubit?
4. What does `emit` do?
5. What is the difference between `Cubit` and `Bloc`?
6. Why is `Provider` useful?
7. When do you use `Riverpod` instead of `Provider`?
8. Why should business logic not live inside a widget?
9. What is the value of immutable state?
10. How does `BlocBuilder` help in a Flutter app?
11. What happens when the state changes in a cubit?
12. Name one reason why a large app must use structured state management.

---

## 📝 Summary

State management is not Optional. It is the mechanism that keeps a Flutter interface consistent, predictable, and maintainable.

The most important ideas are:

- local state is fast and simple
- shared state needs structure
- `Cubit` is lightweight and state-focused
- `Bloc` is event-driven and more architectural
- `Provider` and `Riverpod` help with dependency and app-level state

If you remember just one rule, remember this:

> Use the simplest state management solution that matches the level of your app's complexity.
