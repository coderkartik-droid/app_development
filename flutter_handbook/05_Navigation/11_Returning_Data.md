# Returning Data from a Screen in Flutter

## 📖 Introduction
In mobile applications, user interactions often follow a **picker or query pattern**: 
- A user wants to set an account profile picture, so you open the photo gallery screen and wait for them to pick an image.
- A user wants to select a shipping address at checkout, so you open the address book screen and wait for them to choose an address.
- A user enters a form to add a new contact, and you wait for them to hit "Save".

In Flutter, returning data from a child screen back to its parent caller does not require complex state management solutions, event buses, or global stores. The [Navigator](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/05_Navigation/02_Navigator.md) architecture was built from day one as an **asynchronous request-response system**. Calling `Navigator.push()` yields a Dart `Future`, and calling `Navigator.pop(context, result)` fulfills that `Future` with the returned data.

---

## 🤔 Why Do We Need It?
1. **Decoupled Architecture**: Screen A does not need to pass a callback function or reference itself to Screen B. Screen B simply finishes its job and pops with a result.
2. **Standard Dart Concurrency**: Uses standard `async/await` syntax that every Dart and Flutter developer already understands.
3. **Graceful Cancellation Handling**: If the user dismisses the screen using the hardware back button or AppBar back arrow, the `Future` completes with `null`, making cancellation trivial to handle.
4. **Dart 3 Records Support**: You can effortlessly return multiple values simultaneously using Dart 3 records (e.g. `Navigator.pop(context, (name, age))`).

---

## 🌍 Real-Life Analogy
Think of returning data from a screen as **Sending an Assistant to the Store**:

```
+-------------------------------------------------------------+
|                      ERRAND RUNNER ANALOGY                  |
|                                                             |
|   You (Parent Screen):                                      |
|   "Hey assistant, please go to the bakery and bring back    |
|   a loaf of sourdough bread."                               |
|   -> You wait asynchronously at home (await Navigator.push).|
|                                                             |
|   Assistant (Child Screen):                                 |
|   Runs the errand at the bakery (User interacts with UI).   |
|                                                             |
|   Outcome A (Success):                                      |
|   Returns through the front door handing you the bread!     |
|   -> Navigator.pop(context, breadLoaf)                      |
|                                                             |
|   Outcome B (Cancelled / Bakery Closed):                    |
|   Returns empty-handed.                                     |
|   -> Navigator.pop(context, null)                           |
+-------------------------------------------------------------+
```

---

## 🧠 Internal Working
To appreciate how Flutter fulfills return values without memory leaks, look inside the `Route` and `Completer` implementation:

```
                  Parent Screen calls:
         final result = await Navigator.push<T>(context, route);
                                │
                                ▼
         1. Route creates internal Completer<T?> _popCompleter
         2. Returns _popCompleter.future to the caller
                                │
                                ▼
         User interacts with Child Screen
                                │
                                ▼
                  Child Screen calls:
              Navigator.pop(context, myData);
                                │
                                ▼
         3. Route executes: _popCompleter.complete(myData)
         4. Plays exit transition animation
         5. Unmounts Child Screen elements
                                │
                                ▼
         Parent Screen resumes execution:
         result receives myData!
```

1. **The `Completer<T?>` Bridge**: The connection between the pushing screen and the popping screen is an internal Dart `Completer`.
2. **Type Safety with Generics**: By providing the type argument `Navigator.push<T>()`, you ensure the compiler knows the type of `result`.
3. **System Back Gestures**: When the user presses the system back button on Android or swipes the left edge on iOS, Flutter calls `Navigator.maybePop(context)` with **no arguments**, which automatically completes the `Future` with `null`.

---

## ⚙ Syntax

### 1. Pushing and Awaiting a Result
```dart
final String? selectedItem = await Navigator.push<String>(
  context,
  MaterialPageRoute(builder: (context) => const SelectionScreen()),
);

if (selectedItem != null) {
  // User selected an item
}
```

### 2. Popping and Delivering the Result
```dart
// Deliver a single value
Navigator.pop(context, 'Selected Option A');

// Deliver a custom class model
Navigator.pop(context, Contact(name: 'Sarah', phone: '555-0192'));

// Deliver multiple values using Dart 3 Record
Navigator.pop(context, (selectedCategory, maxPrice));
```

