# Navigator.push in Flutter

## 📖 Introduction
Moving forward into an application's visual hierarchy is the most fundamental user interaction on mobile devices. When you tap a contact in a list, you expect their detailed profile to appear. When you tap a product card in a catalog, you expect the product specification page to display.

In Flutter, moving forward to a new screen is accomplished via [Navigator.push()](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/05_Navigation/04_Push.md). The `push` operation adds a new [Route](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/05_Navigation/01_Introduction_to_Navigation.md) to the top of the history stack managed by the [Navigator](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/05_Navigation/02_Navigator.md). Crucially, `Navigator.push()` is an **asynchronous operation** that returns a Dart `Future`. This means the calling screen can pause and await a return value delivered whenever the newly pushed screen is eventually popped.

---

## 🤔 Why Do We Need It?
1. **Hierarchical Drill-Down**: Enables moving from broad overviews (e.g. Categories) down to specific items (e.g. Products $\to$ Reviews).
2. **State Preservation**: Pushing a new screen does not destroy the current screen; the current screen remains preserved in memory underneath.
3. **Data Return Channel**: The `Future` returned by `push()` provides an elegant, decoupled channel to receive user selections, form results, or status confirmations back from the child screen.
4. **Platform Gesture Support**: Automatically equips the new screen with standard OS gestures (like Android's system back button and iOS's edge-swipe).

---

## 🌍 Real-Life Analogy
Think of `Navigator.push()` as **Placing a New Book on Top of a Stack of Books on Your Desk**:

```
+-------------------------------------------------------------+
|                      THE DESK BOOKSTACK                     |
|                                                             |
|   +-----------------------------------------------------+   |
|   | Book 3: Chapter 4 Details (Just Placed on Top)      |   |  <- Navigator.push()
|   +-----------------------------------------------------+   |
|   | Book 2: Table of Contents                           |   |
|   +-----------------------------------------------------+   |
|   | Book 1: Textbook Cover                              |   |
|   +-----------------------------------------------------+   |
|                                                             |
|   Book 3 completely obscures Book 2 from your sight.        |
|   You read Book 3. When finished, you set it aside (pop),   |
|   and Book 2 is still resting on the desk exactly where     |
|   you left it!                                              |
+-------------------------------------------------------------+
```

---

## 🧠 Internal Working
When you execute `Navigator.push()`, Flutter's internal engine executes this precise sequence:

```
                  User Taps List Tile / Button
                                │
                                ▼
         Navigator.push(context, MaterialPageRoute(...))
                                │
                                ▼
          NavigatorState.push<T>(Route<T> route)
                                │
                                ▼
           Route.install() -> Allocates AnimationController
                                │
                                ▼
         Route.didPush() -> Inserts OverlayEntry into Overlay
                                │
                                ▼
           Entrance Transition Plays (e.g. 300ms Slide/Fade)
                                │
                                ▼
          Previous Route marked as inactive (touches disabled)
                                │
                                ▼
     Returns Future<T?> that stays uncompleted until route.pop()
```

1. **Overlay Insertion**: `NavigatorState` creates an `OverlayEntry` wrapping the new route's widget and inserts it above the previous route's overlay entry.
2. **Focus Management**: The framework transfers primary keyboard and accessibility focus from the old screen to the new screen.
3. **Future Allocation**: A `Completer<T?>` is created internally by the route. The `Future` of this completer is returned by `Navigator.push()`. When `Navigator.pop(context, value)` is called on the child screen, the completer completes with that `value`.

---

## ⚙ Syntax

### Basic Push (Fire-and-Forget)
```dart
Navigator.push(
  context,
  MaterialPageRoute(
    builder: (BuildContext context) => const SettingsScreen(),
  ),
);
```

### Awaiting Return Data from the Pushed Route
```dart
final String? selectedColor = await Navigator.push<String>(
  context,
  MaterialPageRoute(
    builder: (BuildContext context) => const ColorPickerScreen(),
  ),
);

if (selectedColor != null) {
  print('User picked color: $selectedColor');
}
```

