# Cubit in Flutter

## 📖 Introduction

A `Cubit` is a lighter and more focused version of BLoC. If BLoC is a full event-driven architecture, `Cubit` is the streamlined version designed for one simple idea: "I have state, and I want to update it with a method."

Instead of moving through events and handlers, a `Cubit` exposes normal methods like `increment()`, `loadUsers()`, or `toggleTheme()`, and every method ends by emitting a new state.

This makes `Cubit` a favorite choice for:
- simple screen state
- button toggles
- forms with validation
- asynchronous loading states
- moderate app complexity without full BLoC boilerplate

The main idea is simple:

- `Cubit` holds a current state
- you call a method
- the method computes the next state
- `emit(newState)` updates the UI

This chapter teaches you when to use `Cubit`, how it works internally, and how to build a complete app state flow with minimal ceremony.

---

## 🤔 Why Do We Need It?

Flutter apps usually need more than a single boolean or counter. Screens often need:

- loading state
- success state
- error state
- empty state
- filtered state
- user interaction state

With plain `setState`, this can become chaotic because logic and UI are mixed together. With `Cubit`, the logic is isolated.

### `setState` vs `Cubit`

```dart
// setState approach
setState(() {
  isLoading = true;
});

// Cubit approach
cubit.loadData();
```

The second example is cleaner because:
- the UI does not know the business logic
- the state transition is explicit
- the state can be tested easily
- rebuild logic is centralized

### Why not just use `ChangeNotifier`?

`ChangeNotifier` is useful, but it is more mutable and less structured. `Cubit` enforces a cleaner state emission pattern:

- `Cubit` is state-based
- state is replaced, not mutated in place
- each update is predictable
- state changes are easier to test and debug

That is why `Cubit` is often chosen for medium-sized apps and feature modules.

---

## 🌍 Real-Life Analogy

Think of a `Cubit` as a smart assistant in a restaurant kitchen.

- The waiter says: "We need the order status updated."
- The kitchen assistant knows the current state: `idle`, `loading`, `success`, or `error`.
- It receives a command such as `prepareOrder()`.
- It decides the next state and sends it out.

The restaurant does not need every single action to be expressed as a huge event list. It just needs to know the next state of a process.

A `Cubit` is like a manager with a clear, narrow job:

> "I manage the state of this screen or feature. When a user action happens, I compute the next state and publish it."

---

## 🧠 Internal Working

A `Cubit` is built on the idea of a single current state and a stream of updated states.

Internally, it behaves like this:

```dart
class CounterCubit extends Cubit<int> {
  CounterCubit() : super(0);

  void increment() => emit(state + 1);
}
```

### What is happening here?

1. `CounterCubit()` calls `super(0)`, which sets the initial state to `0`.
2. `state` is the current value.
3. `increment()` calculates `state + 1`.
4. `emit(state + 1)` publishes the new state.
5. Any listening widget rebuilds when the new state arrives.

### Important concept: `emit` is the state update trigger

Unlike `setState`, a `Cubit` does not mutate the old object in place. It emits a new state, which makes updates more predictable.

This pattern is valuable because:

- no silent bugs
- easier debugging
- no stale UI state
- simpler testing

The `Cubit` lifecycle is:

```
Initial state
     |
     v
method called
     |
     v
new state computed
     |
     v
emit(newState)
     |
     v
UI rebuilds
```

---

## ⚙ Syntax

The syntax of `Cubit` is very compact:

```dart
import 'package:flutter_bloc/flutter_bloc.dart';

class CounterCubit extends Cubit<int> {
  CounterCubit() : super(0);

  void increment() => emit(state + 1);

  void decrement() => emit(state - 1);

  void reset() => emit(0);
}
```

### Explanation

- `extends Cubit<int>` means this cubit manages an integer state.
- `super(0)` sets the initial value to 0.
- `emit(state + 1)` sends a new value to listeners.
- The widgets listen to the cubit state and rebuild automatically.