---

## 📋 Return Types & Patterns Comparison

| Return Pattern | Syntax Example | When to Use |
| :--- | :--- | :--- |
| **Primitive Value** | `Navigator.pop(context, 42)` | Simple IDs, index numbers, booleans, string choices. |
| **Custom Class Model** | `Navigator.pop(context, Address(...))` | Rich structured data like an address, user profile, or order. |
| **Dart 3 Record (Tuple)**| `Navigator.pop(context, ('USD', 100.0))`| Returning 2-3 disparate values without creating a throwaway class. |
| **Nullable / Cancel** | `Navigator.pop(context, null)` | User tapped "Cancel" or backed out without making a choice. |

---

## 💻 Complete Runnable Example
The following production-ready application demonstrates:
1. Pushing a `LanguagePickerScreen` to choose an app language.
2. Returning a strongly typed `AppLanguage` object.
3. Handling user cancellation safely with null-checks and `context.mounted`.
4. Updating the parent screen dynamically with the chosen language.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ReturningDataMasterclassApp());
}

// 1. Strongly typed model representing selectable options
class AppLanguage {
  final String name;
  final String code;
  final String flagEmoji;

  const AppLanguage({
    required this.name,
    required this.code,
    required this.flagEmoji,
  });
}

class ReturningDataMasterclassApp extends StatelessWidget {
  const ReturningDataMasterclassApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Returning Data Guide',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.teal),
        useMaterial3: true,
      ),
      home: const SettingsHomeScreen(),
    );
  }
}

// ---------------------------------------------------------------------------
// SCREEN 1: Settings Home (Awaits Result)
// ---------------------------------------------------------------------------
class SettingsHomeScreen extends StatefulWidget {
  const SettingsHomeScreen({super.key});

  @override
  State<SettingsHomeScreen> createState() => _SettingsHomeScreenState();
}

class _SettingsHomeScreenState extends State<SettingsHomeScreen> {
  // Current active language selection
  AppLanguage _currentLanguage = const AppLanguage(
    name: 'English (US)',
    code: 'en_US',
    flagEmoji: '🇺🇸',
  );

