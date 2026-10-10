# When to Use Which in Flutter State Management

## 📖 Introduction

Flutter gives you many ways to manage state. That is both a gift and a challenge.

Each approach has a different strength:

- `setState` is best for tiny UI state
- `InheritedWidget` is useful for dependency flow
- `Provider` is flexible and common
- `Riverpod` is safer and more scalable
- `Bloc` is ideal for complex business logic
- `Cubit` is a lighter BLoC alternative
- `ChangeNotifier` is straightforward for simple observable models

The real problem is not that Flutter lacks state management tools. The real problem is choosing the right one.

This chapter helps you answer the question every developer asks:

> "Which state management solution should I use for this app?"

The answer is not one-size-fits-all. It depends on app size, complexity, testing needs, team structure, and long-term maintainability.

---

## 🤔 Why Do We Need It?

A bad choice creates pain later:

- UI becomes hard to reason about
- logic is spread across many widgets
- tests become challenging
- code becomes difficult to change
- bug fixes create regressions

The right tool reduces cognitive overload.

### Decision Rule

Use the simplest tool that solves your problem.

A lot of developer confusion disappears when we stop asking:

> "What is the best state management package?"

and instead ask:

> "What level of complexity do I need to manage?"

---

## 🌍 Real-Life Analogy

Choosing a state manager is like choosing transport.

- Need to commute a few blocks? Use a bicycle.
- Need to move a family across the city? Use a car.
- Need to carry heavy industrial goods across the country? Use a truck.

Each option is valid, but not equivalent.

Similarly:

- `setState` is the bicycle
- `Provider` is the car
- `Bloc` is the industrial logistics system

The correct choice depends on the task, not personal preference.

---

## 🧠 Internal Working

All state management solutions try to solve the same problem:

> Keep UI consistent with application data.

But they differ in how they spread responsibilities.

### 1. Local UI state
A widget owns data used only inside itself.

Example:
- current tab index
- dropdown selection
- expand/collapse flag

This is best solved with `setState`.

### 2. Shared app state
Several widgets need access to the same data.

Example:
- user profile
- shopping cart
- theme setting

This is where `Provider`, `InheritedWidget`, or `Riverpod` fit better.

### 3. Business logic with input-output rules
A feature may require validation, network calls, event sequences, and predictable transitions.

Example:
- login flow
- checkout process
- order lifecycle
- multi-step form

This is where `Bloc` or `Cubit` becomes the right answer.

---

## ⚙ Syntax (if applicable)

Here is the same logic represented across different tools:

### `setState`
```dart
setState(() {
  isExpanded = !isExpanded;
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
class LoginBloc extends Bloc<LoginEvent, LoginState> {
  LoginBloc() : super(LoginInitial()) {
    on<LoginSubmitted>((event, emit) async {
      emit(LoginLoading());
      await Future.delayed(const Duration(seconds: 1));
      emit(LoginSuccess());
    });
  }
}
```

The more complexity you need, the more structured the architecture becomes.

---

## 📋 Important Methods / Properties

| Tool | Best for | Complexity | Typical use |
| :--- | :--- | :--- | :--- |
| `setState` | Small local UI state | Low | toggles, forms, small screens |
| `InheritedWidget` | Data sharing down the tree | Medium | custom framework logic |
| `ChangeNotifier` | Observable models | Medium | simple app-level services |
| `Provider` | App state and dependency injection | Medium | common production apps |
| `Cubit` | State transitions | Medium | feature state, forms, loading workflows |
| `Bloc` | Event-driven business logic | High | large apps, validation, transactions |
| `Riverpod` | Scalable app architecture | Medium-High | modern, test-friendly apps |

### Quick comparison

| Need | Best choice |
| :--- | :--- |
| Single widget local toggle | `setState` |
| Shared config across tree | `InheritedWidget` or `Provider` |
| Lightweight state with methods | `Cubit` |
| Full event-driven business flow | `Bloc` |
| Modern scalable architecture | `Riverpod` |

---

## 💻 Complete Runnable Example

This example shows a simple decision table in code-like form:

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const StateSelectionDemo());
}

