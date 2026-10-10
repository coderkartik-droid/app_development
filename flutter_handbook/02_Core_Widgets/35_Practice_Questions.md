# Practice Questions

## 📖 Introduction

This chapter is a self-check for the core Flutter concepts and widgets covered so far. The questions move from foundational widget-tree ideas to layout, Material controls, and user feedback.

## 🤔 Why Do We Need It?

Practice helps reveal whether you can choose a widget for a reason, explain its constraints, and predict how it behaves in a real screen. Recalling definitions is useful, but applying concepts to a new layout is a stronger test of understanding.

## 🧠 Internal Working

Answer each question before looking at the self-check key. For code questions, trace the tree from parent to child: identify the constraints each parent passes, what each child contributes, and where a callback or navigation result goes.

The example below combines a scaffold, a card, spacing, text, and an action button. It is a starting point for reasoning about composition, not a complete app feature.

## ⚙ Constructor (if applicable)

This chapter has no new widget constructor. It revises common patterns:

```dart
const Padding(
  padding: EdgeInsets.all(16),
  child: Text('A widget can compose other widgets.'),
)
```

## 📋 Properties (only relevant ones)

| Topic | Review prompt |
|---|---|
| Widget configuration | Which values are immutable inputs? |
| Constraints | What limits the child's width and height? |
| Layout | Which widget communicates the intended spacing or positioning? |
| Interaction | What callback runs when the user activates a control? |
| Feedback | Should the result be inline, transient, or require a decision? |

## 💻 Complete Runnable Example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const PracticeQuestionsApp());

class PracticeQuestionsApp extends StatelessWidget {
  const PracticeQuestionsApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('Practice Review')),
        body: Center(
          child: Card(
            margin: const EdgeInsets.all(24),
            child: Padding(
              padding: const EdgeInsets.all(20),
              child: Column(
                mainAxisSize: MainAxisSize.min,
                children: [
                  const Text(
                    'Ready to review?',
                    style: TextStyle(fontSize: 20),
                  ),
                  const SizedBox(height: 12),
                  const Text('Think through each question before checking your notes.'),
                  const SizedBox(height: 16),
                  OutlinedButton(
                    onPressed: () {},
                    child: const Text('Start review'),
                  ),
                ],
              ),
            ),
          ),
        ),
      ),
    );
  }
}
```

## 🔍 Line-by-Line Example Explanation

- `main` runs the app using `PracticeQuestionsApp`.
- `MaterialApp` sets up the Material environment.
- `Scaffold` builds a screen with a top app bar and body.
- `Center` centers the card within the body space.
- `Card` groups the review prompt and action.
- `Padding` provides internal breathing room.
- `Column` arranges the message and action vertically.
- `MainAxisSize.min` makes the column fit its contents.
- The first `Text` serves as the card heading.
- `SizedBox` inserts deliberate vertical spacing.
- The second `Text` explains what to do.
- `OutlinedButton` presents a secondary-style action.
- The example callback is empty because the sample focuses on layout; connect a real review flow in an application.

## 📱 ASCII Phone Screen Preview

```text
+----------------------+
| Practice Review      |
|----------------------|
|                      |
|    +------------+    |
|    | Ready to   |    |
|    | review?    |    |
|    | Think ...  |    |
|    |[Start review]|  |
|    +------------+    |
|                      |
+----------------------+
```

## 🌳 Widget Tree Diagram

```text
PracticeQuestionsApp
└── MaterialApp
    └── Scaffold
        ├── AppBar
        │   └── Text
        └── Center
            └── Card
                └── Padding
                    └── Column
                        ├── Text
                        ├── SizedBox
                        ├── Text
                        ├── SizedBox
                        └── OutlinedButton
                            └── Text
```

## ⚙ Flutter Execution Flow

```text
Root widget builds the app and page
        ↓
Center and Card arrange the content within parent constraints
        ↓
Padding and Column lay out the message and action
        ↓
Button callback runs when activated
        ↓
Application provides a suitable result or feedback
```

## ⚠ Common Beginner Mistakes

- Reading the question but not explaining why a widget fits the situation.
- Confusing a widget's requested size with the size allowed by its parent.
- Treating an `Icon` as interactive or assuming a decorative arrow performs navigation.
- Forgetting that button behavior belongs in its callback.
- Using a dialog, snack bar, or progress indicator without matching it to the actual user need.
- Checking an answer without trying to reason through the widget tree first.

## 🚀 Professional Usage

Use these questions during code reviews and design discussions: ask what each widget is responsible for, what constraints it receives, and how a user will understand and complete the action. When something fails, inspect the nearest ancestor constraints and context before adding more wrappers.

## 💡 Best Practices

- Explain both what a widget does and why it belongs in the tree.
- Trace layout constraints from parent to child.
- Prefer semantic, focused widgets over manual positioning.
- Verify that actions have callbacks and appropriate outcomes.
- Test long text, small screens, and accessibility settings.
- Use the answer key as a prompt for further reading, not as a substitute for building examples.

## 🧪 Try It Yourself

Answer these ten questions in your own words before checking the key:

1. What does a Flutter widget describe?
2. Why should `BuildContext` be understood as location-dependent?
3. What is the difference between a stateless widget and a stateful widget?
4. How does `Center` decide where to place its child?
5. When should you choose `Padding` instead of a fixed-size `SizedBox`?
6. Why does a `ListTile` commonly belong in a scrollable parent?
7. What happens to a Material button when `onPressed` is null?
8. When should a progress indicator use a determinate value?
9. What is the difference between a `SnackBar` and an `AlertDialog`?
10. How does `Navigator.pop` close a modal and optionally return a result?

### Self-check answer key

1. A widget is an immutable description of part of the user interface.
2. Context identifies a widget's position in the tree, so inherited services and ancestors available depend on where it is used.
3. A stateless widget has no mutable local state; a stateful widget has a separate `State` object for data that can change over time.
4. It centers its child within the space it receives from its parent, subject to those constraints.
5. Use `Padding` when the intent is inset space around a child; a `SizedBox` is useful for a fixed gap or dimension.
6. The list may be taller than the available viewport, so a scrollable parent allows users to reach all rows.
7. The button is disabled and does not invoke an action.
8. Use a determinate value when the app can report a meaningful fraction of completion between zero and one.
9. A snack bar gives brief, non-blocking feedback; an alert dialog requests attention for an important message or decision.
10. It removes the current route, and an optional value supplied to `pop` completes the route's future with that result.

## 🎯 Challenge

Build a small page that applies at least five concepts from this section. Before running it, draw its widget tree and explain which parent constraints reach each layout widget. After running, test a narrow screen and a longer text scale.

## ❓ 10 Practice Questions

1. Why are Flutter's widget configurations immutable?
2. How does a parent's constraint affect a child's desired dimensions?
3. What is the difference between a visual widget and an interactive control?
4. When should a reusable piece of UI become its own widget?
5. Why can an empty callback create a misleading interface?
6. How does a tooltip support an icon-only action?
7. What should an app display while progress is genuinely unknown?
8. Why is a modal sheet sometimes preferable to navigating to a new page?
9. How can a caller safely use a context after awaiting a dialog?
10. What should you inspect first when a widget appears in the wrong position?

## 📝 Summary

Strong widget knowledge means being able to explain responsibilities, constraints, and user behavior—not just memorize names. Work through the questions, draw widget trees, and verify your reasoning by building and testing small examples.
