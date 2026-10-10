# CircularProgressIndicator

## 📖 Introduction

`CircularProgressIndicator` displays progress as a circular arc. It can communicate that work is ongoing or show a known fraction of a task.

## 🤔 Why Do We Need It?

When an operation takes time, users need to know that the app is working rather than frozen. A progress indicator gives immediate feedback while waiting for content or an operation to finish.

## 🧠 Internal Working

When `value` is null, the indicator is indeterminate: it animates to show ongoing work without claiming a specific completion percentage. When `value` is between `0.0` and `1.0`, it displays determinate progress. It paints within the size allowed by its parent and animates its visual state.

An indicator alone may not explain what is happening. Include concise text or semantic information when context is not obvious.

## ⚙ Constructor (if applicable)

```dart
const CircularProgressIndicator()
```

For a known fraction, use `CircularProgressIndicator(value: 0.6)`.

## 📋 Properties (only relevant ones)

| Property | Purpose |
|---|---|
| `value` | Progress from `0.0` to `1.0`; null means indeterminate |
| `backgroundColor` | Color of the unfilled track |
| `color` | Optional active indicator color |
| `strokeWidth` | Thickness of the circular stroke |
| `semanticsLabel` | Accessible description of the activity |
| `semanticsValue` | Accessible progress value description |

## 💻 Complete Runnable Example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const CircularProgressApp());

class CircularProgressApp extends StatelessWidget {
  const CircularProgressApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('Circular Progress')),
        body: const Center(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              CircularProgressIndicator(
                semanticsLabel: 'Loading content',
              ),
              SizedBox(height: 16),
              Text('Loading content...'),
            ],
          ),
        ),
      ),
    );
  }
}
```

## 🔍 Line-by-Line Example Explanation

- `main` launches `CircularProgressApp`.
- `MaterialApp` creates the Material app environment.
- `Scaffold` provides the screen and app bar.
- `Center` positions the loading group in the body.
- `Column` places the indicator above its message.
- `MainAxisSize.min` keeps the group compact around its children.
- `CircularProgressIndicator` with no `value` shows indeterminate progress.
- `semanticsLabel` describes the activity to assistive technology.
- `SizedBox` adds space between the indicator and label.
- `Text` explains the visible waiting state.

## 📱 ASCII Phone Screen Preview

```text
+----------------------+
| Circular Progress    |
|----------------------|
|                      |
|          ◌           |
|   Loading content... |
|                      |
+----------------------+
```

## 🌳 Widget Tree Diagram

```text
CircularProgressApp
└── MaterialApp
    └── Scaffold
        ├── AppBar
        └── Center
            └── Column
                ├── CircularProgressIndicator
                ├── SizedBox
                └── Text
```

## ⚙ Flutter Execution Flow

```text
Indicator is built with a null or fractional value
        ↓
Null value animates indeterminate activity; fraction shows completion
        ↓
Indicator paints within parent constraints
        ↓
Work completes and the app replaces the waiting UI
```

## ⚠ Common Beginner Mistakes

- Showing a spinner without starting or tracking any actual work.
- Using an indeterminate spinner when a meaningful completion fraction is available.
- Leaving the indicator onscreen after the operation ends.
- Displaying a spinner with no explanation in a context where the activity is unclear.
- Supplying a value outside the expected `0.0` to `1.0` range.

## 🚀 Professional Usage

Use an indeterminate circular indicator for short operations with unknown duration, and determinate progress when the app can measure completion. For long waits, explain the task, preserve useful context, and provide a way to recover if the operation fails.

## 💡 Best Practices

- Show progress only while work is genuinely active.
- Use determinate progress when the completion fraction is known.
- Add a concise visible or semantic description.
- Avoid blocking the whole interface for work that can run in the background.
- Handle completion and errors, not only the loading state.

## 🧪 Try It Yourself

Set `value: 0.6` and observe the determinate arc. Then remove `value` to return to indeterminate activity.

## 🎯 Challenge

Design a loading state for a page that explains what is being loaded and includes both visible and accessible context. Sketch what the page should show if loading fails.

## ❓ 10 Practice Questions

1. What does a circular progress indicator communicate?
2. What does `value: null` mean?
3. What does `value: 0.6` represent?
4. Which range is expected for a determinate value?
5. What does `strokeWidth` change?
6. Why might a text label accompany a spinner?
7. When should an indicator disappear?
8. Which mode is suitable when completion can be measured?
9. What does `semanticsLabel` provide?
10. What should the app do if the work fails?

## 📝 Summary

`CircularProgressIndicator` provides animated circular feedback for ongoing work or a known completion fraction. Match determinate or indeterminate mode to what the app actually knows, explain the activity, and handle completion and failure.
