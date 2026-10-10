# TextFormField

## 📖 Introduction

In the previous chapters, you mastered `TextField`. You know how to style it, attach a controller, and listen to keystrokes.

Now imagine building a real registration screen with 6 fields: First Name, Last Name, Email, Phone, Password, and Confirm Password.

If you use basic `TextField` widgets:
* You have to write manual `if-else` checks for all 6 fields on button press.
* You have to manually track 6 individual error string variables in state (`_emailError`, `_phoneError`, etc.).
* You have to manually call `setState()` to update the UI when errors occur.
* You have to manually clear all 6 fields when the form resets.

That is an immense amount of error-prone, repetitive code!

To eliminate this burden, Flutter introduces the enterprise-grade **`TextFormField`**.

A `TextFormField` is a `TextField` wrapped inside a `FormField`. It connects directly to an enclosing `Form` widget, bringing automatic validation, centralized error message rendering, automated form state saving, and one-tap resetting.

---

## 🤔 Why Do We Need It?

Without `TextFormField`, validating multiple inputs is tedious and disorganized:

1. **Integrated `validator` Callback:** You simply provide a `validator: (value) { ... }` function. If the function returns a string (e.g. `'Invalid Email'`), Flutter automatically changes the border to red and displays the error message. If it returns `null`, the field is marked valid.
2. **Centralized One-Line Validation:** Instead of checking 6 fields manually, you call `formKey.currentState!.validate()`. That single line triggers the validators on all 6 `TextFormField` widgets simultaneously!
3. **Form State Lifecycle:** Supports `onSaved` callbacks, so valid data can be automatically committed to your data model in one synchronized call.

---

## 🌍 Real-Life Analogy

Think of **a standardized passport application form checked by an automated scanner**:

```text
 ┌────────────────────────────────────────────────────────┐
 │           Passport Application (The Form)              │
 ├────────────────────────────────────────────────────────┤
 │ Field 1: [ John Doe               ]  validator -> OK   │
 │ Field 2: [ invalid-email          ]  validator -> ERR  │
 │ Field 3: [                        ]  validator -> ERR  │
 ├────────────────────────────────────────────────────────┤
 │ [ SUBMIT ]: Scanner checks all fields in 1 millisecond │
 └────────────────────────────────────────────────────────┘
```

* With raw `TextField`, you are handing a loose sheet of scratch paper to a clerk who must manually read and verify every line with a magnifying glass.
* With `TextFormField`, each blank has an automated bar-code scanner attached.
* When you press "Submit", the scanner runs down the whole page in one millisecond. Any field with an issue automatically glows red and prints its correction instructions.

---

## 🧠 Internal Working

How does `TextFormField` operate under the hood?

```text
Form (Holds FormState with GlobalKey)
 │
 ├── Registers all child FormFieldState objects
 │
 └── User taps "Submit" -> formKey.currentState!.validate()
       │
       ▼
 1. Form iterates through all registered FormFieldState instances:
    ├── TextFormField 1 (Name):
    │   Runs validator("John") -> returns null (VALID)
    │
    ├── TextFormField 2 (Email):
    │   Runs validator("invalid") -> returns "Enter valid email" (INVALID)
    │   Updates internal state: hasError = true, errorText = "Enter valid email"
    │
 2. FormState.validate() returns false
 │
 3. Each invalid TextFormField automatically rebuilds its InputDecoration:
    - border changes to errorBorder (Red)
    - errorText is rendered beneath the box
```

### The Class Hierarchy:
```text
StatefulWidget
   └── FormField<String>
         └── TextFormField (Wraps an internal TextField)
```
`TextFormField` gives you 100% of the styling and typing capabilities of `TextField`, plus the full power of `FormField`.

---

## ⚙ Syntax

The constructor of `TextFormField`:

