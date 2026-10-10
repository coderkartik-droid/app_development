# Quick Revision: Input & Forms in Flutter

## 🚀 Module Overview
This quick revision cheat sheet condenses the essential concepts, architectures, syntax patterns, and best practices across the entire **04_Input_Forms** module into a high-yield reference guide.

---

## 📊 Comprehensive Widget Comparison Matrix

| Widget | Purpose | State Management | Key Properties / Methods | Common Gotcha |
| :--- | :--- | :--- | :--- | :--- |
| **`TextField`** | Basic single or multi-line text input | Uses `TextEditingController` | `controller`, `decoration`, `keyboardType`, `inputFormatters`, `onChanged` | Recreating controller inside `build()` causes loss of focus/text. |
| **`TextEditingController`** | Reads, mutates, and monitors input text & selection | Extends `ChangeNotifier` | `text`, `selection`, `clear()`, `addListener()`, `dispose()` | Forgetting to call `.dispose()` causes memory leaks. |
| **`InputDecoration`** | Visual styling of text boxes | Immutable configuration | `labelText`, `hintText`, `prefixIcon`, `suffixIcon`, `border`, `errorText` | Missing `border: OutlineInputBorder()` leaves unstyled underline. |
| **`TextFormField`** | Form-integrated text box with validation | Managed by `FormFieldState` | `validator`, `onSaved`, `initialValue`, `autovalidateMode` | Don't provide both `controller` and non-null `initialValue`. |
| **`Form`** | Container coordinating a collection of `FormField`s | Injects `_FormScope` | `key: GlobalKey<FormState>()`, `autovalidateMode`, `canPop` | Using raw `TextField` inside `Form` (ignored during `validate()`). |
| **`GlobalKey<FormState>`**| Cross-tree handle to trigger form actions | Central engine registry | `currentState!.validate()`, `save()`, `reset()` | Instantiating inside `build()` wipes form state on every keystroke. |
| **`FocusNode`** | Controls which element receives keyboard events | Extends `ChangeNotifier` | `requestFocus()`, `unfocus()`, `hasFocus`, `dispose()` | Requesting focus in `initState()` synchronously before mount. |
| **`TextInputType`** | Recommends soft keyboard layout to OS | Configures native IME | `.text`, `.emailAddress`, `.phone`, `.numberWithOptions(decimal: true)` | Assuming `number` prevents typing letters without formatters. |
| **`TextInputFormatter`** | Intercepts & transforms text live | Runs `formatEditUpdate` | `digitsOnly`, `allow(regex)`, `deny(regex)`, `LengthLimiting` | Forgetting to preserve or adjust `TextSelection` offsets. |
| **`Checkbox`** | Binary or tristate selection box | Controlled widget | `value`, `onChanged`, `tristate: true` | Expecting it to toggle without calling `setState()`. |
| **`CheckboxListTile`** | Full-width row with title & checkbox | Controlled widget | `title`, `subtitle`, `controlAffinity: leading`, `value` | Using raw tiny Checkbox instead of full-row touch target. |
| **`Radio<T>`** | Mutually exclusive single selection | Controlled widget | `value`, `groupValue`, `onChanged` | Using strings instead of strongly typed `enum`s. |
| **`RadioListTile<T>`** | Full-width row for radio choice | Controlled widget | `value`, `groupValue`, `title`, `secondary`, `onChanged` | Mismatching generic types across group items. |
| **`Switch`** | Instantaneous ON/OFF state toggle | Controlled widget | `value`, `onChanged`, `thumbIcon`, `activeTrackColor` | Using Switch for form selections that require a "Submit" button. |
| **`Switch.adaptive`** | Platform-adaptive toggle (iOS/Android) | Native look and feel | `value`, `onChanged`, `applyCupertinoTheme` | Inconsistent layout styling across platforms. |
| **`Slider`** | Draggable continuous/discrete numeric scale | Controlled widget | `value`, `min`, `max`, `divisions`, `label`, `onChangeEnd` | Initializing `value` outside `[min, max]` throws assertion error. |
| **`RangeSlider`** | Dual-thumb min-max interval selection | Controlled widget | `values: RangeValues(start, end)`, `min`, `max`, `divisions` | Calling heavy network queries inside `onChanged` instead of `onChangeEnd`.|
| **`DropdownButton`** | Space-efficient popup menu selection | Pushes modal overlay | `value`, `items: List<DropdownMenuItem>`, `isExpanded` | Duplicate `value`s in items list throws assertion failure. |
| **`DropdownButtonFormField`**| Form-ready dropdown with validation | Extends `FormField` | `validator`, `onSaved`, `decoration`, `items`, `value` | Forgetting to reset child dropdown value in cascading pickers. |
| **`CalendarDatePicker`**| Embedded inline Material calendar grid | Controlled widget | `initialDate`, `firstDate`, `lastDate`, `selectableDayPredicate`| Not stripping hours/minutes from `DateTime.now()` (timezone bugs). |
| **`CupertinoDatePicker`**| iOS-style spinning drum wheel picker | Wheel scroll physics | `mode: date`, `minimumDate`, `maximumDate`, `onDateTimeChanged` | Placing inside unconstrained `Column` without a `SizedBox(height: 200)`.|
| **`CupertinoTimerPicker`**| Spinning wheel for hours/mins/seconds | Controlled widget | `mode: hms/ms`, `initialTimerDuration`, `minuteInterval` | `minuteInterval` not dividing 60 evenly throws assertion crash. |
| **`showDatePicker`** | Modal dialog returning `Future<DateTime?>` | Dialog route | `await showDatePicker(context, initialDate, firstDate, lastDate)`| Forgetting to check `if (!mounted) return;` across async gap. |
| **`showDateRangePicker`**| Modal full-screen range picker | Dialog route | `await showDateRangePicker(context, firstDate, lastDate)` | Not handling `null` when user cancels dialog. |
| **`showTimePicker`** | Modal clock dial returning `Future<TimeOfDay?>`| Dialog route | `initialTime`, `initialEntryMode: dial/input`, `builder` | Calling `.toString()` instead of `.format(context)`. |
| **`SearchBar`** | Material 3 search bar container | Material 3 specs | `controller`, `leading`, `trailing`, `onChanged`, `onSubmitted` | Confusing with `TextField` without search styling. |
| **`SearchAnchor`** | SearchBar + full-screen suggestions view | Overlay controller | `searchController`, `builder`, `suggestionsBuilder` | Forgetting to call `controller.closeView(val)` upon suggestion tap. |
| **`Autocomplete<T>`** | Floating typeahead dropdown attached to field | CompositedTransform | `optionsBuilder`, `onSelected`, `displayStringForOption` | Missing `displayStringForOption` prints `"Instance of 'Object'"`. |

