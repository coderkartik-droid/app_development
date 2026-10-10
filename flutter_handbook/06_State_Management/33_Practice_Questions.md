# Practice Questions on Flutter State Management

## 📖 Introduction

This chapter is designed for revision, interviews, and self-checking.

State management is not only about syntax. It is about understanding:
- where state lives
- how it flows
- when it updates
- how widgets react to it
- why one pattern works better than another

These questions are meant to sharpen your thinking and prepare you for coding interviews, project architecture decisions, and debugging sessions.

---

## 🤔 Why Do We Need It?

A strong developer is not just someone who can copy code. A strong developer can explain:

- why a widget rebuilds
- why a state object is shared
- when a view should listen to state
- how to avoid stale UI
- which pattern matches the app problem

Practice questions help you move from "I saw this in code" to "I understand why this works."

---

## 🌍 Real-Life Analogy

Imagine a hospital system.

- The doctor sees patient data.
- The nurse updates the record.
- The system must update all relevant screens instantly.
- If the wrong state is used, the wrong decision could be made.

State management in Flutter is similar. Bad state management causes wrong UI, wrong timing, and unstable behavior.

---

## 🧠 Internal Working

The main mental model is:

```text
UI Event -> State Change -> Rebuild -> Updated Screen
```

Different patterns use different steps:

- `setState`: direct local update and rebuild
- `Cubit`: method call -> emit new state
- `Bloc`: event -> handler -> emit new state
- `Provider`: data delivered through the widget tree

Understanding the flow helps you reason about bugs faster.

---

## ⚙ Syntax (if applicable)

Here are reusable patterns you should recognize:

```dart
setState(() {
  isSelected = !isSelected;
});
```

```dart
class CounterCubit extends Cubit<int> {
  CounterCubit() : super(0);

  void increment() => emit(state + 1);
}
```

```dart
class AuthBloc extends Bloc<AuthEvent, AuthState> {
  AuthBloc() : super(AuthInitial()) {
    on<LoginSubmitted>((event, emit) async {
      emit(AuthLoading());
      await Future.delayed(const Duration(seconds: 1));
      emit(AuthSuccess());
    });
  }
}
```

If you understand these patterns, you understand the main architecture of Flutter state management.

---

## 📋 Important Methods / Properties

| Pattern | Core idea |
| :--- | :--- |
| `setState` | rebuild current widget |
| `notifyListeners` | notify listeners of a change |
| `emit` | publish a new state |
| `state` | current value |
| `BlocBuilder` | rebuild UI using bloc state |
| `Consumer` | build widget using provider value |
| `Selector` | rebuild only when selected data changes |

These are the most common vocabulary terms you need in interviews and practical work.

---

## 💻 Complete Runnable Example

```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';

void main() {
  runApp(const App());
}

class CounterCubit extends Cubit<int> {
  CounterCubit() : super(0);

  void increment() => emit(state + 1);
  void decrement() => emit(state - 1);
}

class App extends StatelessWidget {
  const App({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: BlocProvider(
        create: (_) => CounterCubit(),
        child: const CounterView(),
      ),
    );
  }
}

class CounterView extends StatelessWidget {
  const CounterView({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Practice Counter')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            BlocBuilder<CounterCubit, int>(
              builder: (context, value) {
                return Text('Value: $value', style: const TextStyle(fontSize: 32));
              },
            ),
            const SizedBox(height: 20),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                ElevatedButton(
                  onPressed: () => context.read<CounterCubit>().decrement(),
                  child: const Text('-'),
                ),
                const SizedBox(width: 12),
                ElevatedButton(
                  onPressed: () => context.read<CounterCubit>().increment(),
                  child: const Text('+'),
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

This example is a compact review of the whole idea: state is held externally, widgets listen, and the UI updates when the state changes.

---

## 🔍 Line-by-Line Example Explanation

### `class CounterCubit extends Cubit<int>`
This creates a state manager for integer values.

### `CounterCubit() : super(0);`
The inital state is 0.

### `void increment() => emit(state + 1);`
The event is a method call, and the output is a new state.

### `BlocProvider(create: (_) => CounterCubit())`
This makes the cubit available to the subtree.

### `BlocBuilder<CounterCubit, int>`
The UI listens when the state changes and rebuilds.

### `context.read<CounterCubit>()`
This retrieves the cubit from the provider tree.

This is the exact flow you need to understand for interviews and real app work.

---

## 📱 ASCII Phone Screen Preview

```text
┌─────────────────────────────┐
authored by : practice       │
│                             │
│       Practice Counter      │
│                             │
│          Value: 4           │
│                             │
│            [-]   [+]        │
│                             │
│                             │
└─────────────────────────────┘
```

---

## 🌳 Widget Tree

```text
MaterialApp
└── BlocProvider
    └── CounterView
        └── Scaffold
            ├── AppBar
            └── Center
                └── Column
                    ├── BlocBuilder
                    └── Row
                        ├── ElevatedButton(-)
                        └── ElevatedButton(+)