```dart
TextFormField({
  Key? key,
  TextEditingController? controller,
  String? initialValue,
  FocusNode? focusNode,
  InputDecoration? decoration = const InputDecoration(),
  TextInputType? keyboardType,
  TextCapitalization textCapitalization = TextCapitalization.none,
  TextInputAction? textInputAction,
  TextStyle? style,
  bool autofocus = false,
  bool readOnly = false,
  bool obscureText = false,
  int? maxLines = 1,
  int? minLines,
  int? maxLength,
  ValueChanged<String>? onChanged,
  VoidCallback? onEditingComplete,
  ValueChanged<String>? onFieldSubmitted,
  FormFieldSetter<String>? onSaved,
  FormFieldValidator<String>? validator,
  List<TextInputFormatter>? inputFormatters,
  bool? enabled,
  AutovalidateMode? autovalidateMode,
})
```

### Key Differences from `TextField`:
* **`validator`**: `String? Function(String? value)` returning an error message, or `null` if valid.
* **`onSaved`**: `void Function(String? value)` called when `formKey.currentState!.save()` runs.
* **`initialValue`**: Initial text (cannot be used if `controller` is also provided).
* **`autovalidateMode`**: Controls when validation triggers (`onUserInteraction`, `always`, `disabled`).

---

## 📋 Important Properties

| Property | Type | Default Value | Purpose |
|---|---|---|---|
| `validator` | `FormFieldValidator<String>?` | `null` | Validation logic. Returns error string or `null`. |
| `onSaved` | `FormFieldSetter<String>?` | `null` | Callback to store value into a data model. |
| `autovalidateMode`| `AutovalidateMode?` | `disabled` | When validation runs (`disabled`, `always`, `onUserInteraction`). |
| `controller` | `TextEditingController?` | `null` | Controls the text buffer. |
| `initialValue` | `String?` | `null` | Pre-filled text (do not use with `controller`). |

### Understanding `AutovalidateMode`:
* **`AutovalidateMode.disabled`:** Only validates when you explicitly call `.validate()`.
* **`AutovalidateMode.onUserInteraction`:** Validates automatically as soon as the user starts typing, providing real-time feedback.
* **`AutovalidateMode.always`:** Validates immediately upon screen launch (shows red errors before the user even types—avoid this!).

---

## 💻 Complete Runnable Example

Below is a complete, runnable Flutter app showcasing `TextFormField` with multi-field validation, custom regex matching, real-time error clearance, and form submission.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const TextFormFieldDemoApp());
}

class TextFormFieldDemoApp extends StatelessWidget {
  const TextFormFieldDemoApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: LoginValidationScreen(),
    );
  }
}

class LoginValidationScreen extends StatefulWidget {
  const LoginValidationScreen({super.key});

  @override
  State<LoginValidationScreen> createState() => _LoginValidationScreenState();
}

class _LoginValidationScreenState extends State<LoginValidationScreen> {
  // 1. GlobalKey to coordinate the Form
  final GlobalKey<FormState> _formKey = GlobalKey<FormState>();

  final TextEditingController _emailController = TextEditingController();
  final TextEditingController _passwordController = TextEditingController();

  bool _isPasswordHidden = true;

  @override
  void dispose() {
    _emailController.dispose();
    _passwordController.dispose();
    super.dispose();
  }

