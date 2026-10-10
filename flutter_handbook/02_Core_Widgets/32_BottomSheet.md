# BottomSheet

## 📖 Introduction

A bottom sheet is a surface that slides up from the bottom edge of the screen to show a short set of related choices or supporting content. Flutter provides modal bottom sheets through `showModalBottomSheet`.

## 🤔 Why Do We Need It?

Some actions need more explanation or options than fit comfortably in a menu, but do not require a full page. A bottom sheet keeps the current screen visible behind a temporary panel and lets the user complete a focused task.

## 🧠 Internal Working

`showModalBottomSheet` places a route in the app's navigation overlay and returns a `Future` that completes when the sheet closes. The builder receives a context for the sheet subtree. Use that context with `Navigator.pop` to close the sheet and optionally return a value.

The builder should return content that sizes appropriately. `SafeArea` helps keep controls clear of system intrusions; scrollable content should be made scrollable rather than forced into an unbounded fixed height.

## ⚙ Constructor (if applicable)

The common way to open a modal sheet is:

```dart
showModalBottomSheet<void>(
  context: context,
  builder: (sheetContext) {
    return const SafeArea(
      child: Padding(
        padding: EdgeInsets.all(16),
        child: Text('Sheet content'),
      ),
    );
  },
)
```

## 📋 Properties (only relevant ones)

| Parameter | Purpose |
|---|---|
| `context` | Context used to find the navigator and theme |
| `builder` | Builds the sheet's content |
| `isScrollControlled` | Allows the sheet to use more height for scrollable or larger content |
| `enableDrag` | Allows the user to dismiss the sheet by dragging when enabled |
| `isDismissible` | Allows dismissal by tapping outside the sheet when enabled |
| `useSafeArea` | Applies safe area handling around the sheet |

## 💻 Complete Runnable Example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const BottomSheetApp());

class BottomSheetApp extends StatelessWidget {
  const BottomSheetApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('Bottom Sheet')),
        body: Builder(
          builder: (pageContext) {
            return Center(
              child: ElevatedButton(
                onPressed: () {
                  showModalBottomSheet<void>(
                    context: pageContext,
                    builder: (sheetContext) {
                      return SafeArea(
                        child: Padding(
                          padding: const EdgeInsets.all(24),
                          child: Column(
                            mainAxisSize: MainAxisSize.min,
                            crossAxisAlignment: CrossAxisAlignment.start,
                            children: [
                              const Text(
                                'Choose an option',
                                style: TextStyle(
                                  fontSize: 20,
                                  fontWeight: FontWeight.bold,
                                ),
                              ),
                              const SizedBox(height: 12),
                              const Text('This panel can hold a few related actions.'),
                              Align(
                                alignment: Alignment.centerRight,
                                child: TextButton(
                                  onPressed: () => Navigator.pop(sheetContext),
                                  child: const Text('Close'),
                                ),
                              ),
                            ],
                          ),
                        ),
                      );
                    },
                  );
                },
                child: const Text('Open options'),
              ),
            );
          },
        ),
      ),
    );
  }
}
```

## 🔍 Line-by-Line Example Explanation

- `main` launches `BottomSheetApp`.
- `MaterialApp` provides the navigation and Material styling environment.
- `Scaffold` supplies the screen and app bar.
- `Builder` creates `pageContext` below the scaffold.
- `ElevatedButton` starts the bottom-sheet flow.
- `showModalBottomSheet<void>` presents a modal route that returns no value in this example.
- The sheet builder receives `sheetContext`, the context used to close this sheet.
- `SafeArea` keeps sheet content away from system UI areas.
- `Padding` gives the sheet's content breathing room.
- `Column` groups a heading, explanation, and action vertically.
- `MainAxisSize.min` lets the sheet wrap the content instead of filling unnecessary height.
- `Align` places the Close action at the trailing side of the column.
- The button pops the sheet route with `Navigator.pop`.

## 📱 ASCII Phone Screen Preview

```text
+----------------------+
| Bottom Sheet         |
|----------------------|
|                      |
|    [Open options]    |
|                      |
|======================|
| Choose an option     |
| This panel can hold  |
| a few related actions|
|               Close  |
+----------------------+
```

## 🌳 Widget Tree Diagram

```text
BottomSheetApp
└── MaterialApp
    └── Scaffold
        ├── AppBar
        └── Builder
            └── Center
                └── ElevatedButton

Overlay (when the sheet is open)
└── SafeArea
    └── Padding
        └── Column
            ├── Text (heading)
            ├── SizedBox
            ├── Text (description)
            └── Align
                └── TextButton
```

## ⚙ Flutter Execution Flow

```text
User opens the options
        ↓
showModalBottomSheet adds a modal route
        ↓
Builder creates the sheet content
        ↓
User chooses an action or dismisses the sheet
        ↓
The route closes and its future completes
```

## ⚠ Common Beginner Mistakes

- Placing a long, non-scrollable column in a sheet that cannot fit on a small screen.
- Using the page context to close the sheet instead of its route context.
- Forgetting safe areas around content near screen edges.
- Treating a bottom sheet as a permanent replacement for navigation.
- Providing many unrelated actions without a clear grouping or heading.

## 🚀 Professional Usage

Use a modal bottom sheet for a compact set of contextual options, filters, or details that relate to the current screen. For complex tasks, use a full screen or a dedicated route. If the sheet returns a selection, define a result type and handle it after the future completes.

## 💡 Best Practices

- Keep the task and available actions focused.
- Use `SafeArea` and support scrolling for content that can grow.
- Make dismissal behavior appropriate to the task.
- Use the builder's context to pop the sheet.
- Return a value when the caller needs to know which option was selected.
- Avoid obscuring important, unsaved content without a clear way back.

## 🧪 Try It Yourself

Add a second button that closes the sheet with `Navigator.pop(sheetContext, 'share')`. Change the generic result type and inspect the returned value in the caller.

## 🎯 Challenge

Build a filter sheet with a heading, two filter choices, and an Apply action. Return the selected filter to the page and keep the content usable on a short screen.

## ❓ 10 Practice Questions

1. What is a modal bottom sheet useful for?
2. Which function displays a modal bottom sheet?
3. What does the returned `Future` represent?
4. What context should close the sheet?
5. What does `SafeArea` help prevent?
6. Why might sheet content need to scroll?
7. What does `isDismissible` control?
8. When is a full page preferable to a sheet?
9. How can a sheet return a selection?
10. Why should sheet actions be related to one another?

## 📝 Summary

`showModalBottomSheet` presents focused, temporary content from the bottom of the screen. Build it with safe, appropriately sized content, close it using its own context, and return a result when the caller needs the user's choice.
