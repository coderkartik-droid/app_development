# TextField

## 📖 Introduction

If you ask any mobile application user what they interact with the most, after tapping buttons, the answer is undoubtedly typing into text boxes.

Whether you are logging into an account, searching for a product, composing a WhatsApp message, or entering a credit card number, the fundamental building block is the text entry field.

In Flutter, the primary, foundational widget for capturing text from a user’s keyboard is the **`TextField`**.

A `TextField` is a rich, highly customizable input box. It handles cursor blinking, text selection, copy-pasting, keyboard triggering, obscuring passwords, character limits, custom styling, and native platform interactions effortlessly.

---

## 🤔 Why Do We Need It?

Without `TextField`, capturing text on a mobile touchscreen would require immense low-level engineering:

1. **Managing the Native Keyboard:** You would have to write custom native Kotlin/Java code for Android and Swift/Objective-C code for iOS to summon the keyboard, listen to keycodes, handle autocorrect, and detect backspaces. `TextField` handles this automatically across Android, iOS, Web, Windows, macOS, and Linux.
2. **Text Selection and Gestures:** Double-tapping to select a word, dragging cursor handles, showing the native context menu (Cut, Copy, Paste), and magnifying text are all built directly into `TextField`.
3. **Rich Visual Styling:** A raw input line is ugly. `TextField` integrates with `InputDecoration` to support floating labels, placeholder hints, prefix icons, helper texts, and custom border states (focused, error, disabled).

---

## 🌍 Real-Life Analogy

Think of **a blank line on a physical bank slip with a blinking guide cursor**:

```text
 ┌────────────────────────────────────────────────────────┐
 │ Account Number:                                        │
 │ ┌────────────────────────────────────────────────────┐ │
 │ │ 4920 1842 0912 |                                   │ │ <--- The Blinking Cursor
 │ └────────────────────────────────────────────────────┘ │
 └────────────────────────────────────────────────────────┘
```

* When you write on a paper slip, the blank rectangle defines where your pen writes.
* If you make a mistake, you use an eraser (backspace).
* If you want to insert a word in the middle, you point your finger there.
* The `TextField` is that interactive blank line: it guides the user, tracks the exact insertion point (the cursor), and stores the ink as digital text.

---

## 🧠 Internal Working

What happens inside the Flutter Engine when a `TextField` is placed on the screen?

```text
Parent Widget
     │
     ▼
[TextField]
     │
     ├─► 1. Creates an EditableText Widget Under the Hood:
     │      TextField is actually a Material Design wrapper around EditableText.
     │      EditableText manages the cursor, selection, and platform keyboard channel.
     │
     ├─► 2. Instantiates RenderEditable:
     │      A specialized RenderBox that measures text metrics, fonts, line breaks,
     │      and draws the blinking vertical cursor bar at 500ms intervals.
     │
     ├─► 3. Establishes TextInputConnection:
     │      When focused, communicates with the operating system to request the
     │      appropriate software keyboard (e.g. text, numeric, email).
     │
     ├─► 4. Listens to Raw Keystrokes & IME (Input Method Editor):
     │      Supports complex language input (Japanese, Chinese, Arabic, emojis)
     │      and predictive typing suggestions.
     │
     ▼
Canvas paints the decorated border, label, text glyphs, and blinking cursor
```

---

## ⚙ Syntax

The constructor of `TextField`:

```dart
TextField({
  Key? key,
  TextEditingController? controller,
  FocusNode? focusNode,
  InputDecoration? decoration = const InputDecoration(),
  TextInputType? keyboardType,
  TextInputAction? textInputAction,
  TextCapitalization textCapitalization = TextCapitalization.none,
  TextStyle? style,
  TextAlign textAlign = TextAlign.start,
  bool autofocus = false,
  bool readOnly = false,
  bool? showCursor,
  bool obscureText = false,
  String obscuringCharacter = '•',
  bool autocorrect = true,
  bool enableSuggestions = true,
  int? maxLines = 1,
  int? minLines,
  bool expands = false,
  int? maxLength,
  ValueChanged<String>? onChanged,
  VoidCallback? onEditingComplete,
  ValueChanged<String>? onSubmitted,
  List<TextInputFormatter>? inputFormatters,
  bool? enabled,
  double cursorWidth = 2.0,
  double? cursorHeight,
  Radius? cursorRadius,
  Color? cursorColor,
  Brightness? keyboardAppearance,
  EdgeInsets scrollPadding = const EdgeInsets.all(20.0),
  bool enableInteractiveSelection = true,
  TapRegionCallback? onTapOutside,
})
```

