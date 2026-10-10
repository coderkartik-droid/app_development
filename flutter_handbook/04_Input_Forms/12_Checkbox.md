# Checkbox in Flutter

## 📖 Introduction
Not all data input in mobile applications involves typing text. Binary decisions—such as accepting terms and conditions, toggling notification preferences, or selecting multiple items from an e-commerce list—require intuitive, low-friction controls.

The [Checkbox](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/12_Checkbox.md) widget in Flutter provides a classic two-state (checked / unchecked) or three-state (checked / unchecked / indeterminate) selection box. True to Flutter's declarative philosophy, [Checkbox](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/12_Checkbox.md) is a **controlled widget**—it does not store its own checked state internally. Instead, it reflects whatever value you pass to it, and when tapped, notifies you through an `onChanged` callback so you can update your state and trigger a rebuild.

---

## 🤔 Why Do We Need It?
When collecting multi-choice options or binary agreements:
- **Radio buttons** only allow picking one single option from a group.
- **Switches** imply immediate, live setting activation (like turning on Wi-Fi or Bluetooth).
- **Checkboxes** are uniquely suited for:
  1. Selecting zero, one, or multiple items independently from a collection.
  2. Confirming explicit consent (e.g. *"I agree to the Terms & Conditions"*).
  3. Batch actions with an "indeterminate / Select All" master toggle.

Flutter also provides [CheckboxListTile](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/12_Checkbox.md), which bundles the checkbox with a title, subtitle, and an expanded touch target across the entire width of the row, significantly improving usability on small mobile touchscreens.

---

## 🌍 Real-Life Analogy
Think of a [Checkbox](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/12_Checkbox.md) as a **Paper Ballot Voting Form or Packing Checklist**:

```
+-------------------------------------------------------------+
|                     TRIP PACKING LIST                       |
|                                                             |
|   [✓] Passport & Boarding Pass (Checked / Done)             |
|   [ ] Sunscreen & Sunglasses (Unchecked / Remaining)        |
|   [-] Toiletries Bag (Indeterminate / Partially Packed)     |
|                                                             |
|   Each item is independent. Checking off your passport      |
|   doesn't uncheck your sunscreen!                           |
+-------------------------------------------------------------+
```

---

## 🧠 Internal Working
Understanding how [Checkbox](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/12_Checkbox.md) renders and dispatches events clarifies why `setState()` is required:

```
                      User Taps Checkbox
                              │
                              ▼
                      Gesture Recognizer
                              │
                              ▼
                  Checkbox.onChanged(bool? newValue)
                              │
                              ▼
                      setState(() {
                         _isChecked = newValue;
                      })
                              │
                              ▼
                      Flutter Rebuilds
                              │
                              ▼
        RenderToggleable paints checkmark animation
```

1. **RenderToggleable**: Checkbox relies on `RenderToggleable`, an internal render object that manages the tween animation for the checkmark path, the background fill, and the ripple splash effect.
2. **Stateless Control**: The widget receives `value: bool?`. If you tap the checkbox but your `onChanged` callback does not call `setState()` to update the variable passed into `value`, the checkbox will visually remain unchanged!
3. **Tristate Logic**: When `tristate: true` is configured, cycling happens as: `false` $\to$ `true` $\to$ `null` $\to$ `false`.

---

## ⚙ Syntax

### Basic Checkbox
```dart
Checkbox(
  value: _isChecked,
  onChanged: (bool? newValue) {
    setState(() {
      _isChecked = newValue ?? false;
    });
  },
)
```

### CheckboxListTile (Recommended for Mobile Rows)
```dart
CheckboxListTile(
  title: const Text('Accept Terms and Conditions'),
  subtitle: const Text('Must be accepted to continue'),
  value: _agreeToTerms,
  controlAffinity: ListTileControlAffinity.leading, // Checkbox on the left
  onChanged: (bool? value) {
    setState(() {
      _agreeToTerms = value ?? false;
    });
  },
)
```

---

## 📋 Important Properties

| Property | Type | Description | Default |
| :--- | :--- | :--- | :--- |
| `value` | `bool?` | Whether the checkbox is checked (`true`), unchecked (`false`), or indeterminate (`null`). | *Required* |
| `onChanged` | `ValueChanged<bool?>?` | Called when the user taps the checkbox. If `null`, the checkbox renders in a disabled gray state. | *Required* |
| `tristate` | `bool` | If `true`, permits `value` to be `null` to represent an indeterminate state. | `false` |
| `activeColor` | `Color?` | The fill color of the checkbox when `value` is `true`. | Theme default |
| `checkColor` | `Color?` | The color of the checkmark icon inside the box. | Theme default (usually white) |
| `shape` | `OutlinedBorder?` | Border shape of the checkbox (e.g. `RoundedRectangleBorder(borderRadius: BorderRadius.circular(4))`). | Rounded rectangle |
| `side` | `BorderSide?` | Color and width of the checkbox border when unchecked. | Theme default |

