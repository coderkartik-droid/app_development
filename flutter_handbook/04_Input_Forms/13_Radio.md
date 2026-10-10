# Radio & RadioListTile in Flutter

## 📖 Introduction
While checkboxes allow selecting multiple options independently, many user decisions require strict mutual exclusivity. When choosing a delivery speed (Standard, Express, or Overnight) or selecting a payment method (Credit Card, PayPal, or Apple Pay), picking one option must automatically deselect any prior choice.

In Flutter, this mutually exclusive selection pattern is implemented via [Radio](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/13_Radio.md) and [RadioListTile](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/13_Radio.md). Like checkboxes, radio buttons are controlled widgets. A group of radio buttons is coordinated not by a parent container widget, but by sharing a common **group value** variable. Whenever a button's distinct `value` matches the shared `groupValue`, that button renders filled with an inner circular dot.

---

## 🤔 Why Do We Need It?
Without a dedicated radio component:
- You would need to manually write logic unchecking five boolean variables whenever one variable is checked.
- Managing multiple selection states becomes error-prone as option lists expand.
- Accessibility tools wouldn't announce the options as a unified "one-of-many" radio group to screen readers.

With [Radio](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/13_Radio.md) and [RadioListTile](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/13_Radio.md):
- **Type Safety**: Strongly typed with generics `Radio<T>`.
- **Automatic De-selection**: Changing the `groupValue` automatically clears the previous selection and activates the new one across the entire group.
- **Full Touch Target**: `RadioListTile` allows tapping anywhere on the text, icon, or row to select that option.

---

## 🌍 Real-Life Analogy
Think of a group of Radio buttons as the **Physical Push-Buttons on an Old Car Stereo**:

```
+-------------------------------------------------------------+
|                     VINTAGE CAR RADIO                       |
|                                                             |
|   [ 88.5 FM ]    [ 94.7 FM ]    [ 101.1 FM ]    [ 106.9 FM ]|
|   (Depressed)    (Popped Out)   (Popped Out)    (Popped Out)|
|                                                             |
|   When you push the [ 101.1 FM ] button down:               |
|   -> A mechanical lever INSIDE instantly pops the           |
|      [ 88.5 FM ] button back out!                           |
|                                                             |
|   Only ONE station button can ever be depressed at a time.  |
+-------------------------------------------------------------+
```

---

## 🧠 Internal Working
A radio group in Flutter does not require a wrapping "RadioGroup" parent widget. Mutual exclusivity is achieved through a shared state variable:

```
                            Parent State
                      _selectedSpeed = Speed.fast
                                 │
             ┌───────────────────┼───────────────────┐
             │                   │                   │
             ▼                   ▼                   ▼
      Radio<Speed>        Radio<Speed>        Radio<Speed>
     value: Speed.slow   value: Speed.fast   value: Speed.express
     group: _selected    group: _selected    group: _selected
             │                   │                   │
      (slow != fast)      (fast == fast)      (express != fast)
             │                   │                   │
             ▼                   ▼                   ▼
       [ ] Unchecked       (•) CHECKED         [ ] Unchecked
```

1. Each `Radio<T>` has its own distinct identity represented by `value: T`.
2. All radios in the group bind to the same state property `groupValue: T`.
3. During layout painting, the radio asks: `value == groupValue`.
   - If `true`, the inner dot animates into view.
   - If `false`, only the circular border is painted.
4. When tapped, the radio calls `onChanged(value)`, passing its own `value` to your callback. Calling `setState(() => _groupValue = newValue)` triggers a rebuild, updating every button in the group.

---

## ⚙ Syntax

### Standalone Radio
```dart
Radio<PaymentMethod>(
  value: PaymentMethod.creditCard, // Unique value for this button
  groupValue: _selectedMethod,     // Shared state variable
  onChanged: (PaymentMethod? value) {
    setState(() {
      _selectedMethod = value;
    });
  },
)
```

### RadioListTile (Recommended for Mobile UI)
```dart
RadioListTile<PaymentMethod>(
  title: const Text('Credit Card'),
  subtitle: const Text('Visa, Mastercard, Amex'),
  secondary: const Icon(Icons.credit_card),
  value: PaymentMethod.creditCard,
  groupValue: _selectedMethod,
  onChanged: (PaymentMethod? value) {
    setState(() {
      _selectedMethod = value;
    });
  },
)
```