---

## 📋 Important Properties

| Property | Type | Default Value | Purpose |
|---|---|---|---|
| `controller` | `TextEditingController?` | `null` | Controls the text being edited and cursor position. |
| `decoration` | `InputDecoration?` | `InputDecoration()` | Visual border, label, hint, icons, and counter styling. |
| `keyboardType` | `TextInputType?` | `TextInputType.text`| Type of keyboard to display (email, phone, number, url). |
| `textInputAction`| `TextInputAction?` | `TextInputAction.done`| Action button on the keyboard (search, next, done, send). |
| `obscureText` | `bool` | `false` | Masks text with dots (`•`), essential for passwords. |
| `autofocus` | `bool` | `false` | Automatically focuses and opens the keyboard on screen launch. |
| `readOnly` | `bool` | `false` | Prevents keyboard from opening; user can copy but not edit. |
| `maxLines` | `int?` | `1` | Max vertical lines allowed (set to `null` for multiline expanding). |
| `maxLength` | `int?` | `null` | Enforces a strict character limit with a built-in counter. |
| `onChanged` | `ValueChanged<String>?` | `null` | Fired on every single keystroke. |
| `onSubmitted` | `ValueChanged<String>?` | `null` | Fired when the user taps the keyboard action button (Done/Enter). |
| `onTapOutside` | `TapRegionCallback?` | `null` | Perfect for dismissing the keyboard when tapping outside the box. |

---

## 💻 Complete Runnable Example

Below is a complete, runnable Flutter app showcasing `TextField` in three essential real-world forms: an email input with action triggers, a secure password field with visibility toggle, and a multiline notes feedback area.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const TextFieldDemoApp());
}

class TextFieldDemoApp extends StatelessWidget {
  const TextFieldDemoApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: TextFieldDemoScreen(),
    );
  }
}

class TextFieldDemoScreen extends StatefulWidget {
  const TextFieldDemoScreen({super.key});

  @override
  State<TextFieldDemoScreen> createState() => _TextFieldDemoScreenState();
}

class _TextFieldDemoScreenState extends State<TextFieldDemoScreen> {
  final TextEditingController _emailController = TextEditingController();
  final TextEditingController _passwordController = TextEditingController();
  final TextEditingController _notesController = TextEditingController();

  bool _isPasswordHidden = true;

  @override
  void dispose() {
    _emailController.dispose();
    _passwordController.dispose();
    _notesController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFFF8FAFC),
      appBar: AppBar(
        title: const Text(
          'TextField Deep Dive',
          style: TextStyle(fontWeight: FontWeight.bold),
        ),
        backgroundColor: Colors.blue.shade900,
        foregroundColor: Colors.white,
        elevation: 2,
      ),
      body: SafeArea(
        child: SingleChildScrollView(
          padding: const EdgeInsets.all(20.0),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              _buildSectionHeader('1. Email Field (Keyboard Type & Action)'),
              const SizedBox(height: 8),

              // TextField 1: Email Input
              TextField(
                controller: _emailController,
                keyboardType: TextInputType.emailAddress,
                textInputAction: TextInputAction.next,
                decoration: InputDecoration(
                  labelText: 'Email Address',
                  hintText: 'alex.morgan@example.com',
                  prefixIcon: const Icon(Icons.email_outlined),
                  filled: true,
                  fillColor: Colors.white,
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(12),
                  ),
                ),
              ),

              const SizedBox(height: 24),

              _buildSectionHeader('2. Password Field (Obscured with Toggle)'),
              const SizedBox(height: 8),

              // TextField 2: Secure Password Input
              TextField(
                controller: _passwordController,
                obscureText: _isPasswordHidden,
                textInputAction: TextInputAction.next,
                decoration: InputDecoration(
                  labelText: 'Password',
                  hintText: 'Enter at least 8 characters',
                  prefixIcon: const Icon(Icons.lock_outline),
                  suffixIcon: IconButton(
                    icon: Icon(
                      _isPasswordHidden ? Icons.visibility_off : Icons.visibility,
                      color: Colors.grey,
                    ),
                    onPressed: () {
                      setState(() {
                        _isPasswordHidden = !_isPasswordHidden;
                      });
                    },
                  ),
                  filled: true,
                  fillColor: Colors.white,
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(12),
                  ),
                ),
              ),