---

## 📋 Important Properties & Variants

| Method Variant | Purpose | Stack Impact | Example Use Case |
| :--- | :--- | :--- | :--- |
| `Navigator.push()` | Pushes a route on top of the stack. | $+1$ route to stack | Opening a product detail screen from a catalog. |
| `Navigator.pushNamed()` | Pushes a route using its string identifier (e.g. `'/details'`). | $+1$ route to stack | Deep linking or centralized route tables. |
| `Navigator.pushReplacement()`| Replaces the current top route with a new one. | Stack size remains identical | Transitioning from Splash Screen to Login Screen. |
| `Navigator.pushAndRemoveUntil()`| Pushes a route and deletes previous routes until a condition is met. | Clears previous routes | Logging out to the Login Screen; completing an order to Home. |

---

## 💻 Complete Runnable Example
The following production-ready application demonstrates:
1. Pushing a `UserProfileScreen` while passing data forward through its constructor.
2. Awaiting a response from the pushed screen.
3. Updating the parent screen's UI dynamically upon receiving the returned data.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const PushMasterclassApp());
}

class PushMasterclassApp extends StatelessWidget {
  const PushMasterclassApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Navigator.push Masterclass',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
        useMaterial3: true,
      ),
      home: const AccountSettingsScreen(),
    );
  }
}

// ---------------------------------------------------------------------------
// SCREEN 1: Account Settings (Caller Screen)
// ---------------------------------------------------------------------------
class AccountSettingsScreen extends StatefulWidget {
  const AccountSettingsScreen({super.key});

  @override
  State<AccountSettingsScreen> createState() => _AccountSettingsScreenState();
}

class _AccountSettingsScreenState extends State<AccountSettingsScreen> {
  String _displayName = 'Alex Morgan';
  String _jobTitle = 'Senior Mobile Architect';
  String _statusMessage = 'Available for consulting';