---

## 📋 Important Properties

| Property | Type | Description | Default |
| :--- | :--- | :--- | :--- |
| `value` | `T` | The unique value this radio button represents. | *Required* |
| `groupValue` | `T?` | The currently selected value for the entire radio group. | *Required* |
| `onChanged` | `ValueChanged<T?>?` | Called when this button is tapped. Passing `null` disables the button. | *Required* |
| `activeColor` | `Color?` | The color of the inner circular dot when selected. | Theme primary color |
| `title` *(ListTile)* | `Widget?` | The primary text label beside the radio button. | `null` |
| `subtitle` *(ListTile)* | `Widget?` | Secondary explanatory text below the title. | `null` |
| `secondary` *(ListTile)* | `Widget?` | An icon or widget displayed opposite the radio button. | `null` |
| `controlAffinity` *(ListTile)* | `ListTileControlAffinity` | Where to place the radio circle (`leading` on the left, `trailing` on the right). | `platform` |

---

## 💻 Complete Runnable Example
The following production-grade application demonstrates a complete checkout shipping selection flow using typed Enums, dynamic price calculation, and [RadioListTile](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/13_Radio.md).

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const RadioMasterclassApp());
}

class RadioMasterclassApp extends StatelessWidget {
  const RadioMasterclassApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Radio & RadioListTile Masterclass',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const DeliverySelectionScreen(),
    );
  }
}

// 1. Define strongly-typed enum for shipping choices
enum DeliverySpeed {
  standard(name: 'Standard Ground', price: 4.99, days: '3-5 Business Days', icon: Icons.local_shipping),
  express(name: 'Express Air', price: 12.99, days: '1-2 Business Days', icon: Icons.flight),
  overnight(name: 'Priority Overnight', price: 24.99, days: 'Next Morning by 10 AM', icon: Icons.bolt);

  final String name;
  final double price;
  final String days;
  final IconData icon;

  const DeliverySpeed({
    required this.name,
    required this.price,
    required this.days,
    required this.icon,
  });
}

class DeliverySelectionScreen extends StatefulWidget {
  const DeliverySelectionScreen({super.key});

  @override
  State<DeliverySelectionScreen> createState() =>
      _DeliverySelectionScreenState();
}

class _DeliverySelectionScreenState extends State<DeliverySelectionScreen> {
  // 2. Shared group value initialized to default choice
  DeliverySpeed _selectedSpeed = DeliverySpeed.standard;

  final double _subtotal = 89.50;