              const SizedBox(height: 24),

              _buildSectionHeader('3. Multiline Feedback Notes (Expanding)'),
              const SizedBox(height: 8),

              // TextField 3: Multiline Input with Max Length
              TextField(
                controller: _notesController,
                maxLines: 4,
                maxLength: 150,
                keyboardType: TextInputType.multiline,
                textCapitalization: TextCapitalization.sentences,
                decoration: InputDecoration(
                  labelText: 'Delivery Notes',
                  hintText: 'Provide gate code, building number, or drop-off instructions...',
                  alignLabelWithHint: true, // Aligns label to top-left in multiline
                  prefixIcon: const Padding(
                    padding: EdgeInsets.only(bottom: 60.0),
                    child: Icon(Icons.note_alt_outlined),
                  ),
                  filled: true,
                  fillColor: Colors.white,
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(12),
                  ),
                ),
              ),

              const SizedBox(height: 24),

              // Submit Button demonstrating reading controller values
              SizedBox(
                width: double.infinity,
                height: 50,
                child: ElevatedButton(
                  style: ElevatedButton.styleFrom(
                    backgroundColor: Colors.blue.shade900,
                    foregroundColor: Colors.white,
                    shape: RoundedRectangleBorder(
                      borderRadius: BorderRadius.circular(12),
                    ),
                  ),
                  onPressed: () {
                    // Read text from controllers
                    final email = _emailController.text.trim();
                    final password = _passwordController.text;
                    final notes = _notesController.text.trim();

                    ScaffoldMessenger.of(context).showSnackBar(
                      SnackBar(
                        content: Text('Submitted: $email | Notes: ${notes.length} chars'),
                      ),
                    );
                  },
                  child: const Text('Save Details', style: TextStyle(fontSize: 16)),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }

  static Widget _buildSectionHeader(String title) {
    return Text(
      title,
      style: const TextStyle(fontSize: 14, fontWeight: FontWeight.bold),
    );
  }
}
```

---

## 🔍 Line-by-Line Example Explanation

### 1. `keyboardType: TextInputType.emailAddress`
* **What it does:** Configures the native mobile keyboard to show the `@` symbol and `.com` shortcut keys directly on the primary keyboard view.
* **Why it exists:** Prevents the user from having to tap the symbols button to find the `@` sign.
* **What Flutter does internally:** Passes `TextInputType.emailAddress` over the platform channel to the Android `InputType` or iOS `UIKeyboardType`.
* **What happens if removed:** Defaults to plain text; user has to search for the `@` key.

### 2. `textInputAction: TextInputAction.next`
* **What it does:** Replaces the generic "Return" key on the software keyboard with a blue "Next" arrow icon.
* **Why it exists:** Guides the user smoothly to the next field when tapped.
* **What Flutter does internally:** Configures the keyboard's IME action identifier.
* **What happens if removed:** Defaults to `done`, which closes the keyboard immediately.

### 3. `obscureText: _isPasswordHidden`
* **What it does:** Masks the typed characters into bullet dots (`••••••••`).
* **Why it exists:** Protects passwords from shoulder surfers viewing the screen in public.
* **What Flutter does internally:** Replaces glyph indices in the text layout painter with the `obscuringCharacter` (`•`).
* **What happens if removed:** The password is displayed in plain readable text.

### 4. `maxLines: 4` and `textCapitalization: TextCapitalization.sentences`
* **What it does:** Expands the text field to a height of 4 lines and capitalizes the first letter of each sentence automatically.
* **Why it exists:** Multiline notes need vertical space, and automatic capitalization ensures professional, clean text input.
* **What Flutter does internally:** Sets minimum height constraints to 4 lines of font metrics and instructs the native keyboard to capitalize after periods.
* **What happens if removed:** Defaults to single-line horizontal scrolling; text does not wrap.

---

## 📱 ASCII Phone Screen Preview

```text
+------------------------------------------+
| TextField Deep Dive                      |
+------------------------------------------+
| 1. Email Field                           |
| +--------------------------------------+ |
| | (✉) Email Address                    | |
| |     alex.morgan@example.com          | |
| +--------------------------------------+ |
|                                          |
| 2. Password Field                        |
| +--------------------------------------+ |
| | (🔒) Password                   (👁) | |
| |      ••••••••••••                    | | <--- Tapping eye reveals text!
| +--------------------------------------+ |
|                                          |
| 3. Multiline Feedback Notes              |
| +--------------------------------------+ |
| | (📝) Delivery Notes                  | |
| |      Gate code is #4821. Please      | |
| |      leave package at the porch...   | |
| |                                      | |
| |                              48 / 150| | <--- Built-in character counter
| +--------------------------------------+ |
|                                          |
| [            Save Details              ] |
+------------------------------------------+
```

---

## 🌳 Widget Tree

```text
TextFieldDemoApp
└── MaterialApp
    └── TextFieldDemoScreen (StatefulWidget)
        └── Scaffold
            ├── AppBar
            │   └── Text ("TextField Deep Dive")
            └── SafeArea
                └── SingleChildScrollView
                    └── Column
                        ├── TextField (Email: controller: _emailController)
                        │   └── InputDecoration (labelText, prefixIcon)
                        ├── TextField (Password: obscureText: true)
                        │   └── InputDecoration (suffixIcon: IconButton)
                        ├── TextField (Notes: maxLines: 4, maxLength: 150)
                        │   └── InputDecoration (alignLabelWithHint: true)
                        └── SizedBox (width: double.infinity, height: 50)
                            └── ElevatedButton ("Save Details")