  Future<void> _editProfile() async {
    // Navigator.push returns a Future that completes when EditProfileScreen pops!
    final Map<String, String>? updatedData = await Navigator.push<Map<String, String>>(
      context,
      MaterialPageRoute(
        builder: (context) => EditProfileScreen(
          currentName: _displayName,
          currentJobTitle: _jobTitle,
        ),
      ),
    );

    // CRITICAL: Always guard with mounted check across async gap
    if (!mounted) return;

    if (updatedData != null) {
      setState(() {
        _displayName = updatedData['name'] ?? _displayName;
        _jobTitle = updatedData['jobTitle'] ?? _jobTitle;
      });

      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          content: Text('Profile updated for $_displayName!'),
          backgroundColor: Colors.indigo,
          behavior: SnackBarBehavior.floating,
        ),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Account Overview'),
        centerTitle: true,
      ),
      body: SafeArea(
        child: Padding(
          padding: const EdgeInsets.all(24.0),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: [
              // User Profile Header Card
              Card(
                elevation: 2,
                shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(16)),
                child: Padding(
                  padding: const EdgeInsets.all(20.0),
                  child: Row(
                    children: [
                      CircleAvatar(
                        radius: 36,
                        backgroundColor: Colors.indigo.shade100,
                        child: Text(
                          _displayName.isNotEmpty ? _displayName[0] : '?',
                          style: const TextStyle(fontSize: 28, fontWeight: FontWeight.bold, color: Colors.indigo),
                        ),
                      ),
                      const SizedBox(width: 16),
                      Expanded(
                        child: Column(
                          crossAxisAlignment: CrossAxisAlignment.start,
                          children: [
                            Text(
                              _displayName,
                              style: const TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
                            ),
                            const SizedBox(height: 4),
                            Text(
                              _jobTitle,
                              style: TextStyle(color: Colors.grey.shade700, fontSize: 14),
                            ),
                            const SizedBox(height: 4),
                            Text(
                              _statusMessage,
                              style: const TextStyle(color: Colors.green, fontSize: 12, fontWeight: FontWeight.w600),
                            ),
                          ],
                        ),
                      ),
                    ],
                  ),
                ),
              ),
              const SizedBox(height: 32),

              // Button to push Screen 2 and await changes
              FilledButton.icon(
                onPressed: _editProfile,
                icon: const Icon(Icons.edit),
                label: const Text('Edit Profile Details (push)', style: TextStyle(fontSize: 16)),
                style: FilledButton.styleFrom(
                  padding: const EdgeInsets.symmetric(vertical: 16),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

// ---------------------------------------------------------------------------
// SCREEN 2: Edit Profile (Pushed Screen)
// ---------------------------------------------------------------------------
class EditProfileScreen extends StatefulWidget {
  final String currentName;
  final String currentJobTitle;

  const EditProfileScreen({
    super.key,
    required this.currentName,
    required this.currentJobTitle,
  });

  @override
  State<EditProfileScreen> createState() => _EditProfileScreenState();
}

class _EditProfileScreenState extends State<EditProfileScreen> {
  late final TextEditingController _nameController;
  late final TextEditingController _jobController;

  @override
  void initState() {
    super.initState();
    _nameController = TextEditingController(text: widget.currentName);
    _jobController = TextEditingController(text: widget.currentJobTitle);
  }

  @override
  void dispose() {
    _nameController.dispose();
    _jobController.dispose();
    super.dispose();
  }

  void _saveAndReturn() {
    // Pop this screen and pass updated data back to AccountSettingsScreen
    Navigator.pop(context, {
      'name': _nameController.text.trim(),
      'jobTitle': _jobController.text.trim(),
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Edit Profile'),
      ),
      body: SafeArea(
        child: Padding(
          padding: const EdgeInsets.all(24.0),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: [
              TextField(
                controller: _nameController,
                decoration: const InputDecoration(
                  labelText: 'Full Name',
                  border: OutlineInputBorder(),
                  prefixIcon: Icon(Icons.person),
                ),
              ),
              const SizedBox(height: 16),
              TextField(
                controller: _jobController,
                decoration: const InputDecoration(
                  labelText: 'Job Title',
                  border: OutlineInputBorder(),
                  prefixIcon: Icon(Icons.work),
                ),
              ),
              const Spacer(),
              FilledButton(
                onPressed: _saveAndReturn,
                style: FilledButton.styleFrom(
                  padding: const EdgeInsets.symmetric(vertical: 16),
                ),
                child: const Text('Save & Return to Overview', style: TextStyle(fontSize: 16)),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

---

## 🔍 Line-by-Line Example Explanation
- `final Map<String, String>? updatedData = await Navigator.push<Map<String, String>>(...)`:
  - **What it does**: Pushes `EditProfileScreen` and pauses execution of `_editProfile` until `EditProfileScreen` pops.
  - **Type safety**: `<Map<String, String>>` specifies that this `Future` resolves to a strongly typed map or `null`.
- `if (!mounted) return;`:
  - **What it does**: Protects against calling `setState()` or `ScaffoldMessenger` if the user closed the app or navigated away while on Screen 2.
- `Navigator.pop(context, { ... })`:
  - **What it does**: Closes Screen 2 and delivers the data map into the waiting `await Navigator.push` line on Screen 1.
- `builder: (context) => EditProfileScreen(...)`:
  - **What it does**: Passes `_displayName` and `_jobTitle` directly into `EditProfileScreen`'s constructor.

---

## 📱 ASCII Phone Screen Preview

```
[ Screen 1: Account Overview ]              [ Screen 2: Edit Profile ]
+----------------------------+             +----------------------------+
|      Account Overview      |             | [←]  Edit Profile          |
+----------------------------+             +----------------------------+
|                            |   PUSH      |                            |
| +------------------------+ | ----------> | +------------------------+ |
| | (A) Alex Morgan        | |             | | Full Name              | |
| |     Senior Architect   | |             | | Alex Morgan|           | |
| |     Available          | |             | +------------------------+ |
| +------------------------+ |             |                            |
|                            |   POP       | +------------------------+ |
| [ Edit Profile Details ]   | <---------- | | Job Title              | |
|                            | (Sends Map) | | Senior Architect       | |
|                            |             | +------------------------+ |
|                            |             |                            |
|                            |             | [ Save & Return ]          |
+----------------------------+             +----------------------------+
```

---

## 🌳 Widget Tree
```
MaterialApp
 └── Navigator [State: NavigatorState, _history: [Route 1, Route 2]]
      └── Overlay
           ├── OverlayEntry (AccountSettingsScreen) [Waiting underneath]
           │    └── Scaffold
           │         ├── AppBar
           │         └── Column (Profile Card & Push Button)
           └── OverlayEntry (EditProfileScreen) [Pushed on top]
                └── Scaffold
                     ├── AppBar
                     └── Column (Text fields & Save Button)
```

---

## ⚙ Flutter Execution Flow
1. **User Action**: The user taps `[Edit Profile Details]`.
2. **Push**: `Navigator.push` runs, mounting `EditProfileScreen` as an `OverlayEntry`.
3. **Suspension**: The async `_editProfile()` function suspends execution at `await Navigator.push`.
4. **User Edits**: The user changes their job title to "Lead Mobile Architect" and taps `[Save & Return]`.
5. **Pop**: `Navigator.pop(context, data)` completes the internal route completer and animates Screen 2 off the screen.
6. **Resume**: `_editProfile()` resumes execution. `updatedData` receives the map.
7. **Repaint**: `setState()` triggers a rebuild of `AccountSettingsScreen`, displaying the updated title and showing a SnackBar.

---

## ⚠ Common Beginner Mistakes

### 1. Forgetting `await` When Expecting Return Data
```dart
// ❌ BUG: Execution doesn't wait!
void _pickItem() {
  final result = Navigator.push(context, MaterialPageRoute(...));
  print(result); // Prints: Instance of 'Future<dynamic>'! NOT the returned data!
}

// ✅ GOOD: Mark function async and await the result
void _pickItem() async {
  final result = await Navigator.push(context, MaterialPageRoute(...));
  print(result); // Prints the actual returned data!
}
```

### 2. Missing `if (!mounted) return;` After Awaiting Push
```dart
// ❌ LINTER WARNING / CRASH:
final data = await Navigator.push(context, route);
setState(() => _data = data); // May throw: setState() called after dispose()

// ✅ GOOD:
final data = await Navigator.push(context, route);
if (!mounted) return;
setState(() => _data = data);
```

### 3. Pushing Duplicate Screens on Rapid Double-Taps
```dart
// ❌ BUG: Rapid double-tapping pushes Screen 2 TWICE onto the stack!
onPressed: () {
  Navigator.push(context, MaterialPageRoute(builder: (_) => const NextScreen()));
}

// ✅ GOOD: Debounce button or check navigation state
```

---

## 🚀 Professional Usage
- **Checkout Flows (e.g., Shopify, Amazon)**: Pushing `PaymentMethodScreen`, awaiting the selected card token, and returning to the checkout review screen.
- **Search Filters (e.g., Airbnb, Booking.com)**: Pushing a filter screen, awaiting applied filters, and refreshing the search results list.
- **Permission Requests**: Pushing a dedicated permission explanation screen, awaiting user consent, and proceeding with camera access.

---

## 💡 Best Practices
1. **Pass data forward via constructors**: Avoid global singletons; passing data directly through widget constructors guarantees compile-time safety.
2. **Always specify generic return types**: Use `Navigator.push<T>()` so the returned value is strongly typed.
3. **Always check `mounted` after `await Navigator.push`**: Protects against calling `setState` on disposed state objects.
4. **Do not use `push` for login completion**: Use `pushReplacement` or `pushAndRemoveUntil` when transitioning after login so users cannot press back to return to the login screen.

---

## 🧪 Try It Yourself
1. Change `EditProfileScreen` to pop with `null` if the user cancels, and verify that Screen 1 leaves profile data unchanged.
2. Add a third text field for "Status Message" and pass all three fields across screens.
3. Test tapping the hardware back button on Android to verify that `Navigator.push` completes with `null`.

---

## 🎯 Challenge
Build a "Product Color & Size Customizer":
- Screen 1: Product Preview showing a T-Shirt with selected Color (Default: White) and Size (Default: Medium).
- Button: *"Customize Options"*.
- Screen 2: Customization Screen with Radio buttons for Color (Red, Blue, Green) and Size (S, M, L, XL).
- When the user taps *"Apply Customization"*, pop back to Screen 1 and render the T-shirt preview with the newly selected color and size!

---

## ❓ Practice Questions

### Q1. What does `Navigator.push()` return?
- A) `void`
- B) `bool`
- C) `Future<T?>`
- D) `Route<T>`
*Answer: C* — It returns a `Future` that resolves when the pushed route is popped off the stack.

### Q2. How does a child screen send a return value back to the `Navigator.push` caller?
- A) By assigning it to a global variable
- B) By passing the value as the second argument to `Navigator.pop(context, result)`
- C) By calling `Navigator.send(result)`
- D) It cannot be done in Flutter
*Answer: B* — `Navigator.pop(context, result)` completes the `Future` returned by `push`.

### Q3. What is the value of `result` if the user dismisses the pushed screen via the AppBar back arrow?
- A) `""` (empty string)
- B) `false`
- C) `null`
- D) It throws an exception
*Answer: C* — Standard back button dismissals pop the route with `null`.