  @override
  Widget build(BuildContext context) {
    final totalPrice = _subtotal + _selectedSpeed.price;

    return Scaffold(
      appBar: AppBar(
        title: const Text('Checkout: Delivery Speed'),
        centerTitle: true,
      ),
      body: SafeArea(
        child: Column(
          children: [
            Expanded(
              child: ListView(
                padding: const EdgeInsets.all(16.0),
                children: [
                  const Text(
                    'Select Shipping Method',
                    style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
                  ),
                  const SizedBox(height: 6),
                  const Text(
                    'Choose the delivery timeline that best fits your needs.',
                    style: TextStyle(color: Colors.grey),
                  ),
                  const SizedBox(height: 16),

                  // Generate RadioListTile options dynamically from Enum
                  ...DeliverySpeed.values.map((speed) {
                    final isSelected = _selectedSpeed == speed;
                    return Card(
                      elevation: isSelected ? 2 : 0,
                      margin: const EdgeInsets.only(bottom: 12),
                      shape: RoundedRectangleBorder(
                        borderRadius: BorderRadius.circular(12),
                        side: BorderSide(
                          color: isSelected
                              ? Theme.of(context).colorScheme.primary
                              : Colors.grey.shade300,
                          width: isSelected ? 2 : 1,
                        ),
                      ),
                      child: RadioListTile<DeliverySpeed>(
                        value: speed,
                        groupValue: _selectedSpeed,
                        title: Row(
                          children: [
                            Text(
                              speed.name,
                              style: const TextStyle(fontWeight: FontWeight.bold),
                            ),
                            const Spacer(),
                            Text(
                              '\$${speed.price.toStringAsFixed(2)}',
                              style: TextStyle(
                                fontWeight: FontWeight.bold,
                                color: Theme.of(context).colorScheme.primary,
                              ),
                            ),
                          ],
                        ),
                        subtitle: Text(speed.days),
                        secondary: Icon(speed.icon, color: Colors.deepPurple),
                        controlAffinity: ListTileControlAffinity.leading,
                        onChanged: (DeliverySpeed? value) {
                          if (value != null) {
                            setState(() {
                              _selectedSpeed = value;
                            });
                          }
                        },
                      ),
                    );
                  }),
                ],
              ),
            ),

            // Order Summary Footer
            Container(
              padding: const EdgeInsets.all(20.0),
              decoration: BoxDecoration(
                color: Colors.white,
                boxShadow: [
                  BoxShadow(
                    color: Colors.black.withOpacity(0.05),
                    blurRadius: 10,
                    offset: const Offset(0, -4),
                  ),
                ],
              ),
              child: Column(
                children: [
                  Row(
                    mainAxisAlignment: MainAxisAlignment.spaceBetween,
                    children: [
                      const Text('Subtotal:'),
                      Text('\$${_subtotal.toStringAsFixed(2)}'),
                    ],
                  ),
                  const SizedBox(height: 6),
                  Row(
                    mainAxisAlignment: MainAxisAlignment.spaceBetween,
                    children: [
                      Text('Shipping (${_selectedSpeed.name}):'),
                      Text('\$${_selectedSpeed.price.toStringAsFixed(2)}'),
                    ],
                  ),
                  const Divider(height: 20),
                  Row(
                    mainAxisAlignment: MainAxisAlignment.spaceBetween,
                    children: [
                      const Text('Order Total:', style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
                      Text(
                        '\$${totalPrice.toStringAsFixed(2)}',
                        style: TextStyle(
                          fontSize: 20,
                          fontWeight: FontWeight.bold,
                          color: Theme.of(context).colorScheme.primary,
                        ),
                      ),
                    ],
                  ),
                  const SizedBox(height: 16),
                  SizedBox(
                    width: double.infinity,
                    child: FilledButton(
                      onPressed: () {
                        ScaffoldMessenger.of(context).showSnackBar(
                          SnackBar(
                            content: Text('Order placed with ${_selectedSpeed.name}!'),
                            backgroundColor: Colors.deepPurple,
                          ),
                        );
                      },
                      style: FilledButton.styleFrom(
                        padding: const EdgeInsets.symmetric(vertical: 16),
                      ),
                      child: const Text('Confirm Shipping Option', style: TextStyle(fontSize: 16)),
                    ),
                  ),
                ],
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
- `enum DeliverySpeed { standard, express, overnight }`:
  - **What it does**: Declares an enhanced Dart enum encapsulating all options, prices, and display text.
  - **Why it exists**: Avoids brittle raw strings (like `"Standard"`) that invite typo bugs and make refactoring difficult.
- `DeliverySpeed _selectedSpeed = DeliverySpeed.standard;`:
  - **What it does**: Holds the current selection state for the entire radio group.
  - **Why it exists**: Serves as the single source of truth passed to `groupValue`.
- `RadioListTile<DeliverySpeed>(value: speed, groupValue: _selectedSpeed, ...)`:
  - **What it does**: Instantiates a typed radio list tile.
  - **Internal behavior**: Evaluates `speed == _selectedSpeed`. If equal, the inner dot is rendered; if not, it remains empty.
- `onChanged: (DeliverySpeed? value) { if (value != null) setState(() => _selectedSpeed = value); }`:
  - **What it does**: Updates the shared state variable with the tapped button's enum value and triggers an instant UI rebuild.
  - **What happens if omitted**: Tapping options produces no visual change on screen.

---

## 📱 ASCII Phone Screen Preview
```
+------------------------------------------+
|        Checkout: Delivery Speed          |
+------------------------------------------+
| Select Shipping Method                   |
| Choose the delivery timeline...          |
|                                          |
| +--------------------------------------+ |
| | (•) [🚚] Standard Ground     $4.99   | |
| |          3-5 Business Days           | |
| +--------------------------------------+ |
|                                          |
| +--------------------------------------+ |
| | ( ) [✈] Express Air         $12.99   | |
| |          1-2 Business Days           | |
| +--------------------------------------+ |
|                                          |
| +--------------------------------------+ |
| | ( ) [⚡] Priority Overnight   $24.99   | |
| |          Next Morning by 10 AM       | |
| +--------------------------------------+ |
|                                          |
| ======================================== |
| Subtotal:                         $89.50 |
| Shipping (Standard Ground):        $4.99 |
| Order Total:                      $94.49 |
|                                          |
| +--------------------------------------+ |
| |     Confirm Shipping Option          | |
| +--------------------------------------+ |
+------------------------------------------+
```

---

## 🌳 Widget Tree
```
MaterialApp
 └── DeliverySelectionScreen (StatefulWidget)
      └── Scaffold
           ├── AppBar
           └── SafeArea
                └── Column
                     ├── Expanded
                     │    └── ListView
                     │         ├── Text ("Select Shipping Method")
                     │         ├── Card -> RadioListTile<DeliverySpeed> (standard)
                     │         ├── Card -> RadioListTile<DeliverySpeed> (express)
                     │         └── Card -> RadioListTile<DeliverySpeed> (overnight)
                     └── Container (Summary Footer)
                          └── Column
                               ├── Row (Subtotal)
                               ├── Row (Shipping)
                               ├── Divider
                               ├── Row (Total)
                               └── FilledButton ("Confirm")
```

---

## ⚙ Flutter Execution Flow
1. **User Action**: The user taps the "Priority Overnight" card.
2. **Hit Test & Callback**: `RadioListTile.onChanged(DeliverySpeed.overnight)` is invoked.
3. **State Mutation**: `setState()` assigns `_selectedSpeed = DeliverySpeed.overnight`.
4. **Rebuild**:
   - `totalPrice` recalculates from `$94.49` to `$114.49`.
   - "Standard Ground" evaluates `speed == _selectedSpeed` $\to$ `false` (clears dot).
   - "Priority Overnight" evaluates `speed == _selectedSpeed` $\to$ `true` (animates dot in).
   - The card border animates to bold purple.
5. **Frame Paint**: Flutter paints the updated summary total and new radio selection.

---

## ⚠ Common Beginner Mistakes

### 1. Hardcoding Raw Strings with Inconsistent Casing
```dart
// ❌ BAD: Prone to silent bugs due to typos or casing differences
Radio<String>(
  value: 'PayPal',
  groupValue: 'paypal', // 'PayPal' != 'paypal' -> Never selected!
  onChanged: (val) {},
)

// ✅ GOOD: Use type-safe Enums
enum PaymentType { payPal, creditCard }
Radio<PaymentType>(
  value: PaymentType.payPal,
  groupValue: _selectedType,
  onChanged: (val) {},
)
```

### 2. Passing the Same Value to Every Button
```dart
// ❌ BAD: All buttons share the exact same value!
Radio(value: 1, groupValue: _val, onChanged: ...),
Radio(value: 1, groupValue: _val, onChanged: ...), // Both activate simultaneously!

// ✅ GOOD: Every button must have its own distinct value
Radio(value: 1, groupValue: _val, onChanged: ...),
Radio(value: 2, groupValue: _val, onChanged: ...),
```

### 3. Forgetting Generic Type Parameter
```dart
// ❌ RISKY: Untyped Radio defaults to dynamic
Radio(
  value: 'Admin',
  groupValue: 1, // Dynamic comparison allows comparing String to int without error!
  onChanged: (val) {},
)

// ✅ GOOD: Strongly typed
Radio<UserRole>(
  value: UserRole.admin,
  groupValue: _currentRole,
  onChanged: (val) {},
)
```

---

## 🚀 Professional Usage
- **Ride-Sharing (e.g., Uber, Lyft)**: Selecting ride tier (UberX, Comfort, Black) where only one vehicle type can be hailed per booking.
- **Surveys & Polling (e.g., Google Forms, SurveyMonkey)**: Single-choice multiple choice questionnaire options.
- **Language / Locale Switchers**: Selecting app display language (English, Spanish, French) in settings.

---

## 💡 Best Practices
1. **Always use Enums**: Enums eliminate string typo bugs and guarantee compile-time exhaustiveness checking.
2. **Prefer `RadioListTile` over raw `Radio`**: Tapping small 20-pixel circles is frustrating on mobile; `RadioListTile` turns the entire row into a responsive touch area.
3. **Keep `groupValue` nullable if no default selection exists**: If users must actively make a choice (like answering an exam question), initialize `groupValue = null`.
4. **Use `controlAffinity: ListTileControlAffinity.leading`**: Placing the radio circle on the left matches standard UI expectations.

---

## 🧪 Try It Yourself
1. Add a fourth shipping option (`Pick up at Store: $0.00`) to the `DeliverySpeed` enum and verify that the UI and price calculation update automatically.
2. Set the initial `_selectedSpeed` to `null` and disable the "Confirm" button until the user actively selects one option.
3. Change `activeColor` to green and observe how the selected radio dot and borders repaint.

---

## 🎯 Challenge
Create a dynamic Quiz Question Widget:
- Display a question: *"What is the capital of Australia?"*
- Display 4 options using `RadioListTile`: Sydney, Melbourne, Canberra, Brisbane.
- After tapping "Submit Answer", if correct, highlight the selected radio in green; if wrong, highlight the selected radio in red and the correct answer in green.

---

## ❓ Practice Questions

### Q1. How is mutual exclusivity established among a group of `Radio` widgets in Flutter?
- A) By wrapping them in a `RadioGroup` widget
- B) By giving them the same `groupValue` variable
- C) By giving them the same `value` property
- D) Flutter handles it automatically across all radios on the screen
*Answer: B* — Radios that share the same `groupValue` coordinate mutual exclusivity.

### Q2. Under what condition does a `Radio` widget render as selected?
- A) When `value == true`
- B) When `value == groupValue`
- C) When `onChanged` is not null
- D) When `checked: true` is passed
*Answer: B* — The radio compares its own unique `value` against the shared `groupValue`.

### Q3. Why is using an `enum` preferred over `String` for radio values?
- A) Enums run faster on the GPU
- B) Enums provide compile-time type safety, eliminate typo bugs, and allow exhaustive switch statements
- C) Flutter requires enums for Radio widgets
- D) Enums automatically save data to disk
*Answer: B* — Enums eliminate runtime typo errors and ensure strictly controlled option sets.

