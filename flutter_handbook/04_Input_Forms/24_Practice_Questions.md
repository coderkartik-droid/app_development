# Comprehensive Practice Questions: Input & Forms

## 📝 Module Assessment
This question bank tests your end-to-end mastery of user input, form orchestration, controllers, focus management, selection widgets, pickers, and search interfaces in Flutter.

---

## 📌 Section 1: Multiple Choice Questions (MCQs)

### Q1. What is the fundamental difference between `TextField` and `TextFormField`?
- A) `TextField` is deprecated in Flutter 3
- B) `TextFormField` is a wrapper around `TextField` that integrates with `Form`, providing `validator`, `onSaved`, and automatic error decoration
- C) `TextField` works on iOS, while `TextFormField` works only on Android
- D) `TextFormField` does not accept a `TextEditingController`
*Answer: B* — `TextFormField` extends `FormField<String>`, allowing it to register with an enclosing `Form` and participate in unified validation and saving.

### Q2. Where should a `TextEditingController` be instantiated in a `StatefulWidget`?
- A) Inside the `build()` method
- B) Inside `initState()` or as a final state member variable
- C) As a global top-level variable
- D) Inside the widget's constructor
*Answer: B* — Instantiating inside `build()` causes the controller to recreate on every frame, resetting user input and losing focus.

### Q3. What happens if a `validator` function returns an empty string `""` instead of `null`?
- A) The input is marked as valid
- B) Flutter treats any non-null `String` as an error, so it marks the field invalid with a blank error label
- C) The app crashes with an assertion failure
- D) The text field clears its content
*Answer: B* — A validator must return strictly `null` to indicate a valid state.

### Q4. Which `AutovalidateMode` runs validation on every keystroke ONLY after the user has begun interacting with the field?
- A) `AutovalidateMode.always`
- B) `AutovalidateMode.disabled`
- C) `AutovalidateMode.onUserInteraction`
- D) `AutovalidateMode.onFocusChange`
*Answer: C* — It keeps pristine fields clean on screen launch, triggering validation feedback once the user edits the input.

### Q5. What happens if you define `final _formKey = GlobalKey<FormState>()` inside the `build()` method?
- A) The app performs faster
- B) Every keystroke triggers `build()`, recreating the key, destroying the `FormState`, closing the virtual keyboard, and wiping field data
- C) Flutter throws a compile-time syntax error
- D) The form cannot be saved
*Answer: B* — Keys must be persistent across builds to maintain state identity.

### Q6. Which method dismisses the virtual keyboard across the entire active screen?
- A) `SystemNavigator.pop()`
- B) `FocusScope.of(context).unfocus()`
- C) `Navigator.pop(context)`
- D) `FocusManager.disable()`
*Answer: B* — Surrenders primary focus from whatever node currently holds it.

### Q7. Why should you use `TextInputType.numberWithOptions(decimal: true)` instead of `TextInputType.number` when asking for currency on iOS?
- A) iOS does not support currency
- B) Standard `TextInputType.number` renders an integer-only keypad on iOS with no decimal point key
- C) It makes the numbers bold
- D) It automatically adds currency symbols
*Answer: B* — iOS compact number pads omit the period button unless decimal options are requested.

### Q8. What does returning `oldValue` from a `TextInputFormatter`'s `formatEditUpdate()` method do?
- A) Clears the field
- B) Blocks/rejects the newly typed keystroke or pasted character
- C) Moves the cursor to index 0
- D) Crashes the application
*Answer: B* — Discards the proposed change and keeps the field's previous state.

### Q9. What visual indicator is displayed when a `Checkbox` has `tristate: true` and `value: null`?
- A) A question mark
- B) An empty white box
- C) A horizontal line or dash (`-`) representing an indeterminate state
- D) An exclamation point
*Answer: C* — Null represents an indeterminate partial selection.

### Q10. How is mutual exclusivity established between multiple `Radio` widgets?
- A) Wrapping them in a `RadioGroup` container
- B) Assigning them identical `groupValue` variables
- C) Assigning them identical `value` variables
- D) Flutter handles all radios globally on a screen
*Answer: B* — Radios sharing the same `groupValue` state variable coordinate mutual exclusivity.

### Q11. Which widget is semantically designed for an instantaneous live setting change that takes effect immediately?
- A) `Checkbox`
- B) `Radio`
- C) `Switch`
- D) `DropdownButton`
*Answer: C* — A `Switch` implies an immediate live state transition without waiting for a submission button.

### Q12. Under what condition is the floating `label` balloon displayed above a `Slider` thumb?
- A) Always when `label` is not null
- B) Only when `divisions` is set to a non-null integer and the user is dragging the thumb
- C) Only on desktop platforms
- D) Only when `min: 0`
*Answer: B* — Step divisions are mandatory for floating value labels to render.