### Q4. Why is `if (!mounted) return;` mandatory after `await Navigator.push(...)`?
- A) To clear text fields
- B) Because the calling widget might have unmounted while the child screen was open, and calling `setState` on an unmounted widget causes an exception
- C) It is required by the Dart VM to garbage collect
- D) To close the virtual keyboard
*Answer: B* — Guards against executing state operations across asynchronous gaps on disposed widgets.

### Q5. What happens to the underlying screen when a new screen is pushed on top?
- A) It is destroyed and deleted from memory
- B) It remains alive in memory underneath the new screen in an inactive state
- C) It converts into a background service
- D) Its state is cleared
*Answer: B* — By default (`maintainState: true`), inactive routes keep their state intact.

### Q6. What is the difference between `Navigator.push` and `Navigator.pushNamed`?
*Answer*: `Navigator.push` takes an explicit `Route` object (like `MaterialPageRoute`), whereas `Navigator.pushNamed` takes a string route path (like `'/settings'`) defined in a central route table.

### Q7. Output Prediction: What will be printed if `_screenTwo` calls `Navigator.pop(context, 'Success')`?
```dart
final res = await Navigator.push(context, MaterialPageRoute(builder: (_) => const ScreenTwo()));
print(res);
```
*Answer*: `'Success'`.

### Q8. How do you pass data forward to a new screen when using `Navigator.push`?
*Answer*: Pass the data directly into the destination widget's constructor inside the `builder` callback (e.g. `MaterialPageRoute(builder: (ctx) => DetailScreen(item: myItem))`).

### Q9. Can you push multiple screens sequentially onto the navigation stack?
*Answer*: Yes, each `Navigator.push` adds another layer on top of the stack.

### Q10. Write a minimal code snippet pushing `SettingsScreen` and ignoring its return value.
*Answer*:
```dart
Navigator.push(
  context,
  MaterialPageRoute(builder: (context) => const SettingsScreen()),
);
```

---

## 📝 Summary
- [Navigator.push()](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/05_Navigation/04_Push.md) adds a new screen to the top of the navigation stack.
- It returns a `Future<T?>` that completes when the pushed screen is popped.
- Pass data forward via widget constructors; pass data backward via `Navigator.pop(context, result)`.
- Always guard with `if (!mounted) return;` when performing state mutations after `await Navigator.push()`.