class StateSelectionDemo extends StatelessWidget {
  const StateSelectionDemo({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('Choose the Right Tool')),
        body: const Padding(
          padding: EdgeInsets.all(20),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Text('Use setState for local widget state.',
                  style: TextStyle(fontSize: 20)),
              SizedBox(height: 12),
              Text('Use Cubit for feature-level state changes.',
                  style: TextStyle(fontSize: 20)),
              SizedBox(height: 12),
              Text('Use Bloc for complex business workflows.',
                  style: TextStyle(fontSize: 20)),
              SizedBox(height: 12),
              Text('Use Riverpod for scalable architecture.',
                  style: TextStyle(fontSize: 20)),
            ],
          ),
        ),
      ),
    );
  }
}
```

This code is not a full implementation of every state system, but it demonstrates the core idea: choose the state solution according to the problem’s complexity.

---

## 🔍 Line-by-Line Example Explanation

### `StateSelectionDemo extends StatelessWidget`
This is a simple info screen demonstrating a decision model.

### `Text('Use setState for local widget state.')`
This tells the developer the simplest tool for simple UI interactions.

### `Text('Use Cubit for feature-level state changes.')`
This highlights when a state-focused architecture is enough.

### `Text('Use Bloc for complex business workflows.')`
This shows that event-based architecture is useful when logic becomes richer.

This practical explanation matters because state management is not just syntax. It is a design decision.

---

## 📱 ASCII Phone Screen Preview

```text
┌─────────────────────────────┐
authored by : judgment       │
│                             │
│ Choose the Right Tool        │
│                             │
│ setState for local UI        │
│ Cubit for feature state      │
│ Bloc for complex logic       │
│ Riverpod for scalable apps   │
│                             │
│                             │
└─────────────────────────────┘
```

---

## 🌳 Widget Tree

```text
MaterialApp
└── Scaffold
    ├── AppBar
    └── Padding
        └── Column
            ├── Text("Use setState...")
            ├── Text("Use Cubit...")
            ├── Text("Use Bloc...")
            └── Text("Use Riverpod...")
```

---

## ⚙ Flutter Execution Flow

```text
App loads
   |
   v
Widget tree builds
   |
   v
Text instructions render
   |
   v
Developer chooses solution based on complexity
```

In real projects, this decision is made before code is written. The right architecture reduces maintenance cost.

---

## ⚠ Common Beginner Mistakes

### 1. Using `Bloc` for everything
This creates unnecessary boilerplate.

### 2. Using `setState` for app-wide state
This causes widget coupling and difficult maintenance.

### 3. Choosing a tool because it is trending
The best tool is the one that matches your problem.

### 4. Ignoring testability
If a solution is hard to test, it is often too complicated.

### 5. Overengineering early
Start simple, then evolve the architecture as the app grows.

---

## 🚀 Professional Usage

Professional developers usually follow this flow:

1. Start with `setState` for local widget UI
2. Move to `Provider` or `Cubit` for shared screen state
3. Use `Bloc` when business logic becomes complex
4. Consider `Riverpod` when scalability and architecture become more important

This is not dogma. It is strategy.

A good engineering team does not chase the fanciest architecture. It chooses the most maintainable structure for the current needs.

---

## 💡 Best Practices

- Prefer the simplest tool that solves the problem.
- Keep local widget logic local.
- Move complex logic into a state manager.
- Separate UI from business rules.
- Reevaluate architecture when the app grows.
- Learn the patterns deeply before mixing many solutions.

---

## 🧪 Try It Yourself

Write a short checklist for a project:

- If state is only used by one widget, use `setState`.
- If multiple widgets need it, use `Provider` or `Cubit`.
- If the feature has validation, data fetching, and transitions, use `Bloc`.
- If the app is large and the team values modularity, consider `Riverpod`.

Then compare your decision with a real project you have seen or built.

---

## 🎯 Challenge

Create a mini decision matrix for the following app features:

- theme toggle
- login form
- checkout process
- shopping cart
- profile update form
- real-time stock dashboard

For each feature, decide which tool is best and explain why.

This task trains you to think architecturally, not just syntactically.

---

## ❓ Practice Questions (Minimum 10)

1. When is `setState` the best choice?
2. Why is `Bloc` not always the best choice for small apps?
3. Which state manager is best for shared app data across many widgets?
4. Why is `Cubit` often a middle-ground option?
5. What is the main benefit of `Riverpod` in larger apps?
6. When would you choose `Bloc` over `Cubit`?
7. What is the risk of overengineering early?
8. Why do professional teams prefer simple solutions first?
9. Which tool is best for a login feature with loading and error states?
10. Why is architecture choice not just a matter of personal preference?
11. What should you do if state is only used inside a single widget?
12. How does app complexity influence the choice of state management solution?

---

## 📝 Summary

There is no single "correct" state management tool for every Flutter app. The real skill is choosing the right tool for the right level of complexity.

Use this rule:

- `setState` for local UI state
- `Cubit` for screen-level state transitions
- `Bloc` for event-driven business logic
- `Provider` or `Riverpod` for shared dependencies and scalable architecture

The best state management is not the most advanced one. It is the one that keeps the app readable, testable, and maintainable.
