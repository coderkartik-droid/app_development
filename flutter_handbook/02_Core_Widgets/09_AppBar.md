# AppBar

## 📖 Introduction

`AppBar` is the Material top app bar. It commonly displays a page title, navigation affordance, and actions related to the current screen.

## 🤔 Why Do We Need It?

Users need orientation and predictable access to actions. A consistent top bar communicates where they are and gives them a familiar place to navigate or act.

## 🧠 Internal Working

Placed in `Scaffold.appBar`, `AppBar` participates in the scaffold's layout and receives theme values from `MaterialApp`. Its leading widget, title, and actions are arranged according to available width.

## ⚙ Constructor (if applicable)

```dart
AppBar(
  title: const Text('Inbox'),
  actions: <Widget>[...],
)
```

## 📋 Properties (only relevant ones)

| Property | Purpose |
|---|---|
| `title` | Main heading |
| `leading` | Start-side navigation/control |
| `actions` | End-side action widgets |
| `backgroundColor` | Bar color |
| `elevation` | Surface shadow |
| `bottom` | Tab bar or secondary content |

## 💻 Complete Runnable Example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const AppBarApp());

class AppBarApp extends StatelessWidget {
  const AppBarApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(
          title: const Text('Messages'),
          actions: <Widget>[
            IconButton(
              onPressed: () {},
              tooltip: 'Search messages',
              icon: const Icon(Icons.search),
            ),
          ],
        ),
        body: const Center(child: Text('No new messages')),
      ),
    );
  }
}
```

## 🔍 Line-by-Line Example Explanation

- `AppBar` occupies the scaffold's top slot.
- `title` identifies the current page.
- `actions` receives a list because there may be multiple controls.
- `IconButton` provides a tappable search action.
- `tooltip` explains the icon to accessibility tools and long-press users.
- The body is independent of the top bar.

## 📱 ASCII Phone Screen Preview

```text
+----------------------+
| Messages       (🔍)  |
|----------------------|
|                      |
|    No new messages   |
+----------------------+
```

## 🌳 Widget Tree Diagram

```text
Scaffold
├── AppBar
│   ├── Text
│   └── IconButton → Icon
└── Center → Text
```

## ⚙ Flutter Execution Flow

```text
Scaffold reads appBar → AppBar applies theme → title/actions are laid out
                       → body is laid out below → paint
```

## ⚠ Common Beginner Mistakes

- Filling `actions` with too many low-priority controls.
- Using icons without tooltips.
- Putting page content inside the app bar title.
- Making the title so long that actions become inaccessible.
- Changing colors without checking contrast.

## 🚀 Professional Usage

Use the app bar for orientation and high-value contextual actions. For complex workflows, consider a dedicated page layout rather than overcrowding the bar.

## 💡 Best Practices

- Keep the title concise.
- Add tooltips to icon-only actions.
- Use semantic action ordering.
- Test narrow screens and large text settings.

## 🧪 Try It Yourself

Add a second action for settings and change the title.

## 🎯 Challenge

Create an app bar with a leading back button, a title, and two accessible actions.

## ❓ 10 Practice Questions

1. Where is AppBar normally placed?
2. What is its title for?
3. Which property holds end-side controls?
4. What is `leading` used for?
5. Why add tooltips?
6. What does `bottom` support?
7. Why limit actions?
8. Where does AppBar receive theme values?
9. How should long titles be handled?
10. What accessibility issue can icon-only buttons cause?

## 📝 Summary

`AppBar` provides top-level page orientation and contextual actions. Keep it focused, accessible, and appropriately sized for the screen.