---

## 💻 Complete Runnable Example
The following production-ready example showcases:
1. A **Tristate "Select All" master checkbox** coordinating multiple child items.
2. Individual items using [CheckboxListTile](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/12_Checkbox.md).
3. A mandatory **"Terms of Service" checkbox** enabling a submission button.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const CheckboxMasterclassApp());
}

class CheckboxMasterclassApp extends StatelessWidget {
  const CheckboxMasterclassApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Checkbox Masterclass',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.teal),
        useMaterial3: true,
      ),
      home: const NotificationPreferencesScreen(),
    );
  }
}

class NotificationPreferencesScreen extends StatefulWidget {
  const NotificationPreferencesScreen({super.key});

  @override
  State<NotificationPreferencesScreen> createState() =>
      _NotificationPreferencesScreenState();
}

class _NotificationPreferencesScreenState
    extends State<NotificationPreferencesScreen> {
  // Child checklist items
  final Map<String, bool> _channels = {
    'Marketing Emails': true,
    'Order & Shipping Alerts': true,
    'Product Updates': false,
    'Security Notifications': true,
  };

  bool _agreeToTerms = false;

  // Calculates master checkbox state: true (all), false (none), null (some)
  bool? get _selectAllState {
    final allSelected = _channels.values.every((v) => v);
    final noneSelected = _channels.values.every((v) => !v);

    if (allSelected) return true;
    if (noneSelected) return false;
    return null; // Indeterminate state!
  }

  void _onSelectAllChanged(bool? value) {
    setState(() {
      // If currently indeterminate or false, toggle all to true; otherwise false
      final newValue = value ?? true;
      _channels.updateAll((key, _) => newValue);
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Preferences & Checkboxes'),
        centerTitle: true,
      ),
      body: SafeArea(
        child: ListView(
          padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 12),
          children: [
            const Text(
              'Notification Channels',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),

            // Master "Select All" Tristate Checkbox
            Card(
              elevation: 0,
              color: Colors.teal.shade50,
              shape: RoundedRectangleBorder(
                borderRadius: BorderRadius.circular(12),
                side: BorderSide(color: Colors.teal.shade200),
              ),
              child: CheckboxListTile(
                tristate: true,
                value: _selectAllState,
                title: const Text(
                  'Select All Notifications',
                  style: TextStyle(fontWeight: FontWeight.bold),
                ),
                subtitle: Text(
                  _selectAllState == null
                      ? 'Some channels enabled'
                      : _selectAllState!
                          ? 'All channels enabled'
                          : 'All channels disabled',
                ),
                controlAffinity: ListTileControlAffinity.leading,
                onChanged: _onSelectAllChanged,
              ),
            ),
            const Divider(height: 24),

            // Child Checkboxes
            ..._channels.keys.map((channel) {
              return CheckboxListTile(
                title: Text(channel),
                value: _channels[channel],
                controlAffinity: ListTileControlAffinity.leading,
                onChanged: (bool? value) {
                  setState(() {
                    _channels[channel] = value ?? false;
                  });
                },
              );
            }),

            const Divider(height: 32),

            // Terms & Conditions Checkbox
            CheckboxListTile(
              title: const Text('I agree to the Terms of Service & Privacy Policy'),
              subtitle: const Text('Required to proceed'),
              value: _agreeToTerms,
              controlAffinity: ListTileControlAffinity.leading,
              activeColor: Colors.teal,
              onChanged: (bool? value) {
                setState(() {
                  _agreeToTerms = value ?? false;
                });
              },
            ),
            const SizedBox(height: 20),

            // Submit Button enabled only when terms are accepted
            FilledButton.icon(
              onPressed: _agreeToTerms
                  ? () {
                      final selectedCount =
                          _channels.values.where((v) => v).length;
                      ScaffoldMessenger.of(context).showSnackBar(
                        SnackBar(
                          content: Text(
                              'Saved preferences with $selectedCount channels enabled!'),
                          backgroundColor: Colors.teal,
                        ),
                      );
                    }
                  : null, // Disabled when unchecked
              icon: const Icon(Icons.save),
              label: const Text('Save Preferences', style: TextStyle(fontSize: 16)),
              style: FilledButton.styleFrom(
                padding: const EdgeInsets.symmetric(vertical: 16),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## 🔍 Line-by-Line Example Explanation
- `bool? get _selectAllState`:
  - **What it does**: Computes whether all, none, or a subset of child options are selected.
  - **Why it exists**: Supplies the `value` parameter for the master tristate checkbox dynamically.
  - **Indeterminate behavior**: Returns `null` when some items are `true` and some are `false`, displaying a minus sign `-` in the checkbox.
- `tristate: true`:
  - **What it does**: Tells Flutter that this checkbox supports three distinct visual states: checked (`true`), unchecked (`false`), and indeterminate (`null`).
  - **What happens if omitted**: Flutter will throw an assertion error if `value` is `null` when `tristate` is `false`.
- `controlAffinity: ListTileControlAffinity.leading`:
  - **What it does**: Places the checkbox box to the left of the title text (leading edge).
  - **Why it exists**: Matches standard Western design conventions where checkboxes precede their labels.
- `onPressed: _agreeToTerms ? () { ... } : null`:
  - **What it does**: Disables the button when `_agreeToTerms` is `false`.
  - **Why it exists**: Guarantees that users cannot submit the form without explicitly ticking the checkbox.

---

## 📱 ASCII Phone Screen Preview
```
+------------------------------------------+
|        Preferences & Checkboxes          |
+------------------------------------------+
| Notification Channels                    |
|                                          |
| +--------------------------------------+ |
| | [-] Select All Notifications         | |
| |     Some channels enabled            | |
| +--------------------------------------+ |
| ---------------------------------------- |
| [✓] Marketing Emails                     |
| [✓] Order & Shipping Alerts              |
| [ ] Product Updates                      |
| [✓] Security Notifications               |
| ---------------------------------------- |
|                                          |
| [✓] I agree to the Terms of Service      |
|     Required to proceed                  |
|                                          |
| +--------------------------------------+ |
| |        [💾] Save Preferences         | |
| +--------------------------------------+ |
+------------------------------------------+
```

---

## 🌳 Widget Tree
```
MaterialApp
 └── NotificationPreferencesScreen (StatefulWidget)
      └── Scaffold
           ├── AppBar
           └── SafeArea
                └── ListView
                     ├── Text ("Notification Channels")
                     ├── Card
                     │    └── CheckboxListTile [tristate: true, value: _selectAllState]
                     ├── Divider
                     ├── CheckboxListTile ("Marketing Emails")
                     ├── CheckboxListTile ("Order Alerts")
                     ├── CheckboxListTile ("Product Updates")
                     ├── CheckboxListTile ("Security Notifications")
                     ├── Divider
                     ├── CheckboxListTile ("Agree to Terms")
                     └── FilledButton ("Save Preferences")
```

---

## ⚙ Flutter Execution Flow
1. **User Action**: User taps the unchecked "Product Updates" checkbox.
2. **Callback**: `CheckboxListTile.onChanged(true)` is fired.
3. **State Mutation**: `setState()` sets `_channels['Product Updates'] = true`.
4. **Re-Evaluation**:
   - `_selectAllState` getter recalculates.
   - Now, every value in `_channels` is `true`.
   - `_selectAllState` switches from `null` (dash `-`) to `true` (checkmark `✓`).
5. **Repaint**: Flutter redraws the screen; both the tapped item and the master checkbox render with solid checkmarks.

---

## ⚠ Common Beginner Mistakes

### 1. Expecting `Checkbox` to Maintain Its Own State
```dart
// ❌ BAD: Checkbox never updates!
Checkbox(
  value: false,
  onChanged: (val) {
    // Nothing happens because state isn't stored or rebuilt!
  },
)

// ✅ GOOD: Update state and trigger setState
Checkbox(
  value: _isChecked,
  onChanged: (val) {
    setState(() => _isChecked = val ?? false);
  },
)
```

### 2. Passing `null` When `tristate` is `false`
```dart
// ❌ CRASH: Throws assertion failure: "value != null || tristate"
Checkbox(
  value: null, // Tristate is false by default!
  onChanged: (val) {},
)

// ✅ GOOD: Either supply a boolean or enable tristate
Checkbox(
  tristate: true,
  value: null, // Valid: paints an indeterminate dash (-)
  onChanged: (val) {},
)
```

### 3. Using Raw Checkbox and Text Instead of CheckboxListTile
```dart
// ❌ POOR UX: Only the 18x18 pixel box is tappable; tapping the text does nothing!
Row(
  children: [
    Checkbox(value: _val, onChanged: ...),
    Text('Tap me'), // Not clickable!
  ],
)

// ✅ GOOD UX: The entire row width is responsive to touch!
CheckboxListTile(
  title: const Text('Tap me'),
  value: _val,
  onChanged: ...,
)
```

---

## 🚀 Professional Usage
- **E-Commerce Checkouts (e.g., Amazon, eBay)**: "Select all items in cart" uses a master tristate checkbox; individual cart items update the master total dynamically.
- **Privacy & GDPR Compliance**: Explicit checkboxes for analytics tracking, newsletter opt-in, and terms acceptance.
- **Task Management (e.g., Todoist, Google Keep)**: Toggling tasks complete strikethroughs text and reorders the list.

---

## 💡 Best Practices
1. **Always prefer `CheckboxListTile` for mobile forms**: Mobile screens require large touch targets (minimum $48 \times 48$ logical pixels) to avoid frustrating users.
2. **Use `controlAffinity: ListTileControlAffinity.leading`**: Placing the checkbox before the label aligns with standard mobile interaction expectations.
3. **Handle disabled states gracefully**: Passing `onChanged: null` renders the checkbox in a muted disabled theme; communicate why it is disabled.
4. **Wrap in FormField if participating in Form validation**: Use `FormField<bool>` to validate that a checkbox is checked before allowing form submission.

---

## 🧪 Try It Yourself
1. Change the checkbox shape to a circle using `shape: const CircleBorder()`.
2. Wrap the Terms & Conditions checkbox in a `FormField<bool>` and display red validation error text if unchecked on submit.
3. Customize `activeColor` and `checkColor` to match a brand theme (e.g., black box with gold checkmark).

---

## 🎯 Challenge
Build an interactive shopping cart item selection screen:
- A master `CheckboxListTile` that toggles all items.
- A list of 5 products with prices.
- A floating bottom bar that displays: `Selected Total: $XX.XX` and updates dynamically as items are checked or unchecked.

---

## ❓ Practice Questions

### Q1. Why does tapping a `Checkbox` not update its visual checkmark automatically?
- A) Flutter checkboxes are broken
- B) Checkbox is a controlled widget with no internal state; it reflects the `value` passed to it and requires `setState()`
- C) It only works on Android
- D) It requires a GlobalKey
*Answer: B* — Checkbox delegates state storage entirely to its parent widget.

### Q2. What visual mark is displayed when `tristate: true` and `value: null`?
- A) A question mark
- B) An empty white box
- C) A horizontal line or dash (`-`) representing an indeterminate state
- D) An exclamation point
*Answer: C* — Null represents an indeterminate state rendered as a horizontal dash.

