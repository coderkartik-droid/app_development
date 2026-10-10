# Navigator.pop in Flutter

## 📖 Introduction
Just as moving forward through screens with `push` is essential for exploring an application, returning backward with [Navigator.pop()](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/05_Navigation/05_Pop.md) is essential for completing tasks, dismissing dialogs, and returning to safety.

In Flutter, [Navigator.pop()](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/05_Navigation/05_Pop.md) removes the topmost [Route](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/05_Navigation/01_Introduction_to_Navigation.md) from the navigation stack. The removed screen plays its exit animation (such as sliding to the right on iOS or fading down on Android) and is unmounted from the element tree. Crucially, [Navigator.pop()](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/05_Navigation/05_Pop.md) can deliver an optional **result argument** back to the screen that originally pushed it, enabling two-way communication between screens without complex state management.

---

## 🤔 Why Do We Need It?
1. **Closing Screens & Modals**: Used to dismiss standard pages, full-screen dialogs, bottom sheets, and alert dialogs.
2. **Returning Results**: Passes picked dates, selected contacts, or saved form data directly back to the awaiting parent route.
3. **Preventing Memory Bloat**: When a user is done with a screen, popping it ensures its controller instances, image caches, and listeners are properly garbage-collected.
4. **Preventing Root Accidents**: Calling `pop()` blindly on the first screen leaves the user staring at an empty black window. Understanding `canPop()` and `maybePop()` prevents this catastrophic failure.

---

## 🌍 Real-Life Analogy
Think of `Navigator.pop()` as **Stepping Out of an Office Room Back into the Hallway**:

```
+-------------------------------------------------------------+
|                     OFFICE BUILDING HALLWAY                 |
|                                                             |
|   You walk into Conference Room B (Navigator.push).         |
|   Inside the room, you discuss project requirements.        |
|                                                             |
|   When finished:                                            |
|   You step out the door (Navigator.pop) AND hand the signed |
|   contract to the manager standing in the hallway!          |
|                                                             |
|   The conference room lights turn off (disposed), and the   |
|   manager in the hallway resumes their day with the signed  |
|   contract in hand.                                         |
+-------------------------------------------------------------+
```

---

## 🧠 Internal Working
When you call `Navigator.pop(context, result)`:

```
                  User Taps Back Button / Save
                                │
                                ▼
                   Navigator.pop(context, result)
                                │
                                ▼
           NavigatorState finds topmost Route in _history
                                │
                                ▼
         Completes the Route's internal Completer(result)
                                │
                                ▼
           Exit Transition Animation Plays (e.g. Slide-Out)
                                │
                                ▼
          OverlayEntry is unmounted from the Overlay
                                │
                                ▼
           Route State & Subtree Disposed from Memory
                                │
                                ▼
         Underlying Route is marked active & receives focus
```

1. **Completer Resolution**: When the route was originally pushed, Flutter created an internal `Completer<T?>`. Calling `pop(context, result)` calls `completer.complete(result)`.
2. **Animation Teardown**: The route's `AnimationController` runs in reverse (e.g. from `1.0` down to `0.0`), playing the platform exit animation.
3. **Element Disposal**: Once the animation finishes, Flutter removes the route's `OverlayEntry` from the render tree, triggering `dispose()` on all child state objects.
4. **Foreground Restoration**: The route directly underneath receives lifecycle notifications via `didPopNext()` and resumes user interaction.

---

## ⚙ Syntax

### 1. Basic Pop (No Data Returned)
```dart
Navigator.pop(context);
```

### 2. Pop with a Return Result
```dart
Navigator.pop(context, 'Confirmed');
```

### 3. Safe Pop (`maybePop`)
```dart
// Checks with PopScope/WillPopScope before popping
final bool didPop = await Navigator.maybePop(context);
```

### 4. Popping Multiple Screens (`popUntil`)
```dart
// Pops screens until the route named '/home' is reached
Navigator.popUntil(context, ModalRoute.withName('/home'));
```

---

## 📋 Important Pop Methods Comparison