```

---

## ⚙ Flutter Execution Flow

```text
1. User taps Password TextField
   │
2. FocusManager grants focus -> Cursor starts blinking at 500ms intervals
   │
3. TextInput channel requests native keyboard with obscureText: true
   │
4. User taps characters -> Characters stored as plain string in _passwordController
   │
5. RenderEditable displays bullets (••••)
   │
6. User taps Eye icon (IconButton):
   - setState(() => _isPasswordHidden = !_isPasswordHidden) triggers
   - TextField rebuilds with obscureText: false
   - Bullets immediately transform into visible characters on canvas
```

---

## ⚠ Common Beginner Mistakes

### Mistake 1: Setting `maxLines: null` without `keyboardType: TextInputType.multiline`

**Wrong Code:**
```dart
// ❌ CRASH OR AWKWARD BEHAVIOR:
TextField(
  maxLines: null, // Wants to expand infinitely
  // But keyboardType defaults to text (single-line action)!
)
```

**Why it is wrong:**
If `maxLines: null` is set, pressing the keyboard Enter/Return key should insert a new line. But if `keyboardType` is not multiline, the keyboard tries to submit instead of creating new lines.

**Correct Code:**
```dart
// ✅ CORRECT: Pair expanding maxLines with multiline keyboardType
TextField(
  maxLines: null,
  keyboardType: TextInputType.multiline,
)
```

---

### Mistake 2: Creating a new `TextEditingController` inside the `build()` method

**Wrong Code:**
```dart
// ❌ DISASTER: Controller is recreated on EVERY frame!
@override
Widget build(BuildContext context) {
  final controller = TextEditingController(); // RESETS ON EVERY KEYSTROKE!
  return TextField(controller: controller);
}
```

**Why it is wrong:**
Every time the user types a letter, the widget rebuilds. Rebuilding instantiates a brand new controller with empty text, causing the cursor to jump and text to erase itself!

**Correct Code:**
```dart
// ✅ CORRECT: Instantiate in initState() or as a class field, dispose in dispose()
class _MyState extends State<MyWidget> {
  late final TextEditingController _controller;