---

## 🌳 Form & Input Decision Tree

```
                            WHAT DO YOU NEED TO COLLECT?
                                         │
        ┌────────────────────────────────┼────────────────────────────────┐
        ▼                                ▼                                ▼
  Text / Freeform                   Binary / Choice               Time / Date / Search
        │                                │                                │
  ┌─────┴─────┐                    ┌─────┴─────┐                    ┌─────┴─────┐
  ▼           ▼                    ▼           ▼                    ▼           ▼
Single     Part of            Mutually      Multiple/            Calendar/   Search /
Field       Form              Exclusive      Instant              Clock     Typeahead
  │           │                    │           │                    │           │
TextField TextFormField         Radio /     Checkbox /          Picker      SearchBar /
                               RadioList   Switch / Slider     Functions   Autocomplete
```

---

## ⚡ High-Frequency Code Snippets

### 1. Unified Form Validation & Save
```dart
final GlobalKey<FormState> _formKey = GlobalKey<FormState>();

void _submit() {
  if (_formKey.currentState?.validate() ?? false) {
    _formKey.currentState?.save(); // Runs onSaved on all child FormFields
    print('Form is valid and values are saved!');
  }
}
```

### 2. Dismiss Keyboard on Tap Outside
```dart
GestureDetector(
  onTap: () => FocusScope.of(context).unfocus(),
  behavior: HitTestBehavior.opaque,
  child: Scaffold(...),
)
```

### 3. Digits-Only Numeric Formatter with Length Cap
```dart
TextField(
  keyboardType: TextInputType.number,
  inputFormatters: [
    FilteringTextInputFormatter.digitsOnly,
    LengthLimitingTextInputFormatter(10),
  ],
)
```

### 4. Decimal-Safe Numeric Keyboard (iOS & Android Compatible)
```dart
TextField(
  keyboardType: const TextInputType.numberWithOptions(decimal: true),
)
```

### 5. Safe Modal Date Picker Call
```dart
Future<void> _pickDate(BuildContext context) async {
  final now = DateTime.now();
  final today = DateTime(now.year, now.month, now.day);

  final DateTime? picked = await showDatePicker(
    context: context,
    initialDate: today,
    firstDate: today,
    lastDate: today.add(const Duration(days: 365)),
  );

  if (!context.mounted) return; // Guard across async gap!
  if (picked != null) {
    // Process selected date safely
  }
}
```

### 6. Localized Time Formatting
```dart
final TimeOfDay time = const TimeOfDay(hour: 14, minute: 30);
final String formatted = time.format(context); // "2:30 PM" or "14:30"
```

---

## 🧹 Memory Leak & Lifecycle Checklist

| Class | Where to Instantiate | Where to Dispose | Risk if Ignored |
| :--- | :--- | :--- | :--- |
| `TextEditingController` | `initState()` or state field | `dispose()` | Leaks listener subscriptions; continues listening in background. |
| `FocusNode` | `initState()` or state field | `dispose()` | Trapped native engine focus hooks; keyboard may fail to dismiss. |
| `SearchController` | `initState()` or state field | `dispose()` | Unclosed overlay listeners and query stream leaks. |
| `GlobalKey<FormState>` | State field (persistent) | *Garbage collected* | **Never instantiate inside `build()`**; wipes form state on every frame! |

---

## ⚡ Common Gotchas & Rapid Fixes

1. **Assertion Error: `initialDate.isAfter(firstDate)`**
   - *Fix*: Strip time from `DateTime.now()` via `DateTime(now.year, now.month, now.day)` before passing into `firstDate`.
2. **Assertion Error: `value != null || tristate`**
   - *Fix*: If passing `null` to a `Checkbox`, explicitly enable `tristate: true`.
3. **Assertion Error: `items with value: X appears more than once`**
   - *Fix*: Ensure every `DropdownMenuItem` has a distinct, unique `value`.
4. **Assertion Error: `value >= min && value <= max`**
   - *Fix*: Ensure the initial `Slider.value` falls strictly inside `[min, max]`.
5. **RenderBox Unbounded Height Exception in Cupertino Pickers**
   - *Fix*: Wrap `CupertinoDatePicker` or `CupertinoTimerPicker` in `SizedBox(height: 200)`.
6. **Linter Warning: "Don't use BuildContext across async gaps"**
   - *Fix*: Insert `if (!mounted) return;` immediately following `await showDatePicker(...)` or `await showTimePicker(...)`.
7. **Text Field Cursor Snapping to Beginning During Live Masking**
   - *Fix*: In your custom `TextInputFormatter`, always assign `selection: TextSelection.collapsed(offset: formatted.length)`.
8. **Form.validate() completely ignores your `TextField`**
   - *Fix*: Replace `TextField` with `TextFormField` or `DropdownButtonFormField`.