| Method | Behavior | Return Type | Safe on Root Screen? |
| :--- | :--- | :--- | :--- |
| **`Navigator.pop([result])`** | Unconditionally removes the topmost route. | `void` | ❌ No (Results in black void if at root) |
| **`Navigator.maybePop([result])`**| Checks `canPop()` and `PopScope` handlers before popping. | `Future<bool>` | ✅ Yes (Returns `false` safely) |
| **`Navigator.canPop()`** | Queries if there are 2 or more routes on the stack. | `bool` | ✅ Yes (Read-only query) |
| **`Navigator.popUntil(predicate)`**| Pops routes continuously until the predicate returns `true`.| `void` | ✅ Yes (Stops at root) |

---

## 💻 Complete Runnable Example
The following production-ready application demonstrates:
1. Popping with data (picking an avatar color).
2. Guarding against root-screen black voids with `Navigator.canPop(context)`.
3. Intercepting back navigation with `PopScope` to show a "Discard Unsaved Changes?" confirmation dialog.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const PopMasterclassApp());
}

class PopMasterclassApp extends StatelessWidget {
  const PopMasterclassApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Navigator.pop Masterclass',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.teal),
        useMaterial3: true,
      ),
      home: const UserProfileScreen(),
    );
  }
}

// ---------------------------------------------------------------------------
// SCREEN 1: User Profile (Root Route)
// ---------------------------------------------------------------------------
class UserProfileScreen extends StatefulWidget {
  const UserProfileScreen({super.key});

  @override
  State<UserProfileScreen> createState() => _UserProfileScreenState();
}

class _UserProfileScreenState extends State<UserProfileScreen> {
  Color _avatarColor = Colors.teal;
  String _colorName = 'Teal (Default)';

