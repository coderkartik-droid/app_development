# FloatingActionButton

## 📖 Introduction

`FloatingActionButton` (FAB) is a prominent Material button that floats above a screen's main content. It usually represents the primary action for the current page, such as creating a new item.

## 🤔 Why Do We Need It?

Some screens have one central action users need to find quickly. A FAB gives that action a consistent, prominent location without mixing it into every list row or toolbar item.

## 🧠 Internal Working

The FAB is placed in a `Scaffold` slot, which positions it relative to the page content and handles common layout behavior. It uses Material interaction and elevation, and its non-null `onPressed` callback enables it. Its label or tooltip should make the action clear.

## ⚙ Constructor (if applicable)

```dart
FloatingActionButton(
  onPressed: () {},
  tooltip: 'Add item',
  child: const Icon(Icons.add),
)
```

## 📋 Properties (only relevant ones)

| Property | Purpose |
|---|---|
| `onPressed` | Action callback; `null` disables the FAB |
| `child` | Usually an icon or compact content |
| `tooltip` | Explains the action and provides accessible context |
| `backgroundColor` | Optional surface color |
| `mini` | Requests a smaller FAB variant |
| `heroTag` | Identifies the FAB for hero transitions; keep tags unique when multiple FABs participate |

## 💻 Complete Runnable Example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const FabApp());

class FabApp extends StatelessWidget {
  const FabApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('Floating Action Button')),
        body: const Center(child: Text('Create a new note')),
        floatingActionButton: Builder(
          builder: (scaffoldContext) {
            return FloatingActionButton(
              tooltip: 'Add note',
              onPressed: () {
                ScaffoldMessenger.of(scaffoldContext).showSnackBar(
                  const SnackBar(content: Text('New note action selected')),
                );
              },
              child: const Icon(Icons.add),
            );
          },
        ),
      ),
    );
  }
}
```

## 🔍 Line-by-Line Example Explanation

- `main` starts `FabApp`.
- `MaterialApp` supplies the app's Material environment.
- `Scaffold` provides the app bar, body, and a dedicated FAB position.
- `AppBar` names the current page.
- The body text describes the central task.
- `floatingActionButton` places the control above the scaffold body.
- `FloatingActionButton` presents the page's prominent action.
- `tooltip` identifies what the plus symbol does.
- The non-null callback displays feedback when activated.
- `Icon(Icons.add)` is the button's visual child.

## 📱 ASCII Phone Screen Preview

```text
+----------------------+
| Floating Action Btn  |
|----------------------|
|                      |
|   Create a new note  |
|                      |
|                 ( + )|
+----------------------+
```

## 🌳 Widget Tree Diagram

```text
FabApp
└── MaterialApp
    └── Scaffold
        ├── AppBar
        ├── body: Center → Text
        └── floatingActionButton
            └── FloatingActionButton
                └── Icon
```

## ⚙ Flutter Execution Flow

```text
Scaffold lays out page and FAB slot
        ↓
FAB is positioned above the main content
        ↓
Material provides elevation and interaction feedback
        ↓
User activates the page's primary action
```

## ⚠ Common Beginner Mistakes

- Adding several FABs for unrelated actions and weakening the hierarchy.
- Using a generic plus icon without clarifying what it adds.
- Placing the FAB outside the scaffold slot and expecting scaffold positioning.
- Forgetting the tooltip on an icon-only FAB.
- Using a FAB for a secondary or destructive action that does not deserve primary prominence.

## 🚀 Professional Usage

Use one FAB when a screen has a clear, frequent primary action. Its meaning should be specific to the current page. If a screen genuinely needs multiple related FABs, design the interaction and hero tags deliberately rather than stacking ambiguous controls.

## 💡 Best Practices

- Reserve the FAB for the screen's principal action.
- Pair icon-only visuals with a useful tooltip.
- Avoid placing it where it obscures important content or controls.
- Provide an appropriate result or navigation after activation.
- Use a regular button for secondary actions.

## 🧪 Try It Yourself

Change the page to a task list and rename the FAB action to “Add task.” Make the body and tooltip explain the same action.

## 🎯 Challenge

Design a notes screen with a scrollable list and one FAB for creating a note. Ensure the FAB's purpose is clear and it does not obscure list content.

## ❓ 10 Practice Questions

1. What does FAB stand for?
2. Where is a FAB commonly placed?
3. What kind of action should it represent?
4. Which `Scaffold` property hosts the FAB?
5. How is the FAB enabled or disabled?
6. Why should an icon-only FAB have a tooltip?
7. What can go wrong with multiple unrelated FABs?
8. What is the purpose of `heroTag`?
9. When is a regular button more appropriate?
10. What should be checked to prevent content obstruction?

## 📝 Summary

`FloatingActionButton` highlights a screen's primary action in the scaffold. Use it sparingly, make its purpose accessible and specific, and ensure it does not interfere with other content.
