# Quick Revision: Flutter Basics

---

## 📖 Introduction

You have covered a lot of ground! From understanding what Flutter is to building your first app and avoiding common mistakes. This chapter is a "Cheat Sheet" or a "Refresher." 

Whether you are preparing for a quiz, an interview, or just want to quickly remember a concept before you start coding, this revision guide summarizes the core "DNA" of Flutter in one place.

---

## 🧠 Key Concepts Refresher

### 1. What is Flutter?
Flutter is Google's UI Toolkit (SDK) for building multi-platform apps from a **single codebase**. It uses the **Dart** language and paints its own pixels using the **Skia/Impeller** engine.

### 2. Everything is a Widget
In Flutter, even "Padding," "Alignment," and "Themes" are widgets. If you want to change how something looks or where it sits, you wrap it in another widget.

### 3. The Three Trees
Flutter doesn't just manage your code; it manages three trees:
*   **Widget Tree:** Your blueprints.
*   **Element Tree:** The location (BuildContext).
*   **RenderObject Tree:** The actual painters.

### 4. BuildContext
It is the "ID Badge" of a widget. It tells the widget where it is in the tree and allows it to find parent information (like the Theme).

---

## ⚙ Essential Syntax Snippets

**The main function:**
```dart
void main() => runApp(const MyApp());
```

**Stateless Widget Template:**
```dart
class MyHeader extends StatelessWidget {
  const MyHeader({super.key});
  @override
  Widget build(BuildContext context) {
    return const Text('I am permanent');
  }
}
```

**MaterialApp Wrapper:**
```dart
MaterialApp(
  home: Scaffold(
    appBar: AppBar(title: Text('Title')),
    body: Center(child: Text('Content')),
  ),
)
```

---

## 🔄 Lifecycle & Workflow

*   **Hot Reload (`r`):** Updates the UI instantly. **Preserves State** (keeps your data).
*   **Hot Restart (`R`):** Restarts the app from `main()`. **Wipes State** (resets everything).
*   **Full Build:** Required after adding new assets or native plugins.

---

## 📂 Project Structure Map

*   **`lib/`**: Where your code lives.
*   **`main.dart`**: The entry point.
*   **`pubspec.yaml`**: The configuration (packages, images, fonts).
*   **`android/` / `ios/`**: Platform-specific foundation code.

---

## ⚠ Quick "Don'ts"

1.  **Don't** use Tabs in `pubspec.yaml` (Use spaces).
2.  **Don't** forget `const` for static widgets.
3.  **Don't** put logic in the `build()` method.
4.  **Don't** ignore the `analysis_options.yaml` warnings.

---

## 📱 The "Perfect" First App Structure

```dart
// 1. Imports
import 'package:flutter/material.dart';

// 2. Entry
void main() => runApp(const MyApp());

// 3. App Root
class MyApp extends StatelessWidget {
  const MyApp({super.key});
  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      home: MyScreen(),
    );
  }
}

// 4. Individual Screen
class MyScreen extends StatelessWidget {
  const MyScreen({super.key});
  @override
  Widget build(BuildContext context) {
    return const Scaffold(
      body: Center(child: Text('Basics Mastered!')),
    );
  }
}
```

---

## ❓ Rapid Fire Questions

1.  **Language?** Dart.
2.  **Engine?** Skia/Impeller.
3.  **Single/Multiple Codebase?** Single.
4.  **Widget to fill screen?** runApp().
5.  **The "Skeleton" widget?** Scaffold.
6.  **Location in tree?** BuildContext.
7.  **Preserves data during update?** Hot Reload.
8.  **Where to add images?** pubspec.yaml.
9.  **File name case?** snake_case.
10. **Class name case?** PascalCase.

---

## 📝 Final Checklist

- [ ] Flutter SDK is installed and `flutter doctor` is green.
- [ ] You understand the difference between Stateless and Stateful.
- [ ] You know why `MaterialApp` and `Scaffold` are needed.
- [ ] You can explain the "Three Trees" concept simply.
- [ ] You have successfully built a "Hello World" app.

**Congratulations! You have completed the foundation of Flutter. You are now ready to dive into "Core Widgets" where you will learn how to build beautiful, complex interfaces.**