```

---

## ⚙ Flutter Execution Flow

```text
Tap +
   |
   v
Context reads cubit
   |
   v
Cubit emits next value
   |
   v
BlocBuilder rebuilds UI
   |
   v
Text shows updated number
```

This flow is the core concept behind almost every Flutter state pattern.

---

## ⚠ Common Beginner Mistakes

- Confusing local state with shared state
- Forgetting to choose the right tool for the problem
- Overengineering with `Bloc` for simple screens
- Rebuilding too many widgets unnecessarily
- Not separating business logic from view logic

---

## 🚀 Professional Usage

In professional development, these questions are not optional. Real teams ask them every day:

- Where does the state belong?
- Who should update it?
- Which widgets should rebuild?
- How do we keep the UI predictable?
- How do we test the logic?

Strong architecture is built on the answers to these questions.

---

## 💡 Best Practices

- Always identify the ownership of state.
- Keep state changes predictable.
- Use the simplest pattern that fits the feature.
- Avoid mixing too many state systems in one app.
- Test state transition logic separately from UI details.

---

## 🧪 Try It Yourself

Write your own answers to the following questions before looking at the solution:

1. What is the difference between local and shared state?
2. Why does a widget rebuild after `emit` is called?
3. Which pattern is better for a login workflow with loading/error states?
4. Why do we avoid mutating state directly in many apps?

---

## 🎯 Challenge

Create a 1-page decision sheet for a Flutter app with these features:

- theme switch
- todo list
- login form
- API fetch with loading states
- settings page
- chat screen

For each item, specify:
- local or shared state
- simple or complex logic
- recommended tool
- reason

---

## ❓ Practice Questions (Minimum 10)

1. What is state in Flutter?
2. Why do widgets need to rebuild when state changes?
3. What is the difference between `setState` and `Cubit`?
4. Why is `Bloc` useful for complex flows?
5. How does `Provider` help a widget tree?
6. When should you use `Selector` instead of a broad rebuild?
7. What is the purpose of `notifyListeners()`?
8. Why is a single-source-of-truth design helpful?
9. What is the difference between a widget owning state and a separate state manager owning state?
10. Why is immutability useful in state management?
11. Which system would you prefer for a login screen with loading, success, and error states?
12. Why should business logic not be directly inside UI widgets?
13. What does `emit(newState)` mean in a cubit?
14. How does `BlocBuilder` know when to rebuild?
15. What is one real-world reason for choosing `Riverpod` over simpler state tools?

---

## 📝 Summary

State management is one of the most important concepts in Flutter because it controls how UI and logic stay in sync.

The most important pattern to remember is:

> The state should be owned by the right layer, and the UI should react when that state changes.

Whether you use `setState`, `Cubit`, `Bloc`, `Provider`, or `Riverpod`, the deeper principle remains the same:

- state belongs somewhere meaningful
- updates are explicit
- the UI reacts predictably
- the app remains maintainable

If you can reason through this, you can reason through nearly any Flutter app architecture problem.