### Q3. What is the major UX advantage of `CheckboxListTile` over a standalone `Checkbox`?
- A) It supports more colors
- B) The entire row (title, subtitle, and whitespace) becomes an accessible touch target
- C) It automatically saves to SQLite
- D) It requires no controller
*Answer: B* — It provides a full-width touch target meeting mobile accessibility guidelines.

### Q4. How do you disable a `Checkbox` so that it cannot be tapped?
- A) `enabled: false`
- B) `onChanged: null`
- C) `clickable: false`
- D) `value: null`
*Answer: B* — Passing `null` to `onChanged` disables the widget and renders it in a disabled style.

### Q5. Which property moves the checkbox from the right side of a `CheckboxListTile` to the left side?
- A) `alignment: Alignment.centerLeft`
- B) `controlAffinity: ListTileControlAffinity.leading`
- C) `position: CheckboxPosition.left`
- D) `leading: true`
*Answer: B* — `ListTileControlAffinity.leading` positions the control on the leading edge.

### Q6. What error occurs if you pass `value: null` to a `Checkbox` without setting `tristate: true`?
*Answer*: Flutter throws a runtime assertion failure stating that `value != null || tristate`.

### Q7. How can you customize the corner roundness of a `Checkbox`?
*Answer*: Configure the `shape` property with a `RoundedRectangleBorder(borderRadius: BorderRadius.circular(x))`.

### Q8. Output Prediction: What will `_selectAllState` return if a list contains `[true, false, true]`?
*Answer*: `null`. Because some items are true and some are false, it represents an indeterminate state.

### Q9. Can a `Checkbox` be used directly inside a `Form` widget for validation with `_formKey.currentState!.validate()`?
*Answer*: Not directly. A raw `Checkbox` is not a `FormField`. You must wrap it inside a `FormField<bool>` or use a package offering `CheckboxFormField`.

### Q10. Write a minimal snippet that builds a disabled, checked `Checkbox`.
*Answer*:
```dart
const Checkbox(
  value: true,
  onChanged: null, // null callback disables the widget
)
```

---

## 📝 Summary
- [Checkbox](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/12_Checkbox.md) is a controlled widget for binary and indeterminate selections.
- Setting `tristate: true` unlocks three-state logic (`true`, `false`, `null`) for parent "Select All" controllers.
- Always use [CheckboxListTile](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/12_Checkbox.md) in form lists to provide accessible, full-width touch targets.
- Disabling is achieved declaratively by passing `onChanged: null`.
