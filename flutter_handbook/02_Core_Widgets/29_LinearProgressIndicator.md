# LinearProgressIndicator

## 📖 Introduction

`LinearProgressIndicator` displays progress as a horizontal bar. It can indicate ongoing work or represent a known fraction of completion.

## 🤔 Why Do We Need It?

A horizontal bar fits naturally in page headers, upload areas, and multi-step operations. When progress is measurable, the filled portion helps users understand how much work remains.

## 🧠 Internal Working

If `value` is null, the indicator animates an indeterminate segment because the total progress is unknown. If `value` is between `0.0` and `1.0`, the bar shows determinate progress. Its width is governed by its parent constraints.

Progress visuals should reflect real application state. The indicator does not perform or track the operation by itself.

## ⚙ Constructor (if applicable)

```dart
const LinearProgressIndicator()
```

For known progress, provide a fraction, for example `LinearProgressIndicator(value: 0.35)`.

## 📋 Properties (only relevant ones)

| Property | Purpose |
|---|---|
| `value` | Progress from `0.0` to `1.0`; null means indeterminate |
| `backgroundColor` | Color behind the active progress |
| `color` | Optional active progress color |
| `minHeight` | Minimum height of the bar |
| `semanticsLabel` | Accessible description of the activity |
| `semanticsValue` | Accessible progress value description |

## 💻 Complete Runnable Example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const LinearProgressApp());

class LinearProgressApp extends StatelessWidget {
  const LinearProgressApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('Linear Progress')),
        body: const Padding(
          padding: EdgeInsets.all(24),
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              Text('Uploading file: 35%'),
              SizedBox(height: 12),
              LinearProgressIndicator(
                value: 0.35,
                semanticsLabel: 'File upload progress',
                semanticsValue: '35 percent',
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

## 🔍 Line-by-Line Example Explanation

- `main` starts the app.
- `MaterialApp` and `Scaffold` provide the Material page.
- `Padding` gives the content room from the page edges.
- `Column` puts the text label above the bar.
- `MainAxisSize.min` keeps the sample group at its content height.
- `Text` states which operation the progress belongs to and gives a human-readable percentage.
- `SizedBox` adds separation between the label and indicator.
- `LinearProgressIndicator(value: 0.35)` visualizes 35 percent completion.
- `semanticsLabel` and `semanticsValue` describe the progress to assistive technology.

## 📱 ASCII Phone Screen Preview

```text
+----------------------+
| Linear Progress      |
|----------------------|
|                      |
| Uploading file: 35%  |
| [=======------------]|
|                      |
+----------------------+
```

## 🌳 Widget Tree Diagram

```text
LinearProgressApp
└── MaterialApp
    └── Scaffold
        ├── AppBar
        └── Padding
            └── Column
                ├── Text
                ├── SizedBox
                └── LinearProgressIndicator
```

## ⚙ Flutter Execution Flow

```text
Application updates the indicator value
        ↓
Null value animates an unknown segment; a fraction fills the matching portion
        ↓
Bar is laid out to the width allowed by its parent
        ↓
Operation completes and the progress UI is updated or removed
```

## ⚠ Common Beginner Mistakes

- Hard-coding a percentage that does not match the actual operation.
- Using a determinate value when the completion amount is unknown.
- Showing a bar without identifying what is progressing.
- Forgetting to update or remove the indicator when work ends.
- Supplying a value outside the `0.0` to `1.0` range.

## 🚀 Professional Usage

Use a linear indicator for uploads, downloads, and other operations where a horizontal track fits the surrounding UI. Drive its value from actual progress state, and provide clear labels and completion/error handling.

## 💡 Best Practices

- Connect the indicator to real progress data.
- Choose indeterminate mode when the total work is unknown.
- Explain the task and, when known, display progress in text as well.
- Provide semantic progress information for assistive technology.
- Avoid leaving a completed bar in a busy or misleading state.

## 🧪 Try It Yourself

Remove `value` and the percentage semantics to preview indeterminate activity. Consider what visible text should remain to explain the upload.

## 🎯 Challenge

Sketch a file upload row with a filename, linear progress, and a cancel action. Describe how the row changes on completion and on error.

## ❓ 10 Practice Questions

1. What shape does this progress indicator use?
2. What does a null `value` mean?
3. What does `value: 0.35` show?
4. What constraints determine the bar's width?
5. What does `minHeight` affect?
6. Why should a progress value reflect real work?
7. When is an indeterminate bar appropriate?
8. How can progress be communicated accessibly?
9. What should happen when the task completes?
10. What should the interface show if an upload fails?

## 📝 Summary

`LinearProgressIndicator` visualizes an ongoing task as a horizontal bar. Use a real fraction when known and indeterminate mode otherwise, while explaining the task and updating the UI as work changes.
