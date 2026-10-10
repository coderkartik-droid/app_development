# Form Validation in Flutter

## 📖 Introduction
User input is inherently unpredictable. When prompted to enter an email, users may omit the `@` symbol, mistype domain extensions, leave required entries blank, enter letters into credit card fields, or supply passwords too weak to protect their accounts. Processing corrupted or malicious data on your backend leads to database crashes, security vulnerabilities, and degraded user experiences.

**Form Validation** is the discipline of testing user input against defined business rules before that data leaves the device or triggers business logic. In Flutter, validation is not an afterthought handled by manual string checks and alert popups. It is deeply integrated into the [FormField](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/05_TextFormField.md) and [Form](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/06_Form_Widget.md) architecture via pure validator functions.

---

## 🤔 Why Do We Need It?
Without a structured validation framework:
1. **Network Waste**: Submitting malformed data forces expensive HTTP roundtrips only for the backend API to return `400 Bad Request`.
2. **Poor Usability**: Displaying generic dialog boxes like *"Invalid submission"* forces users to guess which specific field among ten caused the failure.
3. **Spaghetti Code**: Developers clutter their UI files with dozens of boolean flags, manual error variables, and nested conditional trees to toggle error text visibility.

With Flutter's Form Validation:
- Each field owns its discrete `validator` callback.
- Rules return a descriptive `String` when invalid, or `null` when valid.
- The framework handles coloring borders red, rendering error text underneath the field, and adjusting layout heights automatically.
- Validation cascades across the entire form with a single call to `_formKey.currentState!.validate()`.

---

## 🌍 Real-Life Analogy
Imagine submitting an application at an **Embassy Visa Counter**:

```
+-------------------------------------------------------------+
|                      EMBASSY DESK (Form)                    |
|                                                             |
|  [Officer Inspector]                                        |
|  Rule 1: Is your passport valid for > 6 months?             |
|          -> No? "Passport expires too soon!" [REJECTED]     |
|                                                             |
|  Rule 2: Is your photo 2x2 inches?                          |
|          -> Yes (null error) [PASSED]                       |
|                                                             |
|  Rule 3: Is your signature present?                         |
|          -> No? "Signature missing!" [REJECTED]             |
|                                                             |
|  [VERDICT] If ANY rule failed -> Hand back documents with   |
|  red sticky notes on the exact faulty pages.                |
|  If ALL passed -> Stamp APPROVED and file into archive.     |
+-------------------------------------------------------------+
```

The embassy officer will not accept your application file if even a single document fails the checklist. You receive immediate, specific feedback right next to the faulty document.

---

## 🧠 Internal Working
Flutter form validation operates through a deterministic contract:

```
                  User taps "Submit" Button
                             │
                             ▼
                _formKey.currentState!.validate()
                             │
                             ▼
           Loop through each registered FormFieldState
                             │
                ┌────────────┴────────────┐
                ▼                         ▼
         Field 1 Validator         Field 2 Validator
                │                         │
         Returns String?           Returns String?
                │                         │
      ┌─────────┴─────────┐         ┌─────┴─────┐
   != null              == null   != null     == null
      │                   │         │           │
      ▼                   ▼         ▼           ▼
  Error Text         Valid Field  Error Text  Valid Field
  Displayed          (No error)   Displayed   (No error)
      │                   │         │           │
      └─────────┬─────────┘         └─────┬─────┘
                │                         │
                └───────────┬─────────────┘
                            ▼
        Did ANY field return a non-null String?
                     ├── YES ──► Form returns false (Abort submit)
                     └── NO  ──► Form returns true (Proceed to save)
```

The signature of a validator callback is:
```dart
String? Function(T? value)
```
- **If the input is valid**: The function must return `null`.
- **If the input is invalid**: The function must return a `String` containing the exact error message to be displayed.

---

## ⚙ Syntax

### Basic Null & Empty Check
```dart
TextFormField(
  validator: (String? value) {
    if (value == null || value.trim().isEmpty) {
      return 'This field cannot be left blank';
    }
    return null; // Valid!
  },
)
```