### Q13. What happens if two `DropdownMenuItem` items in the same `DropdownButton` have identical `value` properties?
- A) Both items are selected
- B) Flutter crashes with an assertion failure stating items must have unique values
- C) The second item is ignored
- D) The dropdown converts to a multi-select list
*Answer: B* — Each option must possess a unique identifier.

### Q14. What does `showDatePicker()` return if the user dismisses the dialog by tapping outside the barrier?
- A) `DateTime.now()`
- B) `null`
- C) `DateTime(0)`
- D) Throws a `CancellationException`
*Answer: B* — Cancellation resolves the `Future` with `null`.

### Q15. Why does `CupertinoTimerPicker` crash if placed directly inside an unconstrained `Column`?
- A) It is an iOS widget
- B) It has unbounded intrinsic height and requires an explicit height container like `SizedBox(height: 200)`
- C) It requires a GlobalKey
- D) It only works inside modal dialogs
*Answer: B* — The cylindrical wheel projection requires bounded vertical constraints.

---

## 🧠 Section 2: Code Output & Assertion Predictions

### Q16. Predict the console output when the button is tapped:
```dart
String? _nameValidator(String? val) {
  if (val == null || val.trim().isEmpty) return 'Name required';
  return null;
}

// In build():
TextFormField(
  initialValue: '   ',
  validator: _nameValidator,
),
ElevatedButton(
  onPressed: () {
    print(_formKey.currentState?.validate());
  },
  child: const Text('Submit'),
)
```
*Answer*: `false`. The `.trim()` removes spaces, resulting in an empty string which triggers the error return `'Name required'`.

### Q17. Will this code compile and run successfully? If not, what error occurs?
```dart
CupertinoTimerPicker(
  minuteInterval: 7,
  onTimerDurationChanged: (d) {},
)
```
*Answer*: It will compile, but crash at runtime with an assertion failure: `60 % minuteInterval == 0`. The integer `7` does not divide 60 evenly.

### Q18. Predict the output when the user types `"41112222"` into this field:
```dart
TextField(
  inputFormatters: [
    FilteringTextInputFormatter.digitsOnly,
    LengthLimitingTextInputFormatter(4),
  ],
)
```
*Answer*: `"4111"`. The `LengthLimitingTextInputFormatter` truncates input at 4 characters.

### Q19. What will be printed if the user confirms the time 2:15 PM in US English?
```dart
final time = const TimeOfDay(hour: 14, minute: 15);
print(time.format(context));
```
*Answer*: `"2:15 PM"`.

### Q20. What will be visually displayed in the `Autocomplete` text box if `displayStringForOption` is omitted?
```dart
class City {
  final String name;
  City(this.name);
}
// User picks City('Paris') from Autocomplete<City> without displayStringForOption
```
*Answer*: `"Instance of 'City'"` (the default `Object.toString()` output).

---

## 🔍 Section 3: Bug-Hunting & Debugging Challenges

### Q21. Identify the critical bug in this password matching form:
```dart
TextFormField(
  decoration: const InputDecoration(labelText: 'Password'),
  onChanged: (val) => _password = val,
),
TextFormField(
  decoration: const InputDecoration(labelText: 'Confirm Password'),
  validator: (val) {
    if (val != _password) return 'Passwords do not match';
    return null;
  },
),
```
*Bug*: If the user fills both fields correctly, but then returns to the first field and edits their password, the "Confirm Password" field is not automatically re-evaluated, allowing mismatched passwords to pass validation!
*Fix*: Attach a `TextEditingController` to the password field, and call `_formKey.currentState?.validate()` inside the first field's `onChanged` callback if the confirm field is not empty.

### Q22. Identify why the date picker crashes on launch:
```dart
final picked = await showDatePicker(
  context: context,
  initialDate: DateTime.now(),
  firstDate: DateTime(DateTime.now().year, DateTime.now().month, DateTime.now().day + 1),
  lastDate: DateTime(2030),
);
```
*Bug*: `initialDate` (`DateTime.now()`) is **before** `firstDate` (tomorrow). Flutter throws an assertion failure: `initialDate.isAfter(firstDate) || isAtSameMomentAs`.
*Fix*: Ensure `initialDate` is set to a date on or after `firstDate` (e.g. set `initialDate = firstDate`).

### Q23. Why is the user unable to type decimals into this price input on an iPhone?
```dart
TextField(
  keyboardType: TextInputType.number,
  decoration: const InputDecoration(labelText: 'Price'),
)
```
*Bug*: On iOS, `TextInputType.number` renders a 10-key integer keypad with no decimal point button.
*Fix*: Change to `keyboardType: const TextInputType.numberWithOptions(decimal: true)`.