  Future<void> _selectLanguage() async {
    // 1. PUSH LanguagePickerScreen and AWAIT the return value!
    final AppLanguage? selectedLang = await Navigator.push<AppLanguage>(
      context,
      MaterialPageRoute(
        builder: (context) => LanguagePickerScreen(currentlySelectedCode: _currentLanguage.code),
      ),
    );

    // 2. CRITICAL: Verify mounted status across async gap
    if (!mounted) return;

    // 3. Handle result or cancellation
    if (selectedLang != null) {
      setState(() {
        _currentLanguage = selectedLang;
      });

      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          content: Text('Language switched to ${selectedLang.flagEmoji} ${selectedLang.name}!'),
          backgroundColor: Colors.teal,
          behavior: SnackBarBehavior.floating,
        ),
      );
    } else {
      // User tapped back or cancel
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(
          content: Text('Language selection cancelled.'),
          behavior: SnackBarBehavior.floating,
        ),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('App Preferences'),
        centerTitle: true,
      ),
      body: SafeArea(
        child: Padding(
          padding: const EdgeInsets.all(24.0),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: [
              Card(
                elevation: 0,
                color: Colors.teal.shade50,
                shape: RoundedRectangleBorder(
                  borderRadius: BorderRadius.circular(16),
                  side: BorderSide(color: Colors.teal.shade200),
                ),
                child: Padding(
                  padding: const EdgeInsets.all(20.0),
                  child: Row(
                    children: [
                      Text(
                        _currentLanguage.flagEmoji,
                        style: const TextStyle(fontSize: 48),
                      ),
                      const SizedBox(width: 16),
                      Expanded(
                        child: Column(
                          crossAxisAlignment: CrossAxisAlignment.start,
                          children: [
                            const Text('Active App Language:', style: TextStyle(color: Colors.grey, fontSize: 13)),
                            Text(
                              _currentLanguage.name,
                              style: const TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
                            ),
                            Text(
                              'Locale Code: ${_currentLanguage.code}',
                              style: const TextStyle(fontFamily: 'monospace', fontSize: 12),
                            ),
                          ],
                        ),
                      ),
                    ],
                  ),
                ),
              ),
              const SizedBox(height: 32),

              FilledButton.icon(
                onPressed: _selectLanguage,
                icon: const Icon(Icons.language),
                label: const Text('Change Language (await return data)', style: TextStyle(fontSize: 16)),
                style: FilledButton.styleFrom(padding: const EdgeInsets.symmetric(vertical: 16)),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

// ---------------------------------------------------------------------------
// SCREEN 2: Language Picker (Delivers Result via Navigator.pop)
// ---------------------------------------------------------------------------
class LanguagePickerScreen extends StatelessWidget {
  final String currentlySelectedCode;

  const LanguagePickerScreen({super.key, required this.currentlySelectedCode});

  static const List<AppLanguage> supportedLanguages = [
    AppLanguage(name: 'English (US)', code: 'en_US', flagEmoji: '🇺🇸'),
    AppLanguage(name: 'Spanish (Español)', code: 'es_ES', flagEmoji: '🇪🇸'),
    AppLanguage(name: 'French (Français)', code: 'fr_FR', flagEmoji: '🇫🇷'),
    AppLanguage(name: 'German (Deutsch)', code: 'de_DE', flagEmoji: '🇩🇪'),
    AppLanguage(name: 'Japanese (日本語)', code: 'ja_JP', flagEmoji: '🇯🇵'),
    AppLanguage(name: 'Hindi (हिन्दी)', code: 'hi_IN', flagEmoji: '🇮🇳'),
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Select Language'),
        actions: [
          // Explicit Cancel button
          TextButton(
            onPressed: () => Navigator.pop(context, null), // Delivers null!
            child: const Text('Cancel', style: TextStyle(color: Colors.redAccent)),
          ),
        ],
      ),
      body: SafeArea(
        child: ListView.separated(
          padding: const EdgeInsets.all(16.0),
          itemCount: supportedLanguages.length,
          separatorBuilder: (_, __) => const SizedBox(height: 8),
          itemBuilder: (context, index) {
            final lang = supportedLanguages[index];
            final bool isSelected = lang.code == currentlySelectedCode;

            return Card(
              elevation: isSelected ? 2 : 0,
              color: isSelected ? Colors.teal.shade50 : Colors.white,
              shape: RoundedRectangleBorder(
                borderRadius: BorderRadius.circular(12),
                side: BorderSide(
                  color: isSelected ? Colors.teal : Colors.grey.shade300,
                  width: isSelected ? 2 : 1,
                ),
              ),
              child: ListTile(
                leading: Text(lang.flagEmoji, style: const TextStyle(fontSize: 32)),
                title: Text(lang.name, style: const TextStyle(fontWeight: FontWeight.bold)),
                subtitle: Text('Code: ${lang.code}'),
                trailing: isSelected
                    ? const Icon(Icons.check_circle, color: Colors.teal)
                    : const Icon(Icons.circle_outlined, color: Colors.grey),
                onTap: () {
                  // CRITICAL: Pop this screen and return the chosen AppLanguage object!
                  Navigator.pop<AppLanguage>(context, lang);
                },
              ),
            );
          },
        ),
      ),
    );
  }
}
```

---

## 🔍 Line-by-Line Example Explanation
- `final AppLanguage? selectedLang = await Navigator.push<AppLanguage>(...)`:
  - **What it does**: Pushes `LanguagePickerScreen` and pauses execution.
  - **Type safety**: Notice the generic `<AppLanguage>`. The compiler guarantees `selectedLang` is typed as `AppLanguage?`.
- `Navigator.pop<AppLanguage>(context, lang)`:
  - **What it does**: Pops `LanguagePickerScreen` off the stack and passes `lang` directly into the waiting `_popCompleter`.
- `if (!mounted) return;`:
  - **What it does**: Ensures the `SettingsHomeScreen` widget is still mounted in the tree before calling `setState()` or displaying a SnackBar.
- `onPressed: () => Navigator.pop(context, null)`:
  - **What it does**: Allows explicit cancellation, passing `null` so the caller knows the user deliberately opted out.

---

## 📱 ASCII Phone Screen Preview

```
[ Screen 1: App Preferences ]              [ Screen 2: Select Language ]
+----------------------------+             +----------------------------+
|      App Preferences       |             | [←] Select Language [Cancel|
+----------------------------+             +----------------------------+
|                            |    PUSH     |                            |
|  +----------------------+  | ----------> | [🇺🇸] English (US)    (•)   |
|  | 🇺🇸 English (US)      |  |             |                            |
|  | Locale: en_US        |  |    POP      | [🇪🇸] Spanish          ( )   |
|  +----------------------+  | <---------- |                            |
|                            | (Sends lang)| [🇯🇵] Japanese         ( )   |
|  [ Change Language ]       |             |                            |
|                            |             | [🇮🇳] Hindi            ( )   |
+----------------------------+             +----------------------------+
```

---

## 🌳 Widget Tree
```
MaterialApp
 └── Navigator [State: NavigatorState]
      └── Overlay
           ├── OverlayEntry (SettingsHomeScreen) [Waiting on Future<AppLanguage?>]
           └── OverlayEntry (LanguagePickerScreen) [When pushed: calls Navigator.pop(context, lang)]
```

---

## ⚙ Flutter Execution Flow
1. **User Action**: The user taps `[Change Language]`.
2. **Push & Await**: `Navigator.push<AppLanguage>` runs. The async method `_selectLanguage()` suspends execution.
3. **Selection**: Inside `LanguagePickerScreen`, the user taps "Spanish (Español)".
4. **Pop With Data**: `Navigator.pop(context, lang)` executes.
5. **Route Teardown**: `LanguagePickerScreen` slides out and unmounts.
6. **Resume**: `_selectLanguage()` resumes. `selectedLang` receives the Spanish `AppLanguage` object.
7. **UI Update**: `setState()` updates `_currentLanguage` to Spanish, repainting the card with 🇪🇸 and displaying a floating SnackBar.

---

## ⚠ Common Beginner Mistakes

### 1. Forgetting `await` on `Navigator.push`
```dart
// ❌ WRONG: Method doesn't wait!
void _pickColor() {
  final color = Navigator.push(context, route);
  print(color); // Prints: Instance of 'Future<Color?>'! Not the actual color!
}

// ✅ CORRECT: Use async and await
void _pickColor() async {
  final color = await Navigator.push(context, route);
  print(color); // Prints the actual Color!
}
```

### 2. Not Handling `null` When User Cancels
```dart
// ❌ CRASH: Assuming user ALWAYS picks an option
final selected = await Navigator.push(context, route);
print(selected.name); // Null check operator error if user tapped Android back button!

// ✅ GOOD: Always null-check the returned result
if (selected != null) {
  print(selected.name);
}
```

### 3. Missing `if (!mounted) return;`
```dart
// ❌ DANGEROUS:
final item = await Navigator.push(context, route);
setState(() => _item = item); // May throw if widget was unmounted!

// ✅ GOOD:
final item = await Navigator.push(context, route);
if (!mounted) return;
setState(() => _item = item);
```

---

## 🚀 Professional Usage
- **Address & Location Pickers (e.g., Uber, DoorDash)**: Opening Google Places search, picking an address, and returning the `LatLng` coordinate model.
- **Media Uploaders (e.g., Instagram, WhatsApp)**: Opening an image gallery screen and returning the selected `List<File>`.
- **Date & Range Pickers**: Modal bottom sheets returning selected `DateTimeRange` models.

---

## 💡 Best Practices
1. **Always specify generic type parameters**: Write `Navigator.push<MyType>()` and `Navigator.pop<MyType>()` to guarantee compile-time type safety.
2. **Always treat the return value as nullable**: The user can *always* tap the system back button or gesture back, which delivers `null`.
3. **Use Dart 3 Records for multi-value returns**: Instead of creating a throwaway class just to return two strings, return a record: `Navigator.pop(context, (firstName, lastName))`.
4. **Always guard with `if (!mounted) return;`**: Mandatory before executing `setState()` or `ScaffoldMessenger` following an `await Navigator.push` line.

---

## 🧪 Try It Yourself
1. Add a new language (e.g. Italian 🇮🇹) to `supportedLanguages` and verify that selecting it updates the home screen.
2. Tap the Android hardware back button and observe how the SnackBar cleanly reports "Language selection cancelled".
3. Return a Dart 3 record `(AppLanguage, DateTime)` to also return the exact timestamp when the user made the selection.

---

## 🎯 Challenge
Build a "Contact Card Creator":
- Screen 1: Displays an empty contact card with a button *"Add Contact"*.
- Screen 2: Form with `Name`, `Phone`, and `Email` text fields.
- When the user taps *"Save Contact"*, pop Screen 2 and return a `Contact` object.
- If any field is empty, prevent popping and show a validation error.
- When popped, display the full contact card on Screen 1!

---

## ❓ Practice Questions

### Q1. How does `Navigator.push` receive data returned by `Navigator.pop`?
- A) Through a global variable
- B) Through a Dart `Future` that completes when `pop(context, result)` is called
- C) Through an SQLite table
- D) Via an HTTP webhook
*Answer: B* — `Navigator.push` returns a `Future` that resolves with the value passed to `pop`.

### Q2. What value does `await Navigator.push(...)` resolve to if the user presses the system back button?
- A) `""`
- B) `false`
- C) `null`
- D) It throws an uncaught exception
*Answer: C* — Unhandled back navigation dismisses the route with `null`.

### Q3. How do you return a value from a screen using `Navigator.pop`?
- A) `Navigator.pop(context, myValue)`
- B) `Navigator.return(context, myValue)`
- C) `Navigator.push(context, myValue)`
- D) `Navigator.send(myValue)`
*Answer: A* — Pass the value as the second parameter to `Navigator.pop()`.

### Q4. Why is `if (!mounted) return;` critical after `await Navigator.push(...)`?
- A) It prevents the app from pausing
- B) The calling widget might have been removed from the tree while the child screen was open; calling `setState()` on an unmounted state throws an exception
- C) It is required by the garbage collector
- D) To close the virtual keyboard
*Answer: B* — Guards against executing state operations across asynchronous gaps.

### Q5. Can you return a Dart 3 record (tuple) like `('USD', 100.0)` from `Navigator.pop`?
- A) No, Flutter only supports primitive strings and ints
- B) Yes, `Navigator.pop` accepts any Dart object including modern records
- C) Only on Android
- D) Only in debug mode
*Answer: B* — Dart 3 records are first-class objects and work seamlessly with `Navigator.pop`.

### Q6. What does specifying `<String>` in `Navigator.push<String>(...)` do?
*Answer*: It strongly types the returned `Future<String?>`, ensuring compile-time type safety so the caller doesn't have to deal with untyped `dynamic`.

### Q7. Output Prediction: What will `print(result)` display if Screen 2 calls `Navigator.pop(context, 42)`?
```dart
final result = await Navigator.push<int>(context, route);
print(result);
```
*Answer*: `42`.

### Q8. What happens if you call `Navigator.pop(context)` without passing a second argument?
*Answer*: It defaults to popping with `null`.

### Q9. Can you return complex domain models (like `UserProfile` or `CartItem`) through `Navigator.pop`?
*Answer*: Yes, any custom class instance can be passed directly as the result.

### Q10. Write a minimal code snippet pushing `PickerScreen` and printing the returned string if not null.
*Answer*:
```dart
final result = await Navigator.push<String>(
  context,
  MaterialPageRoute(builder: (context) => const PickerScreen()),
);
if (result != null) print(result);
```

---

## 📝 Summary
- Returning data from a screen in Flutter uses an asynchronous **request-response model** via `Navigator.push` and `Navigator.pop(context, result)`.
- The caller awaits `Future<T?>`; `null` indicates cancellation or system back navigation.
- Always check `if (!mounted) return;` after `await Navigator.push`.
- Use custom class models or Dart 3 records to return rich, structured data cleanly.