  @override
  void initState() {
    super.initState();
    _controller = TextEditingController();
  }
}
```

---

## 🚀 Professional Usage

`TextField` is utilized across every industry sector:

* **WhatsApp / Messaging:** The chat composer text field with dynamic multiline expansion (`minLines: 1, maxLines: 5`) that grows as you type longer messages.
* **Amazon / E-commerce:** The top search bar with `textInputAction: TextInputAction.search` triggering product queries.
* **Google Pay / Banking:** Phone number transfer inputs with `keyboardType: TextInputType.phone` and `maxLength: 10`.
* **Instagram / Social:** Caption entry boxes with `textCapitalization: TextCapitalization.sentences`.

---

## 💡 Best Practices

1. **Always Provide `onTapOutside` to Dismiss Keyboard:** Use `onTapOutside: (event) => FocusScope.of(context).unfocus()` so tapping anywhere outside the input box smoothly dismisses the keyboard.
2. **Always Use `textInputAction`:** Use `TextInputAction.next` on consecutive fields, and `TextInputAction.done` on the final field.
3. **Use `alignLabelWithHint: true` on Multiline Fields:** Keeps the floating label pinned to the top-left rather than floating in the vertical center of a tall box.
4. **Never Create Controllers in `build()`:** Always bind them to the `State` lifecycle.

---

## 🧪 Try It Yourself

1. Copy the runnable example above into your Flutter project.
2. In the Email TextField, add:
   * `onTapOutside: (event) => FocusScope.of(context).unfocus(),`
3. Tap into the email box so the keyboard appears.
4. Tap anywhere outside the box on the screen background. Notice how the keyboard immediately slides away smoothly!

---

## 🎯 Challenge

Build a **Credit Card Number Formatter Field**:
* A `TextField` with `keyboardType: TextInputType.number`.
* Set `maxLength: 19`.
* Set `prefixIcon: Icon(Icons.credit_card)`.
* Format the input as the user types so that a space is inserted after every 4 digits (`4920 1842 0912 8842`).
* When 16 digits are entered, change the border color to green!

---

## ❓ Practice Questions

### 1. Multiple Choice Question (MCQ)
Which property on `TextField` is used to mask characters with dots for passwords?
* A) `isPassword: true`
* B) `maskText: true`
* C) `obscureText: true`
* D) `hideText: true`

*(Correct Answer: C)*

---

### 2. True or False
**Statement:** A `TextEditingController` should be instantiated inside the `build()` method of a widget.
*(Answer: False. Instantiating it in `build()` resets the controller on every frame, wiping out user input and jumping the cursor).*

---

### 3. Output Prediction
What will happen if you set `readOnly: true` on a `TextField` and tap it?
* A) It crashes
* B) The text is selected, but the keyboard does NOT open and text cannot be edited
* C) The text turns red
* D) It deletes the text

*(Correct Answer: B)*

---

### 4. Theory
Explain the difference between `TextInputAction.next` and `TextInputAction.done`.
*(Answer: `TextInputAction.next` displays a forward arrow on the software keyboard indicating that pressing it will advance focus to the next field. `TextInputAction.done` displays a checkmark or "Done" button that submits input and dismisses the keyboard).*

---

### 5. Multiple Choice Question (MCQ)
How can you allow a `TextField` to expand vertically indefinitely as the user types more lines?
* A) `expand: true`
* B) `maxLines: null` with `keyboardType: TextInputType.multiline`
* C) `multiLine: true`
* D) `infiniteLines: true`

*(Correct Answer: B)*

---

### 6. Coding Exercise
Write the code for a `TextField` that automatically capitalizes the first letter of every word typed.
*(Answer:*
```dart
TextField(
  textCapitalization: TextCapitalization.words,
  decoration: const InputDecoration(hintText: 'Enter Full Name'),
)
```
*)*

---

### 7. Theory
What does `alignLabelWithHint: true` do on a multiline `TextField`?
*(Answer: It forces the floating label to align at the top-left of the input box alongside the hint text, rather than centering itself vertically in the middle of a tall multiline box).*

---

### 8. Output Prediction
What will happen if a user types past the limit on a `TextField` with `maxLength: 10`?
* A) The app crashes
* B) The user is strictly prevented from typing any further characters beyond 10
* C) The text turns red but typing continues
* D) The first character is deleted

*(Correct Answer: B)*

---

### 9. Multiple Choice Question (MCQ)
Which callback is triggered on every single keystroke as text is typed or deleted?
* A) `onSubmitted`
* B) `onEditingComplete`
* C) `onChanged`
* D) `onTap`

*(Correct Answer: C)*

---

### 10. True or False
**Statement:** You can change the cursor color, width, and radius on a `TextField`.
*(Answer: True. Via `cursorColor`, `cursorWidth`, and `cursorRadius`).*

---

## 📝 Summary

* **Core Purpose:** `TextField` is Flutter’s primary text input widget for freeform text entry.
* **Controller Rule:** Always pair with a `TextEditingController` managed in `initState()` and `dispose()`.
* **Security:** Use `obscureText: true` for passwords and sensitive credentials.
* **Keyboard Matching:** Always set `keyboardType` (`emailAddress`, `phone`, `number`, `multiline`) to match expected data.
* **Usability:** Use `onTapOutside` to dismiss keyboards cleanly and `textInputAction` to guide tab order.