### Q24. Explain why the cursor jumps to the start of the field when typing with this custom formatter:
```dart
class SpacingFormatter extends TextInputFormatter {
  @override
  TextEditingValue formatEditUpdate(TextEditingValue oldVal, TextEditingValue newVal) {
    return TextEditingValue(
      text: newVal.text.toUpperCase(),
    );
  }
}
```
*Bug*: The returned `TextEditingValue` does not specify `selection`. It defaults to index 0, causing the cursor to snap to the beginning on every keystroke.
*Fix*: Pass `selection: newVal.selection`.

### Q25. Why does this form throw an unmounted context warning?
```dart
void _onSubmit() async {
  final date = await showDatePicker(...);
  setState(() {
    _selectedDate = date;
  });
}
```
*Bug*: Calling `setState()` after an `await` without verifying if the widget is still mounted can throw `setState() called after dispose()` if the user navigated back during the dialog.
*Fix*: Add `if (!mounted) return;` immediately after the `await` line.

---

## 🛠 Section 4: Practical Coding Challenges

### Q26. Write a complete pure validator function in Dart that validates a password against:
1. Minimum 8 characters.
2. At least one uppercase letter.
3. At least one number.
4. At least one special symbol.

```dart
String? validateStrongPassword(String? value) {
  if (value == null || value.isEmpty) {
    return 'Password is required';
  }
  if (value.length < 8) {
    return 'Password must be at least 8 characters long';
  }
  if (!value.contains(RegExp(r'[A-Z]'))) {
    return 'Must include at least one uppercase letter';
  }
  if (!value.contains(RegExp(r'[0-9]'))) {
    return 'Must include at least one digit (0-9)';
  }
  if (!value.contains(RegExp(r'[!@#$%^&*(),.?":{}|<>]'))) {
    return 'Must include at least one special symbol';
  }
  return null;
}
```

### Q27. Write a custom `TextInputFormatter` that formats a 16-digit credit card number into 4 blocks separated by spaces (`XXXX XXXX XXXX XXXX`).

```dart
class CreditCardFormatter extends TextInputFormatter {
  @override
  TextEditingValue formatEditUpdate(
    TextEditingValue oldValue,
    TextEditingValue newValue,
  ) {
    final text = newValue.text.replaceAll(' ', '');
    final buffer = StringBuffer();

    for (int i = 0; i < text.length; i++) {
      buffer.write(text[i]);
      final index = i + 1;
      if (index % 4 == 0 && index != text.length) {
        buffer.write(' ');
      }
    }

    final formatted = buffer.toString();
    return TextEditingValue(
      text: formatted,
      selection: TextSelection.collapsed(offset: formatted.length),
    );
  }
}
```

### Q28. Write a function that safely combines a `DateTime date` and a `TimeOfDay time` into a single UTC timestamp.

```dart
DateTime combineDateAndTime(DateTime date, TimeOfDay time) {
  final localCombined = DateTime(
    date.year,
    date.month,
    date.day,
    time.hour,
    time.minute,
  );
  return localCombined.toUtc();
}
```

### Q29. Implement an `optionsBuilder` for `Autocomplete<String>` that performs a case-insensitive prefix search on a list of fruit names and caps suggestions at a maximum of 5 items.

```dart
Iterable<String> searchFruits(TextEditingValue textValue, List<String> fruits) {
  final query = textValue.text.trim().toLowerCase();
  if (query.isEmpty) {
    return const Iterable<String>.empty();
  }
  return fruits
      .where((fruit) => fruit.toLowerCase().startsWith(query))
      .take(5);
}
```

### Q30. Build a reusable `GestureDetector` wrapper that dismisses the virtual keyboard whenever the user taps anywhere on screen outside active input fields.

```dart
Widget dismissKeyboardOnTap({required BuildContext context, required Widget child}) {
  return GestureDetector(
    onTap: () => FocusScope.of(context).unfocus(),
    behavior: HitTestBehavior.opaque,
    child: child,
  );
}
```

---

## 📝 Summary
Mastering user input in Flutter requires balancing three pillars:
1. **Coordination**: Using `Form`, `GlobalKey<FormState>`, and `TextFormField` to cascade validation and saving.
2. **Formatting & Restraint**: Using `TextInputFormatter` and `TextInputType` to sanitize input live.
3. **Ergonomics & Accessibility**: Choosing the right controls (`CheckboxListTile`, `RadioListTile`, `SwitchListTile`, `SearchAnchor`) to maximize touch area and provide responsive user feedback.