You can also use complex states:

```dart
class LoginState {
  final String email;
  final bool isLoading;
  final String? error;

  LoginState({
    required this.email,
    required this.isLoading,
    this.error,
  });
}
```

A `Cubit` can manage any immutable state object, not just numbers or strings.

---

## 📋 Important Methods / Properties

| Member | Purpose |
| :--- | :--- |
| `state` | Current state of the cubit |
| `emit(newState)` | Sends a new state to listeners |
| `stream` | A stream of all emitted states |
| `listen(...)` | Register a listener manually |
| `close()` | Release resources and close stream |
| `onChange(change)` | Called whenever state changes |
| `onError(error, stacktrace)` | Handles errors during emissions |

### Deep understanding

#### `state`
This is the last value currently stored in the cubit.

#### `emit(newState)`
This is the real trigger. Once emitted, all listeners receive the new state.

#### `stream`
The cubit acts like a stream of states. You can inspect it or use it for debugging.

#### `onChange`
This is especially useful for logs and debugging:

```dart
@override
void onChange(Change<int> change) {
  print('Current: ${change.currentValue} -> Next: ${change.nextValue}');
  super.onChange(change);
}
```

This tells you exactly how the state changed.

---

## 💻 Complete Runnable Example

This example creates a simple counter screen using `Cubit`.

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
  void reset() => emit(0);
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: BlocProvider(
        create: (_) => CounterCubit(),
        child: const CounterScreen(),
      ),
    );
  }
}

