# Scaffold

## 📖 Introduction

`Scaffold` provides the standard visual structure of a Material page. It gives named slots for an app bar, body, floating action button, drawer, bottom navigation, and transient messages.

## 🤔 Why Do We Need It?

Page chrome follows familiar patterns. `Scaffold` lets you place each part in the correct role instead of manually stacking every surface and handling insets yourself.

## 🧠 Internal Working

`Scaffold` lays out its slots according to available constraints and coordinates features such as snack bars, drawers, and floating action button placement. It is intended for a page, not as a replacement for the entire app root.

## ⚙ Constructor (if applicable)

```dart
Scaffold(
  appBar: AppBar(...),
  body: ...,
  floatingActionButton: ...,
)
```

## 📋 Properties (only relevant ones)

| Property | Purpose |
|---|---|
| `appBar` | Top page bar |
| `body` | Main content |
| `floatingActionButton` | Primary floating action |
| `drawer` | Side navigation drawer |
| `bottomNavigationBar` | Bottom navigation |
| `bottomSheet` | Persistent bottom sheet |
| `backgroundColor` | Page background |

## 💻 Complete Runnable Example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const ScaffoldApp());

class ScaffoldApp extends StatelessWidget {
  const ScaffoldApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('Scaffold')),
        body: const Center(child: Text('Main page content')),
        floatingActionButton: FloatingActionButton(
          onPressed: null,
          child: Icon(Icons.add),
        ),
      ),
    );
  }
}
```

## 🔍 Line-by-Line Example Explanation

- The app returns a `Scaffold` as its home page.
- `appBar` fills the top slot.
- `body` receives the main content area.
- `Center` positions the message.
- `floatingActionButton` places an action above the lower edge.
- `onPressed: null` intentionally makes this demonstration button disabled.

## 📱 ASCII Phone Screen Preview

```text
+----------------------+
| Scaffold             |
|----------------------|
|                      |
|   Main page content  |
|                  (+) |
+----------------------+
```

## 🌳 Widget Tree Diagram

```text
Scaffold
├── AppBar → Text
├── Center → Text
└── FloatingActionButton → Icon
```

## ⚙ Flutter Execution Flow

```text
Scaffold receives slots → computes page layout → positions body/chrome
                         → coordinates overlays → paints page
```

## ⚠ Common Beginner Mistakes

- Putting a `Scaffold` inside every small component.
- Forgetting that `body` receives finite page constraints.
- Placing a snack bar outside a scaffold messenger.
- Using a drawer without an appropriate app bar/menu affordance.
- Expecting `Scaffold` to provide app-wide navigation automatically.

## 🚀 Professional Usage

Use one `Scaffold` per route-level page. Keep reusable content independent of it so the same component can appear in dialogs, sheets, or other screens.

## 💡 Best Practices

- Use the named slots for their intended roles.
- Keep page content in separate widgets.
- Use `SafeArea` when content needs explicit system-edge protection.
- Give floating actions a clear primary purpose.

## 🧪 Try It Yourself

Add a `BottomNavigationBar` or change the background color.

## 🎯 Challenge

Add a drawer with two menu items and a bottom navigation bar with three destinations.

## ❓ 10 Practice Questions

1. What does `Scaffold` provide?
2. Which slot holds primary content?
3. What is `appBar` for?
4. Where does a floating action button go?
5. What does `drawer` represent?
6. Why avoid nested scaffolds?
7. What is a bottom sheet?
8. Where should a snack bar be shown?
9. Does Scaffold replace MaterialApp?
10. When might SafeArea be useful?

## 📝 Summary

`Scaffold` is a page-layout coordinator. Its named slots make standard Material page structures clear, consistent, and easier to maintain.