### Regular Expression (Regex) Matching
```dart
TextFormField(
  validator: (String? value) {
    if (value == null || value.isEmpty) {
      return 'Email is required';
    }
    final emailRegex = RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$');
    if (!emailRegex.hasMatch(value)) {
      return 'Enter a valid email address (e.g. name@domain.com)';
    }
    return null;
  },
)
```

---

## 📋 Important Properties & Validation Trigger Modes

| Mode | Behavior | Best Used For |
| :--- | :--- | :--- |
| `AutovalidateMode.disabled` | Validates only when `_formKey.currentState!.validate()` is explicitly invoked via code (e.g. upon button press). | Standard login forms where you don't want to distract users while they type. |
| `AutovalidateMode.onUserInteraction` | Suppressed initially. Once the user enters text or leaves the field, validation runs on every keystroke. | Password strength checks, confirm password matches, email formatting. |
| `AutovalidateMode.always` | Validates immediately upon rendering, even when inputs are completely untouched. | Rarely recommended; displays jarring red errors on empty screens. |

---

## 💻 Complete Runnable Example
The following production-ready application demonstrates complete validation logic for:
1. **Full Name** (Length & character requirements)
2. **Email Address** (Regex pattern verification)
3. **Password** (Complexity: uppercase, digits, min length)
4. **Confirm Password** (Cross-field parity check)
5. **Phone Number** (Exact 10-digit numeric constraint)

```dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() {
  runApp(const ValidationMasterclassApp());
}

class ValidationMasterclassApp extends StatelessWidget {
  const ValidationMasterclassApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Form Validation Masterclass',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const SecureRegistrationScreen(),
    );
  }
}

class SecureRegistrationScreen extends StatefulWidget {
  const SecureRegistrationScreen({super.key});

  @override
  State<SecureRegistrationScreen> createState() => _SecureRegistrationScreenState();
}

class _SecureRegistrationScreenState extends State<SecureRegistrationScreen> {
  final GlobalKey<FormState> _formKey = GlobalKey<FormState>();

  // Controllers to enable cross-field validation
  final TextEditingController _passwordController = TextEditingController();
  final TextEditingController _confirmPasswordController = TextEditingController();

  // Model values
  String _name = '';
  String _email = '';
  String _phone = '';

  @override
  void dispose() {
    _passwordController.dispose();
    _confirmPasswordController.dispose();
    super.dispose();
  }

  // Pure validation functions
  String? _validateName(String? value) {
    if (value == null || value.trim().isEmpty) {
      return 'Please enter your legal name';
    }
    if (value.trim().length < 3) {
      return 'Name must contain at least 3 characters';
    }
    final nameRegex = RegExp(r"^[a-zA-Z\s\-']+$");
    if (!nameRegex.hasMatch(value.trim())) {
      return 'Name contains invalid characters';
    }
    return null;
  }

  String? _validateEmail(String? value) {
    if (value == null || value.trim().isEmpty) {
      return 'Email address is required';
    }
    final emailRegex = RegExp(r'^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+$');
    if (!emailRegex.hasMatch(value.trim())) {
      return 'Enter a valid email address (e.g., user@domain.com)';
    }
    return null;
  }

  String? _validatePhone(String? value) {
    if (value == null || value.trim().isEmpty) {
      return 'Phone number is required';
    }
    if (value.trim().length != 10) {
      return 'Phone number must be exactly 10 digits';
    }
    return null;
  }

  String? _validatePassword(String? value) {
    if (value == null || value.isEmpty) {
      return 'Password cannot be blank';
    }
    if (value.length < 8) {
      return 'Password must be at least 8 characters long';
    }
    if (!value.contains(RegExp(r'[A-Z]'))) {
      return 'Must include at least one uppercase letter (A-Z)';
    }
    if (!value.contains(RegExp(r'[0-9]'))) {
      return 'Must include at least one numerical digit (0-9)';
    }
    if (!value.contains(RegExp(r'[!@#$%^&*(),.?":{}|<>]'))) {
      return 'Must include at least one special character';
    }
    return null;
  }

  String? _validateConfirmPassword(String? value) {
    if (value == null || value.isEmpty) {
      return 'Please confirm your password';
    }
    if (value != _passwordController.text) {
      return 'Passwords do not match';
    }
    return null;
  }

  void _submit() {
    if (_formKey.currentState?.validate() ?? false) {
      _formKey.currentState?.save();

      showDialog(
        context: context,
        builder: (ctx) => AlertDialog(
          icon: const Icon(Icons.verified_user, color: Colors.green, size: 48),
          title: const Text('Validation Passed!'),
          content: Text(
            'Account created successfully:\n\n'
            'Name: $_name\n'
            'Email: $_email\n'
            'Phone: +1 $_phone',
          ),
          actions: [
            FilledButton(
              onPressed: () => Navigator.pop(ctx),
              child: const Text('OK'),
            ),
          ],
        ),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Enterprise Validation'),
        centerTitle: true,
      ),
      body: SafeArea(
        child: SingleChildScrollView(
          padding: const EdgeInsets.all(24.0),
          child: Form(
            key: _formKey,
            autovalidateMode: AutovalidateMode.onUserInteraction,
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.stretch,
              children: [
                const Text(
                  'Create Secure Account',
                  style: TextStyle(fontSize: 22, fontWeight: FontWeight.bold),
                ),
                const SizedBox(height: 6),
                const Text(
                  'All fields are rigorously verified in real-time.',
                  style: TextStyle(color: Colors.grey),
                ),
                const SizedBox(height: 24),

                // Name Field
                TextFormField(
                  decoration: const InputDecoration(
                    labelText: 'Full Name *',
                    hintText: 'John Doe',
                    prefixIcon: Icon(Icons.person_outline),
                    border: OutlineInputBorder(),
                  ),
                  validator: _validateName,
                  onSaved: (val) => _name = val?.trim() ?? '',
                  textInputAction: TextInputAction.next,
                ),
                const SizedBox(height: 16),

                // Email Field
                TextFormField(
                  decoration: const InputDecoration(
                    labelText: 'Work Email *',
                    hintText: 'john@company.com',
                    prefixIcon: Icon(Icons.email_outlined),
                    border: OutlineInputBorder(),
                  ),
                  keyboardType: TextInputType.emailAddress,
                  validator: _validateEmail,
                  onSaved: (val) => _email = val?.trim() ?? '',
                  textInputAction: TextInputAction.next,
                ),
                const SizedBox(height: 16),

                // Phone Field
                TextFormField(
                  decoration: const InputDecoration(
                    labelText: 'Phone Number *',
                    hintText: '10 digits without spaces',
                    prefixIcon: Icon(Icons.phone_outlined),
                    prefixText: '+1 ',
                    border: OutlineInputBorder(),
                  ),
                  keyboardType: TextInputType.phone,
                  inputFormatters: [
                    FilteringTextInputFormatter.digitsOnly,
                    LengthLimitingTextInputFormatter(10),
                  ],
                  validator: _validatePhone,
                  onSaved: (val) => _phone = val?.trim() ?? '',
                  textInputAction: TextInputAction.next,
                ),
                const SizedBox(height: 16),

                // Password Field
                TextFormField(
                  controller: _passwordController,
                  obscureText: true,
                  decoration: const InputDecoration(
                    labelText: 'Password *',
                    hintText: 'Min 8 chars, 1 uppercase, 1 digit, 1 symbol',
                    prefixIcon: Icon(Icons.lock_outline),
                    border: OutlineInputBorder(),
                  ),
                  validator: _validatePassword,
                  textInputAction: TextInputAction.next,
                  onChanged: (_) {
                    // Revalidate confirm password if user edits the primary password
                    if (_confirmPasswordController.text.isNotEmpty) {
                      _formKey.currentState?.validate();
                    }
                  },
                ),
                const SizedBox(height: 16),

                // Confirm Password Field
                TextFormField(
                  controller: _confirmPasswordController,
                  obscureText: true,
                  decoration: const InputDecoration(
                    labelText: 'Confirm Password *',
                    hintText: 'Re-enter your password',
                    prefixIcon: Icon(Icons.lock_reset),
                    border: OutlineInputBorder(),
                  ),
                  validator: _validateConfirmPassword,
                  textInputAction: TextInputAction.done,
                ),
                const SizedBox(height: 28),

                // Submit Button
                FilledButton.icon(
                  onPressed: _submit,
                  icon: const Icon(Icons.shield_outlined),
                  label: const Text('Verify & Create Account', style: TextStyle(fontSize: 16)),
                  style: FilledButton.styleFrom(
                    padding: const EdgeInsets.symmetric(vertical: 16),
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
- `String? _validateName(String? value)`:
  - **What it does**: Declares a top-level or class validator method adhering to `FormFieldValidator<String>`.
  - **Why it exists**: Separating validation algorithms from the widget build method improves readability, unit testability, and code reusability.
- `if (value == null || value.trim().isEmpty)`:
  - **What it does**: Guards against `null` references and handles strings containing only blank spaces.
  - **Internal behavior**: If a user enters five spaces `"     "`, `.trim().isEmpty` evaluates to `true`, preventing empty inputs from passing validation.
- `final emailRegex = RegExp(...)`:
  - **What it does**: Defines a standard RFC-compliant regular expression pattern for email addresses.
  - **Why it exists**: Ensures domain names contain valid labels, top-level domains, and standard structure.
- `if (value != _passwordController.text)`:
  - **What it does**: Performs cross-field parity validation between the confirm password field and the primary password field.
  - **Why it exists**: Guarantees the user didn't make a typo when entering an obscured secret key.
- `onChanged: (_) { if (_confirmPasswordController.text.isNotEmpty) _formKey.currentState?.validate(); }`:
  - **What it does**: Triggers cross-field re-evaluation. If the user first types identical passwords, then goes back and changes the primary password, the confirm password field immediately flags the mismatch.

---

## 📱 ASCII Phone Screen Preview
```
+------------------------------------------+
|          Enterprise Validation           |
+------------------------------------------+
| Create Secure Account                    |
| All fields are rigorously verified...    |
|                                          |
|  +------------------------------------+  |
|  | [👤] Full Name *                   |  |
|  | John Doe                           |  |
|  +------------------------------------+  |
|                                          |
|  +------------------------------------+  |
|  | [✉] Work Email *                   |  |
|  | john@company.                      |  |
|  +------------------------------------+  |
|  * Enter a valid email address           |
|                                          |
|  +------------------------------------+  |
|  | [📞] Phone Number *                |  |
|  | +1 5551234                         |  |
|  +------------------------------------+  |
|  * Phone number must be exactly 10 digits|
|                                          |
|  +------------------------------------+  |
|  | [🔒] Password *                    |  |
|  | Secret123!                         |  |
|  +------------------------------------+  |
|                                          |
|  +------------------------------------+  |
|  | [🔐] Confirm Password *            |  |
|  | Secret1234                         |  |
|  +------------------------------------+  |
|  * Passwords do not match                |
|                                          |
|  +------------------------------------+  |
|  |   [🛡] Verify & Create Account     |  |
|  +------------------------------------+  |
+------------------------------------------+
```

---

## 🌳 Widget Tree
```
MaterialApp
 └── SecureRegistrationScreen (StatefulWidget)
      └── Scaffold
           ├── AppBar
           └── SafeArea
                └── SingleChildScrollView
                     └── Form [key: _formKey, Mode: onUserInteraction]
                          └── Column
                               ├── Text (Header & Subtitle)
                               ├── TextFormField (_validateName)
                               ├── TextFormField (_validateEmail)
                               ├── TextFormField (_validatePhone)
                               ├── TextFormField (_validatePassword)
                               ├── TextFormField (_validateConfirmPassword)
                               └── FilledButton ("Verify & Create Account")