  void _submitForm() {
    // 2. Validate all fields in one single call!
    if (_formKey.currentState!.validate()) {
      // All fields passed validation!
      final email = _emailController.text.trim();
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          backgroundColor: Colors.green.shade800,
          content: Text('Validation Successful! Logging in as $email...'),
        ),
      );
    } else {
      // At least one field failed validation
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(
          backgroundColor: Colors.redAccent,
          content: Text('Please fix the errors in red before continuing.'),
        ),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFFF8FAFC),
      appBar: AppBar(
        title: const Text(
          'TextFormField Validation',
          style: TextStyle(fontWeight: FontWeight.bold),
        ),
        backgroundColor: Colors.indigo.shade800,
        foregroundColor: Colors.white,
        elevation: 2,
      ),
      body: SafeArea(
        child: SingleChildScrollView(
          padding: const EdgeInsets.all(24.0),
          // Form widget wrapping all TextFormFields
          child: Form(
            key: _formKey,
            autovalidateMode: AutovalidateMode.onUserInteraction,
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                const Text(
                  'Welcome Back',
                  style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
                ),
                const SizedBox(height: 6),
                Text(
                  'Enter your credentials to access your account.',
                  style: TextStyle(color: Colors.grey.shade600, fontSize: 13),
                ),
                const SizedBox(height: 28),

                // Field 1: Email with Regex Validation
                TextFormField(
                  controller: _emailController,
                  keyboardType: TextInputType.emailAddress,
                  textInputAction: TextInputAction.next,
                  decoration: InputDecoration(
                    labelText: 'Email Address',
                    hintText: 'name@domain.com',
                    prefixIcon: const Icon(Icons.email_outlined),
                    filled: true,
                    fillColor: Colors.white,
                    border: OutlineInputBorder(borderRadius: BorderRadius.circular(12)),
                  ),
                  validator: (value) {
                    if (value == null || value.trim().isEmpty) {
                      return 'Email is required';
                    }
                    final emailRegex = RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$');
                    if (!emailRegex.hasMatch(value.trim())) {
                      return 'Please enter a valid email address';
                    }
                    return null; // Valid!
                  },
                ),

                const SizedBox(height: 20),

                // Field 2: Password with Length Validation
                TextFormField(
                  controller: _passwordController,
                  obscureText: _isPasswordHidden,
                  textInputAction: TextInputAction.done,
                  decoration: InputDecoration(
                    labelText: 'Password',
                    hintText: 'At least 8 characters',
                    prefixIcon: const Icon(Icons.lock_outline),
                    suffixIcon: IconButton(
                      icon: Icon(
                        _isPasswordHidden ? Icons.visibility_off : Icons.visibility,
                        color: Colors.grey,
                      ),
                      onPressed: () {
                        setState(() => _isPasswordHidden = !_isPasswordHidden);
                      },
                    ),
                    filled: true,
                    fillColor: Colors.white,
                    border: OutlineInputBorder(borderRadius: BorderRadius.circular(12)),
                  ),
                  validator: (value) {
                    if (value == null || value.isEmpty) {
                      return 'Password is required';
                    }
                    if (value.length < 8) {
                      return 'Password must be at least 8 characters';
                    }
                    return null; // Valid!
                  },
                ),

                const SizedBox(height: 28),

                // Submit Button
                SizedBox(
                  width: double.infinity,
                  height: 52,
                  child: ElevatedButton(
                    style: ElevatedButton.styleFrom(
                      backgroundColor: Colors.indigo.shade800,
                      foregroundColor: Colors.white,
                      shape: RoundedRectangleBorder(
                        borderRadius: BorderRadius.circular(12),
                      ),
                    ),
                    onPressed: _submitForm,
                    child: const Text(
                      'Log In',
                      style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
                    ),
                  ),
                ),

                const SizedBox(height: 12),

                // Reset Button demonstrating form reset
                SizedBox(
                  width: double.infinity,
                  child: TextButton(
                    onPressed: () {
                      _formKey.currentState!.reset();
                      _emailController.clear();
                      _passwordController.clear();
                    },
                    child: const Text('Reset Form'),
                  ),
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }
}
```

---

## 🔍 Line-by-Line Example Explanation

### 1. `final GlobalKey<FormState> _formKey = GlobalKey<FormState>();`
* **What it does:** Creates a unique identifier for the `FormState`.
* **Why it exists:** Provides access to the parent form's methods (`validate()`, `save()`, `reset()`).
* **What Flutter does internally:** Binds the `FormState` in the element tree to this reference key.
* **What happens if removed:** You cannot coordinate validation across the child form fields.

### 2. `_formKey.currentState!.validate()`
* **What it does:** Executes the `validator` callback on every `TextFormField` inside the `Form`.
* **Why it exists:** Validates all fields simultaneously in a single clean expression.
* **What Flutter does internally:** Iterates through all registered fields, checks if any returned an error string, updates their error state, and returns `true` only if ALL fields returned `null`.
* **What happens if removed:** Form submission occurs blindly without checking for invalid or empty inputs.

### 3. `validator: (value) { ... return null; }`
* **What it does:** Declares the specific criteria for validity (e.g. non-empty, regex matching, minimum length).
* **Why it exists:** Encapsulates business logic directly inside the field definition.
* **What Flutter does internally:** If it returns a `String`, Flutter passes it to `InputDecoration.errorText` and turns the border red.
* **What happens if removed:** The field becomes unvalidated, accepting any input.

### 4. `autovalidateMode: AutovalidateMode.onUserInteraction`
* **What it does:** Validates the field in real time as soon as the user starts typing.
* **Why it exists:** Removes the error message immediately the moment the user corrects their typo, without waiting for another button click.
* **What Flutter does internally:** Triggers validation on every `onChanged` event after initial touch.
* **What happens if removed:** Errors only update when the submit button is tapped.

---

## 📱 ASCII Phone Screen Preview

```text
+------------------------------------------+
| TextFormField Validation                 |
+------------------------------------------+
| Welcome Back                             |
| Enter your credentials...                |
|                                          |
| Email Address                            |
| +--------------------------------------+ |
| | (✉) user@domain                      | |
| +--------------------------------------+ |
| ❌ Please enter a valid email address    | <--- Red error message
|                                          |
| Password                                 |
| +--------------------------------------+ |
| | (🔒) ••••••                     (👁) | |
| +--------------------------------------+ |
| ❌ Password must be at least 8 chars     | <--- Red error message
|                                          |
| [               Log In                 ] |
|                                          |
|              Reset Form                  |
+------------------------------------------+
```

---

## 🌳 Widget Tree

```text
TextFormFieldDemoApp
└── MaterialApp
    └── LoginValidationScreen (StatefulWidget)
        └── Scaffold
            ├── AppBar
            │   └── Text ("TextFormField Validation")
            └── SafeArea
                └── SingleChildScrollView
                    └── Form (key: _formKey)
                        └── Column
                            ├── Text ("Welcome Back")
                            ├── TextFormField (Email: validator: regex)
                            ├── TextFormField (Password: validator: length)
                            ├── ElevatedButton ("Log In")
                            └── TextButton ("Reset Form")
```

---

## ⚙ Flutter Execution Flow

```text
1. User taps "Log In" button
   │
2. _submitForm() calls _formKey.currentState!.validate()
   │
3. FormState loops through children:
   - Email validator runs: "user@domain" lacks top-level domain -> Returns error
   - Password validator runs: 6 chars < 8 -> Returns error
   │
4. validate() returns false
   │
5. Both TextFormFields rebuild with error state:
   - Error borders turn red
   - Error texts slide into view beneath each box
   │
6. ScaffoldMessenger displays warning SnackBar: "Please fix errors in red"
```

---

## ⚠ Common Beginner Mistakes

### Mistake 1: Supplying both `controller` and `initialValue`

**Wrong Code:**
```dart
// ❌ RUNTIME ASSERTION CRASH:
TextFormField(
  controller: myController,
  initialValue: 'Hello', // CRASH! Cannot provide both!
)
```

**Why it is wrong:**
Flutter throws an assertion error: `initialValue == null || controller == null`. If you are using a controller, initialize the text via `TextEditingController(text: 'Hello')`.

**Correct Code:**
```dart
// ✅ CORRECT: Set initial value inside the controller
final myController = TextEditingController(text: 'Hello');

TextFormField(
  controller: myController,
)
```

---

### Mistake 2: Forgetting to return `null` when input is valid

**Wrong Code:**
```dart
// ❌ LOGIC BUG: Field is never considered valid!
validator: (value) {
  if (value!.isEmpty) {
    return 'Cannot be empty';
  }
  // Missing return null; (defaults to returning null in Dart, but if you return empty string '' it stays red!)
}
```

**Why it is wrong:**
If you return an empty string `return ''`, Flutter treats it as an error message and paints the border red. **You must return `null` for valid inputs.**

---

## 🚀 Professional Usage

`TextFormField` is the foundation of mission-critical user onboarding:

* **Fintech / KYC Forms:** Validating national identity numbers, tax IDs, and bank account numbers against strict formatting rules.
* **Flight Booking Apps:** Passport expiration date validation, passenger name matching, and seat selection confirmation.
* **E-Commerce Checkout:** Shipping address validation ensuring postal codes match country formats.
* **Authentication Screens:** Login, registration, password reset, and 2FA input forms.

---

## 💡 Best Practices

1. **Always Return `null` for Valid Inputs:** Only return a `String` when an error genuinely exists.
2. **Use `AutovalidateMode.onUserInteraction`:** It provides the best UX by validating as users type without showing aggressive errors on an untouched screen.
3. **Use RegExp for Email and Phone Validation:** Don't just check `value.contains('@')`; use a robust regular expression.
4. **Trim Strings in Validators:** Always use `value?.trim()` so users don't fail validation due to accidental trailing spaces.

---

## 🧪 Try It Yourself

1. Copy the runnable example into your Flutter project.
2. In the Email field, type `"test"`. Observe how the red error appears immediately.
3. Complete the email to `"test@gmail.com"`. Notice how the red error vanishes the instant the email format becomes valid!

---

## 🎯 Challenge

Build a **Sign-Up Form with Confirm Password Matching**:
* A form with three `TextFormField` widgets:
  1. **Username:** Must be at least 4 characters.
  2. **Password:** Must be at least 8 characters and contain at least one number (`RegExp(r'[0-9]')`).
  3. **Confirm Password:** Validator must check that `value == _passwordController.text`, displaying `"Passwords do not match"` if different.
* Ensure all three fields validate together on submit.

---

## ❓ Practice Questions

### 1. Multiple Choice Question (MCQ)
What should a `validator` callback return if the user input is completely valid?
* A) `true`
* B) `""` (Empty string)
* C) `null`
* D) `"Valid"`

*(Correct Answer: C)*

---

### 2. True or False
**Statement:** You can provide both a `controller` and an `initialValue` to the same `TextFormField`.
*(Answer: False. Providing both throws a runtime assertion error).*

---

### 3. Output Prediction
What will happen if your validator returns `return '';` (an empty string)?
* A) The field is marked valid
* B) The field is marked INVALID; the border turns red, but no error text is displayed
* C) It crashes
* D) The text is deleted

*(Correct Answer: B)*

---

### 4. Theory
Explain the difference between `TextField` and `TextFormField`.
*(Answer: `TextField` is a standalone input box. `TextFormField` is a `TextField` wrapped in a `FormField`, allowing integration with a parent `Form` widget for centralized validation (`validate()`), saving (`save()`), and resetting (`reset()`)).*

---

### 5. Multiple Choice Question (MCQ)
Which method on `FormState` triggers the `validator` callback on all child `TextFormField` widgets?
* A) `check()`
* B) `validate()`
* C) `verify()`
* D) `submit()`

*(Correct Answer: B)*

---

### 6. Coding Exercise
Write a `TextFormField` validator that ensures a field is not empty and is at least 6 characters long.
*(Answer:*
```dart
validator: (value) {
  if (value == null || value.trim().isEmpty) {
    return 'Field cannot be empty';
  }
  if (value.trim().length < 6) {
    return 'Must be at least 6 characters';
  }
  return null;
}
```
*)*

---

### 7. Theory
What is the purpose of `AutovalidateMode.onUserInteraction`?
*(Answer: It suppresses validation errors initially, but once the user interacts with the field, it validates continuously on every keystroke, providing immediate feedback as errors are corrected).*

---

### 8. Output Prediction
What does `_formKey.currentState!.reset()` do to all child `TextFormField` widgets?
* A) Closes the app
* B) Resets all fields back to their `initialValue` and clears all red validation error messages
* C) Deletes the widgets from the tree
* D) Rebuilds the Scaffold

*(Correct Answer: B)*

---

### 9. Multiple Choice Question (MCQ)
Which callback on `TextFormField` is called when `_formKey.currentState!.save()` is executed?
* A) `onSubmitted`
* B) `onSaved`
* C) `onChanged`
* D) `onComplete`

*(Correct Answer: B)*

---

### 10. True or False
**Statement:** `TextFormField` supports all styling properties of `InputDecoration` including prefix icons, floating labels, and filled backgrounds.
*(Answer: True).*

---

## 📝 Summary

* **Core Purpose:** `TextFormField` integrates text entry with Flutter’s `Form` validation ecosystem.
* **The Validator Rule:** Return an error `String` if invalid; return `null` if valid.
* **One-Call Validation:** `_formKey.currentState!.validate()` evaluates all fields on the screen simultaneously.
* **Autovalidate UX:** Use `AutovalidateMode.onUserInteraction` for responsive real-time error clearance.
* **Clean Code:** Replaces dozens of manual error strings and `setState` calls with declarative validation logic.