### Q4. What happens when you pass `onChanged: null` to a `Radio` or `RadioListTile`?
- A) The button crashes on tap
- B) The button becomes disabled and renders in a muted gray style
- C) The button is automatically selected
- D) The button disappears from the screen
*Answer: B* — Disables touch interactions and paints the control in the theme's disabled color.

### Q5. What is the role of `secondary` in `RadioListTile`?
- A) A backup value if the primary value fails
- B) A secondary label displayed under the subtitle
- C) A widget (often an `Icon` or `Image`) displayed opposite the radio control
- D) An alternative callback function
*Answer: C* — Displays leading/trailing visual elements like icons or avatars.

### Q6. Can a user uncheck a selected `Radio` button by tapping it again?
*Answer*: Not by default. In standard radio behavior, tapping an already selected radio triggers `onChanged` with the same value, keeping it selected. If deselectable behavior is needed, custom logic inside `onChanged` can set `_groupValue = null` when `value == _groupValue`.

### Q7. What is the difference between `value` and `groupValue`?
*Answer*: `value` is the specific constant value that this individual radio button represents. `groupValue` is the currently selected value for the entire group.

### Q8. Output Prediction: Which radio button will be visually selected?
```dart
Radio<int>(value: 1, groupValue: 2, onChanged: (v) {}),
Radio<int>(value: 2, groupValue: 2, onChanged: (v) {}),
Radio<int>(value: 3, groupValue: 2, onChanged: (v) {}),
```
*Answer*: The second radio (`value: 2`) because its `value` matches `groupValue: 2`.

### Q9. How do you position the radio circle on the leading edge (left side) in a `RadioListTile`?
*Answer*: Set `controlAffinity: ListTileControlAffinity.leading`.

### Q10. Write a minimal code snippet declaring an enum and a state variable for a light/dark theme radio selector.
*Answer*:
```dart
enum AppTheme { light, dark, system }
AppTheme _selectedTheme = AppTheme.system;
```

---

## 📝 Summary
- [Radio](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/13_Radio.md) enforces mutual exclusivity: selecting one choice deselects all others.
- Grouping is determined by sharing a common `groupValue` state variable across buttons.
- A button displays as selected when `value == groupValue`.
- Always use strongly typed Enums (`Radio<MyEnum>`) instead of raw strings or integers.
- Prefer [RadioListTile](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/13_Radio.md) in form layouts to ensure large, accessible touch targets across the entire row width.