```

---

## ⚙ Flutter Execution Flow
1. **User Types**: As each character is typed in the email field, `AutovalidateMode.onUserInteraction` triggers `field.validate()`.
2. **Validator Execution**: `_validateEmail(value)` is invoked.
3. **Branching**:
   - If invalid, returns `'Enter a valid email address'`.
   - The field's `InputDecoration` renders the red error border and error label.
   - Flutter's layout engine requests a relayout to accommodate the extra vertical error text height.
4. **Button Press**: Tapping "Verify & Create Account" executes `_formKey.currentState!.validate()`.
5. **Collective Assessment**:
   - `FormState` iterates through all 5 fields.
   - If all return `null`, `validate()` returns `true`.
   - `_formKey.currentState!.save()` executes all `onSaved` callbacks, updating model properties.
   - The confirmation `AlertDialog` is displayed.

---

## ⚠ Common Beginner Mistakes

### 1. Returning an Empty String Instead of Null on Success
```dart
// ❌ BAD: An empty string is STILL an error!
validator: (value) {
  if (value == null || value.isEmpty) {
    return 'Required';
  }
  return ''; // WRONG! Flutter treats ANY non-null String as an error message!
}

// ✅ GOOD: Return null when input is valid
validator: (value) {
  if (value == null || value.isEmpty) {
    return 'Required';
  }
  return null; // Signals SUCCESS to Flutter
}
```

### 2. Forgetting to Handle Null or Whitespace-Only Strings
```dart
// ❌ BAD: Crashes on null or passes on strings with only spaces "   "
validator: (value) {
  if (value!.isEmpty) { // Null check operator used on a null value crashes!
    return 'Required';
  }
  return null;
}