class CounterScreen extends StatelessWidget {
  const CounterScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Cubit Counter'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Text(
              'Counter value:',
              style: TextStyle(fontSize: 24),
            ),
            const SizedBox(height: 16),
            BlocBuilder<CounterCubit, int>(
              builder: (context, count) {
                return Text(
                  '$count',
                  style: const TextStyle(
                    fontSize: 48,
                    fontWeight: FontWeight.bold,
                  ),
                );
              },
            ),
            const SizedBox(height: 24),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                FloatingActionButton(
                  onPressed: () => context.read<CounterCubit>().decrement(),
                  child: const Icon(Icons.remove),
                ),
                const SizedBox(width: 16),
                FloatingActionButton(
                  onPressed: () => context.read<CounterCubit>().increment(),
                  child: const Icon(Icons.add),
                ),
              ],
            ),
            const SizedBox(height: 20),
            TextButton(
              onPressed: () => context.read<CounterCubit>().reset(),
              child: const Text('Reset'),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## 🔍 Line-by-Line Example Explanation

### `class CounterCubit extends Cubit<int>`
This defines a cubit whose state is an integer.

### `CounterCubit() : super(0);`
The initial state is `0`.

### `void increment() => emit(state + 1);`
When user presses the + button, the cubit emits the next value.

### `BlocProvider(create: (_) => CounterCubit())`
This creates the cubit and makes it accessible to the subtree.

### `BlocBuilder<CounterCubit, int>`
This listens to state changes and rebuilds the widget whenever the integer value changes.

### `context.read<CounterCubit>()`
This reads the current cubit instance from the widget tree.

### `Text('$count')`
This displays the latest state value on the screen.

The key idea is simple: the UI never mutates local state directly. It dispatches an action to the cubit, which emits a new state.

---

## 📱 ASCII Phone Screen Preview

```text
┌─────────────────────────────┐
authored by : cubit         │
│                            │
│      Cubit Counter          │
│                            │
│        Counter value:       │
│            3                │
│                            │
│        [-]    [+]            │
│                            │
│         Reset               │
│                            │
│                            │
└─────────────────────────────┘
```

---

## 🌳 Widget Tree

```text
MaterialApp
└── BlocProvider
    └── CounterScreen
        ├── AppBar
        └── Column
            ├── Text
            ├── BlocBuilder
            │   └── Text('count')
            ├── Row
            │   ├── FloatingActionButton (-)
            │   └── FloatingActionButton (+)
            └── TextButton (Reset)
```

---

## ⚙ Flutter Execution Flow

```text
User taps '+'
     |
     v
CounterScreen calls context.read<CounterCubit>().increment()
     |
     v
Cubit method runs
     |
     v
emit(state + 1)
     |
     v
BlocBuilder listens and rebuilds widget
     |
     v
New value appears on UI
```

The important part is that Flutter rebuilds only where necessary. The state is centralized, and the widget listens to it.

---

## ⚠ Common Beginner Mistakes

### 1. Emitting without changing state
If you emit the same state, the widget may not rebuild depending on equality logic.

### 2. Keeping logic inside the widget
A widget should not contain business logic. Use a cubit instead.

### 3. Forgetting to close the cubit
In long-lived app states, you should close the cubit when it is no longer needed.

### 4. Using too much state in one cubit
If the cubit manages many unrelated things, it becomes difficult to maintain.

### 5. Not using immutable state objects
For large state models, immutable classes help avoid accidental mutation issues.

---

## 🚀 Professional Usage

Use `Cubit` when:
- the screen is moderate in complexity
- there are fewer state transitions
- state changes are still predictable
- you want cleaner code than `setState` but less ceremony than BLoC

Examples:
- login form state
- theme switcher
- cart item count
- search filter state
- user profile editor

`Cubit` is often a perfect choice for a single feature module that does not require event-driven complexity.

---

## 💡 Best Practices

- Keep each cubit focused on one responsibility.
- Use immutable state objects for larger models.
- Prefer `emit` instead of mutating fields in the widget.
- Use `BlocListener` when you need side effects such as navigation or snackbars.
- Keep state transitions easy to understand.
- Use `onChange` only for debugging or analytics.

---

## 🧪 Try It Yourself

Create a `ThemeCubit` that manages a boolean `isDarkMode` state.

Requirements:
- initial state = `false`
- method `toggleTheme()` emits `true` or `false`
- build a button that toggles the theme
- update the app background color accordingly

This will help you understand how a cubit manages a single state type with simple methods.

---

## 🎯 Challenge

Build a `TodoCubit` that manages this state:

```dart
class TodoState {
  final List<String> todos;
  final bool isLoading;

  TodoState({required this.todos, this.isLoading = false});
}
```

Add methods:
- `addTodo(String item)`
- `removeTodo(int index)`
- `toggleLoading(bool value)`

Then connect it to a widget that displays the list and allows adding tasks.

This challenge will teach you how a cubit handles more complex state than a simple integer.

---

## ❓ Practice Questions (Minimum 10)

1. What is the main difference between `setState` and `Cubit`?
2. Why is `emit()` important in a cubit?
3. What does `super(0)` do in a cubit constructor?
4. How does `BlocBuilder` react to a cubit state change?
5. When should you prefer `Cubit` over BLoC?
6. What is the role of the `state` property in a cubit?
7. Why is `Cubit` considered easier than full BLoC for feature modules?
8. What happens if you emit the same state repeatedly?
9. Why should a cubit have a single responsibility?
10. What is the difference between a widget-level state and cubit state?
11. Can a cubit manage a complex object instead of a primitive value?
12. Explain the flow from button tap to UI rebuild in a cubit-based app.

---

## 📝 Summary

`Cubit` is the simplest state manager in the Flutter ecosystem when you want clear, predictable state updates without the full ceremony of BLoC.

It works by:
- storing current state
- exposing methods to change state
- calling `emit(newState)`
- notifying widgets automatically

Use `Cubit` when your app needs clean, feature-focused state management with minimal complexity. It gives you structure, testability, and better separation of logic from UI without the overhead of event-driven architecture.

If your app is small to medium-sized, a `Cubit` is often the ideal balance between simplicity and professionalism.