  Future<void> _chooseColor() async {
    // Push ColorPickerScreen and await the popped Color result!
    final Map<String, dynamic>? selected = await Navigator.push<Map<String, dynamic>>(
      context,
      MaterialPageRoute(builder: (context) => const ColorPickerScreen()),
    );

    if (!mounted) return;

    if (selected != null) {
      setState(() {
        _avatarColor = selected['color'] as Color;
        _colorName = selected['name'] as String;
      });

      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          content: Text('Avatar color updated to $_colorName!'),
          backgroundColor: _avatarColor,
        ),
      );
    }
  }

  void _testRootPop() {
    // SAFE CHECK: Never pop blindly at the root!
    if (Navigator.canPop(context)) {
      Navigator.pop(context);
    } else {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(
          content: Text('Cannot pop! You are already at the root screen.'),
          backgroundColor: Colors.redAccent,
        ),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('User Profile'),
        centerTitle: true,
      ),
      body: SafeArea(
        child: Padding(
          padding: const EdgeInsets.all(24.0),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: [
              Center(
                child: CircleAvatar(
                  radius: 50,
                  backgroundColor: _avatarColor,
                  child: const Icon(Icons.person, size: 50, color: Colors.white),
                ),
              ),
              const SizedBox(height: 16),
              Text(
                'Selected Theme: $_colorName',
                textAlign: TextAlign.center,
                style: const TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
              ),
              const SizedBox(height: 32),

              FilledButton.icon(
                onPressed: _chooseColor,
                icon: const Icon(Icons.palette),
                label: const Text('Change Avatar Color (push & pop data)'),
                style: FilledButton.styleFrom(
                  backgroundColor: _avatarColor,
                  padding: const EdgeInsets.symmetric(vertical: 16),
                ),
              ),
              const SizedBox(height: 16),

              OutlinedButton.icon(
                onPressed: _testRootPop,
                icon: const Icon(Icons.warning_amber),
                label: const Text('Test Root Pop Safety (canPop check)'),
                style: OutlinedButton.styleFrom(padding: const EdgeInsets.symmetric(vertical: 16)),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

// ---------------------------------------------------------------------------
// SCREEN 2: Color Picker (Pushed Screen with PopScope & Data Return)
// ---------------------------------------------------------------------------
class ColorPickerScreen extends StatefulWidget {
  const ColorPickerScreen({super.key});

  @override
  State<ColorPickerScreen> createState() => _ColorPickerScreenState();
}

class _ColorPickerScreenState extends State<ColorPickerScreen> {
  final List<Map<String, dynamic>> _colors = const [
    {'name': 'Deep Purple', 'color': Colors.deepPurple},
    {'name': 'Emerald Green', 'color': Colors.green},
    {'name': 'Crimson Red', 'color': Colors.redAccent},
    {'name': 'Amber Gold', 'color': Colors.amber},
    {'name': 'Royal Indigo', 'color': Colors.indigo},
  ];

  bool _hasMadeSelection = false;

  Future<bool> _showExitDialog() async {
    final shouldLeave = await showDialog<bool>(
      context: context,
      builder: (ctx) => AlertDialog(
        title: const Text('Discard Color Selection?'),
        content: const Text('You have not selected a new color. Do you want to return to your profile?'),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(ctx, false), // Stay on screen
            child: const Text('Cancel'),
          ),
          FilledButton(
            onPressed: () => Navigator.pop(ctx, true), // Allow pop
            child: const Text('Leave'),
          ),
        ],
      ),
    );
    return shouldLeave ?? false;
  }

  @override
  Widget build(BuildContext context) {
    // PopScope intercepts back button and edge swipes
    return PopScope(
      canPop: _hasMadeSelection, // If false, triggers onPopInvokedWithResult
      onPopInvokedWithResult: (bool didPop, dynamic result) async {
        if (didPop) return; // Pop already succeeded

        final shouldPop = await _showExitDialog();
        if (shouldPop && context.mounted) {
          Navigator.pop(context); // Manually pop after user confirms
        }
      },
      child: Scaffold(
        appBar: AppBar(
          title: const Text('Pick Avatar Color'),
        ),
        body: SafeArea(
          child: ListView.separated(
            padding: const EdgeInsets.all(16.0),
            itemCount: _colors.length,
            separatorBuilder: (_, __) => const SizedBox(height: 10),
            itemBuilder: (context, index) {
              final item = _colors[index];
              final color = item['color'] as Color;
              final name = item['name'] as String;

              return Card(
                elevation: 0,
                color: color.withOpacity(0.1),
                shape: RoundedRectangleBorder(
                  borderRadius: BorderRadius.circular(12),
                  side: BorderSide(color: color.withOpacity(0.3)),
                ),
                child: ListTile(
                  leading: CircleAvatar(backgroundColor: color),
                  title: Text(name, style: const TextStyle(fontWeight: FontWeight.bold)),
                  trailing: const Icon(Icons.check_circle_outline),
                  onTap: () {
                    setState(() => _hasMadeSelection = true);
                    // Pop with return data map!
                    Navigator.pop(context, item);
                  },
                ),
              );
            },
          ),
        ),
      ),
    );
  }
}
```

---

## 🔍 Line-by-Line Example Explanation
- `Navigator.pop(context, item)`:
  - **What it does**: Closes `ColorPickerScreen` and delivers `item` back to the awaiting `UserProfileScreen`.
  - **Internal behavior**: The route's exit animation starts; `item` is stored in the route's completer.
- `Navigator.canPop(context)`:
  - **What it does**: Inspects whether the navigation stack has more than 1 entry.
  - **Why it exists**: Prevents invoking `Navigator.pop` when sitting on the root screen, avoiding an empty black screen.
- `PopScope(canPop: _hasMadeSelection, onPopInvokedWithResult: ...)`:
  - **What it does**: Controls whether system back gestures and AppBar back arrows are permitted to pop immediately.
  - **Internal behavior**: If `canPop` is `false`, Flutter cancels the pop and invokes `onPopInvokedWithResult`, allowing you to show the confirmation dialog.

---

## 📱 ASCII Phone Screen Preview

```
[ Screen 1: User Profile ]                   [ Screen 2: Color Picker ]
+----------------------------+              +----------------------------+
|        User Profile        |              | [←]  Pick Avatar Color     |
+----------------------------+              +----------------------------+
|                            |    PUSH      |                            |
|          [ 👤 ]            | -----------> | (●) Deep Purple            |
|    Theme: Teal (Default)   |              | (●) Emerald Green          |
|                            |    POP       | (●) Crimson Red   [Tap!]   |
| [ Change Avatar Color ]    | <----------- | (●) Amber Gold             |
| [ Test Root Pop Safety ]   | (Sends item) |                            |
+----------------------------+              +----------------------------+
                                                    │ Back Button Tapped
                                                    ▼ (If no selection)
                                            ┌────────────────────────────┐
                                            │ Discard Selection?         │
                                            │ [Cancel]          [Leave]  │
                                            └────────────────────────────┘
```

---

## 🌳 Widget Tree
```
MaterialApp
 └── Navigator [State: NavigatorState]
      └── Overlay
           ├── OverlayEntry (UserProfileScreen) [Base layer: canPop = false]
           │    └── Scaffold
           │         ├── AppBar
           │         └── Column (Avatar & Buttons)
           └── OverlayEntry (ColorPickerScreen) [When pushed: canPop = true]
                └── PopScope [canPop: _hasMadeSelection]
                     └── Scaffold
                          ├── AppBar
                          └── ListView (Color tiles)
```

---

## ⚙ Flutter Execution Flow
1. **User Action**: The user taps `[Change Avatar Color]`.
2. **Push**: `Navigator.push` mounts `ColorPickerScreen`.
3. **Item Selected**: The user taps "Crimson Red".
4. **Pop With Data**: `Navigator.pop(context, item)` executes.
5. **Animation**: `ColorPickerScreen` slides out; its widget tree unmounts.
6. **Delivery**: The `await Navigator.push` line receives `item`.
7. **UI Update**: `UserProfileScreen` calls `setState()`, repainting the avatar in Crimson Red and showing a SnackBar.

---

## ⚠ Common Beginner Mistakes

### 1. Popping the Root Screen (The Black Void)
```dart
// ❌ CATASTROPHIC BUG: Calling pop on the home screen
onPressed: () {
  Navigator.pop(context); // Screen vanishes into pitch black emptiness!
}

// ✅ GOOD: Always guard root pops
onPressed: () {
  if (Navigator.canPop(context)) {
    Navigator.pop(context);
  }
}
```

### 2. Forgetting to Return Data in `Navigator.pop`
```dart
// ❌ WRONG: Popping without arguments when the caller expects data
onTap: () {
  _saveColor(color);
  Navigator.pop(context); // Caller receives null!
}

// ✅ CORRECT: Pass the result directly in pop()
onTap: () {
  Navigator.pop(context, color); // Caller receives the color!
}
```

### 3. Calling `Navigator.pop` After an Async Call Without `mounted` Check
```dart
// ❌ BUG: Calling pop after widget is already disposed
void _submit() async {
  await sendNetworkRequest();
  Navigator.pop(context); // Crash if user already navigated back!
}

// ✅ GOOD: Guard with if (!mounted) return;
void _submit() async {
  await sendNetworkRequest();
  if (!mounted) return;
  Navigator.pop(context);
}
```

---

## 🚀 Professional Usage
- **Custom Pickers & Modals (e.g., Image Picker, Location Picker)**: Selecting a photo from an album pops the gallery view and delivers the `File` path back to the form.
- **Form Cancellation Interception**: Banking and checkout screens intercept pops to warn: *"Leaving now will cancel your transaction"*.
- **Multi-Step Unwinding**: Using `Navigator.popUntil` to return straight to the home screen after order completion.

---

## 💡 Best Practices
1. **Use `Navigator.maybePop(context)`**: Prefer `maybePop` over `pop` when user unsaved changes may be lost.
2. **Always check `Navigator.canPop(context)`**: Essential when writing custom back buttons or root-level drawer headers.
3. **Type the result object**: If popping a custom model, ensure the pushing route awaited `Navigator.push<MyModel>()`.
4. **Clean up resources before popping**: Cancel active streams or timers in `dispose()`.

---

## 🧪 Try It Yourself
1. Remove `PopScope` from `ColorPickerScreen` and observe how Android back button pops immediately without confirmation.
2. Add a third screen (`ColorDetailsScreen`) and test popping through both screens back to `UserProfileScreen`.
3. Try clicking "Test Root Pop Safety" on `UserProfileScreen` to observe how `canPop()` protects your app from crashing.

---

## 🎯 Challenge
Build an "Unsaved Document Editor":
- A text field with initial text `"Hello World"`.
- If the user edits the text and attempts to pop via AppBar back arrow or system gesture, trigger a confirmation dialog: *"Discard unsaved changes?"*.
- If the user taps *"Save"*, pop and return the updated string.
- If the user didn't make any changes, allow popping immediately without asking!

---

## ❓ Practice Questions

### Q1. What does `Navigator.pop(context)` do?
- A) Minimizes the app to the background
- B) Removes the topmost route from the navigation stack and unmounts it
- C) Clears all app storage
- D) Restarts the widget tree
*Answer: B* — Pops the active route off the stack, revealing the previous route.

### Q2. What happens if you call `Navigator.pop(context)` when only 1 screen exists on the stack?
- A) The app cleanly exits to the home screen
- B) The app renders a blank black screen because there are no remaining routes to draw
- C) Flutter throws a compile-time error
- D) The screen refreshes
*Answer: B* — Popping the root route leaves Flutter with an empty render stack.

### Q3. How do you return a boolean `true` to the screen that pushed the current route?
- A) `Navigator.return(true)`
- B) `Navigator.pop(context, true)`
- C) `Navigator.push(context, true)`
- D) `Navigator.complete(true)`
*Answer: B* — Pass the value as the second argument to `Navigator.pop()`.

### Q4. What is the difference between `Navigator.pop()` and `Navigator.maybePop()`?
- A) `pop()` is faster
- B) `pop()` unconditionally removes the route; `maybePop()` checks with `PopScope` and `canPop()` first
- C) `maybePop()` only works on iOS
- D) `maybePop()` requires a password
*Answer: B* — `maybePop` safely respects pop-interception handlers.

### Q5. Which widget is used in modern Flutter (v3.12+) to intercept back navigation gestures?
- A) `WillPopScope`
- B) `PopScope`
- C) `BackHandler`
- D) `InterceptRoute`
*Answer: B* — `PopScope` is the modern replacement for legacy `WillPopScope`.

### Q6. What does `Navigator.canPop(context)` return?
*Answer*: A boolean indicating whether there are multiple routes on the navigation stack (`true` if safe to pop, `false` if at the root).

### Q7. How do you pop multiple screens all the way back to the root route?
*Answer*:
```dart
Navigator.popUntil(context, (route) => route.isFirst);
```

### Q8. Output Prediction: What will `print(data)` display if Screen 2 calls `Navigator.pop(context, {'status': 200})`?
```dart
final data = await Navigator.push(context, route);
print(data['status']);
```
*Answer*: `200`.

### Q9. Can you pass complex custom Dart objects through `Navigator.pop(context, result)`?
*Answer*: Yes, `pop` accepts any object type (strings, integers, custom classes, lists, maps).

### Q10. Write a minimal code snippet to pop the current screen only if `canPop` is true.
*Answer*:
```dart
if (Navigator.canPop(context)) Navigator.pop(context);
```

---

## 📝 Summary
- [Navigator.pop()](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/05_Navigation/05_Pop.md) removes the active screen from the stack, playing an exit transition and disposing of its state.
- Pass an optional argument `Navigator.pop(context, result)` to return data back to `await Navigator.push()`.
- Never call unconditional `pop()` at the root route; use `canPop()` or `maybePop()` to prevent black screen crashes.
- Use `PopScope` to intercept back gestures and display "Discard unsaved changes?" confirmation dialogs.