// ✅ GOOD: Safe null check and whitespace trimming
validator: (value) {
  if (value == null || value.trim().isEmpty) {
    return 'Required';
  }
  return null;
}
```

### 3. Executing Async HTTP Calls Inside Synchronous `validator`
```dart
// ❌ BAD: validator must be synchronous! Returning Future<String?> fails.
validator: (value) async { // Compile error! FormFieldValidator is synchronous.
  bool exists = await checkUserOnBackend(value);
  return exists ? 'User exists' : null;
}

// ✅ GOOD: Perform sync validation in validator; run async checks in button callback
void _onFormSubmit() async {
  if (_formKey.currentState!.validate()) {
    final exists = await checkUserOnBackend(_email);
    if (exists) {
      // Show snackbar or set custom field error
    }
  }
}
```

---

## 🚀 Professional Usage
- **Fintech & Banking (e.g., Stripe, PayPal, Wise)**: Luhn algorithm validation ensures credit card numbers are mathematically valid before network transmission; BIC/SWIFT code formats are strictly enforced.
- **Healthcare & Identity (e.g., Epic, NHS App)**: Social Security Numbers, National Insurance codes, and medical record numbers validate checksums instantly to eliminate data entry mistakes.
- **Logistics & Delivery (e.g., FedEx, Uber Eats)**: Validates postal codes against country-specific regex expressions before calculating shipping rates.

---

## 💡 Best Practices
1. **Extract Validators into Pure Static Helper Classes**: Create a `Validators` class (e.g. `Validators.email`, `Validators.password`) to reuse logic across login, signup, and profile update screens.
2. **Unit Test Your Validators**: Because validators are pure functions `(String?) -> String?`, they can be tested via unit tests without mocking Flutter widgets.
3. **Use Friendly, Action-Oriented Error Messages**: Instead of vague errors like *"Invalid"*, specify the exact issue: *"Password must contain at least one digit (0-9)"*.
4. **Combine with InputFormatters**: Use `FilteringTextInputFormatter.digitsOnly` on numeric fields so users cannot enter invalid characters in the first place.

---

## 🧪 Try It Yourself
1. Add an age field and validate that the user is at least 18 years old.
2. Implement a URL validator that requires entries to start with `https://`.
3. Write a custom unit test verifying your email validation regex with valid and invalid strings.

---

