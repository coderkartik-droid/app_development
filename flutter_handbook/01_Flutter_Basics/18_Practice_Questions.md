# Practice Questions: Flutter Basics

---

## 📖 Introduction

Congratulations on completing the first section of the Flutter Handbook! You have moved from a total beginner to someone who understands the architecture, the tools, and the mental model of Flutter development.

But reading isn't enough. In programming, your brain learns by **recalling** and **solving**. This chapter is designed to test your knowledge across all 17 previous chapters. If you can answer these questions, you are truly ready to start building UI components in the next section.

---

## ❓ Practice Questions

### Section 1: Multiple Choice (MCQs)

1.  **What is the core rendering engine of Flutter?**
    a) WebKit
    b) Blink
    c) Skia/Impeller
    d) Gecko

2.  **Which language is used to write Flutter applications?**
    a) JavaScript
    b) Kotlin
    c) Dart
    d) Swift

3.  **What is the purpose of the `pubspec.yaml` file?**
    a) To write the main UI code
    b) To configure project dependencies and assets
    c) To connect the app to a database
    d) To check for coding errors

4.  **Which command is used to see if your Flutter environment is set up correctly?**
    a) flutter health
    b) flutter status
    c) flutter doctor
    d) flutter check

5.  **What is `BuildContext` in simple terms?**
    a) A variable that stores the app's name
    b) The location and identity of a widget in the tree
    c) The color of the screen
    d) A way to restart the app

---

### Section 2: True or False

6.  **True/False:** Flutter uses a "Bridge" to talk to native components like React Native. (False)
7.  **True/False:** Hot Reload preserves the state of the application. (True)
8.  **True/False:** Every Flutter app must have a `main()` function. (True)
9.  **True/False:** `StatelessWidgets` are more memory-intensive than `StatefulWidgets`. (False)
10. **True/False:** YAML files allow the use of Tabs for indentation. (False)

---

### Section 3: Output Prediction

11. **What will be the output in the console for this code?**
    ```dart
    void main() {
      print("A");
      runApp(const MyApp());
      print("B");
    }
    ```
    *Answer:* A, then B. (Code runs top-to-bottom).

12. **If you wrap a `Text` widget inside a `Center` widget, where will the text appear if the screen is 800x400?**
    *Answer:* At coordinates (400, 200) - the exact center.

---

### Section 4: Theory Questions

13. **Explain the difference between a "Framework" and a "Language."**
14. **What are the "Three Trees" of Flutter, and what is the responsibility of the RenderObject tree?**
15. **Why is it recommended to use the `const` keyword for widgets whenever possible?**
16. **What is the difference between "Hot Reload" and "Hot Restart"? Give one example of when you MUST use Hot Restart.**

---

### Section 5: Coding Exercises

17. **Basic Structure:** Write a `main()` function using the "Fat Arrow" syntax that launches a widget named `MyBookApp`.

18. **Stateless Widget:** Create a `StatelessWidget` called `WelcomeText` that returns a `Text` widget saying "Welcome to Flutter!". Use the `const` keyword.

19. **Tree Building:** Write the code to create a `Scaffold` that has an `AppBar` with the title "My Profile" and a `body` containing an `Icon` of a person centered in the middle.

20. **Debugging Challenge:** Fix the error in this `pubspec.yaml` snippet:
    ```yaml
    flutter:
    assets:
    - images/logo.png
    ```
    *(Hint: Indentation is missing!)*

---

## 📝 Summary

• **Theory:** You understand the Skia engine, the "No Bridge" architecture, and the Three Trees.
• **Environment:** You know how to use `flutter doctor` and manage `pubspec.yaml`.
• **Syntax:** You can write a `main()` function, `runApp()`, and basic `Stateless` classes.
• **Structure:** You understand the `MaterialApp` -> `Scaffold` hierarchy.
• **Performance:** You know the importance of `const` and the difference between Reload and Restart.

**You are now officially a "Flutter Basic" graduate. Get ready for Section 02, where we build real UI elements!**