## 🎯 Challenge
Create a composable validator chain in Dart:
```dart
typedef FormValidator = String? Function(String? value);

FormValidator composeValidators(List<FormValidator> validators) {
  // Return a single validator function that checks each validator in sequence
  // and returns the first error encountered, or null if all pass.
}
```
Apply this utility to validate a username that must be: (1) not empty, (2) at least 4 characters, and (3) alphanumeric without spaces.

---

## ❓ Practice Questions

### Q1. What return type must a Flutter `FormFieldValidator` have?
- A) `bool`
- B) `String?`
- C) `Future<bool>`
- D) `void`
*Answer: B* — A validator must return `null` if the value is valid, or a `String` containing the error message if invalid.

### Q2. What does returning an empty string `""` from a validator function indicate?
- A) The input is valid
- B) The field is cleared
- C) An error exists, but the displayed error text is blank
- D) The form is submitted
*Answer: C* — Any non-null `String` is treated as an error by Flutter. To indicate success, you must return `null`.

### Q3. Why is `AutovalidateMode.always` generally discouraged on clean registration screens?
- A) It causes infinite loops
- B) It shows aggressive red error messages before the user has even touched any field
- C) It prevents text input
- D) It disables the virtual keyboard
*Answer: B* — It creates a frustrating user experience by highlighting pristine, untouched fields in red immediately upon screen launch.

### Q4. Which `AutovalidateMode` starts validating a field only after the user has begun interacting with it?
- A) `AutovalidateMode.disabled`
- B) `AutovalidateMode.onUserInteraction`
- C) `AutovalidateMode.always`
- D) `AutovalidateMode.onUnfocus`
*Answer: B* — It leaves fields clean initially and triggers validation only once user interaction occurs.

### Q5. Can a Flutter `validator` function be marked `async`?
- A) Yes, Flutter natively awaits all validators
- B) No, `FormFieldValidator` is strictly a synchronous signature `String? Function(T? value)`
- C) Yes, but only inside a `StatefulWidget`
- D) Yes, if using `FutureBuilder`
*Answer: B* — Validators are synchronous. Asynchronous validation (like verifying an email is taken via an API call) must be executed in the submit action callback.

### Q6. How do you implement cross-field validation between a "Password" and a "Confirm Password" field?
*Answer*: Attach a `TextEditingController` to the primary password field. In the confirm password field's validator, compare the incoming `value` against `_passwordController.text`. If they do not match, return an error string.

### Q7. What is the benefit of calling `.trim()` before validating string lengths?
*Answer*: It strips out invisible leading and trailing whitespace characters, preventing users from bypassing "required" or "minimum length" checks by simply typing spaces.

### Q8. Output Prediction: What will the validator return for an input of `"   "`?
```dart
validator: (value) {
  if (value == null || value.trim().isEmpty) {
    return 'Required field';
  }
  return null;
}
```
*Answer*: `'Required field'`. The `.trim()` removes spaces, resulting in an empty string which triggers the error branch.

### Q9. What happens to the layout height of a `TextFormField` when validation fails?
*Answer*: Unless reserve error space is configured in the decoration (`helperText: ' '`), Flutter automatically expands the vertical height of the field to render the error text beneath the input box.

### Q10. Write a pure validator function that verifies a string contains a valid credit card expiry date formatted as `MM/YY`.
*Answer*:
```dart
String? validateExpiryDate(String? value) {
  if (value == null || value.trim().isEmpty) return 'Expiry required';
  final regex = RegExp(r'^(0[1-9]|1[0-2])\/?([0-9]{2})$');
  if (!regex.hasMatch(value.trim())) return 'Invalid format (MM/YY)';
  return null;
}
```

---

## 📝 Summary
- Form validation in Flutter relies on pure functions returning `String?`: `null` signifies valid input, while a non-null string displays as an error.
- Use `AutovalidateMode.onUserInteraction` for fluid, real-time feedback without premature error warnings.
- Always check both `value == null` and `value.trim().isEmpty` to protect against missing inputs and blank space bypasses.
- Cross-field validation (such as matching passwords) is achieved by comparing values against a sibling field's `TextEditingController`.
- Keep validators synchronous, modular, and unit-testable outside your widget tree.
