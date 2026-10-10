# showDatePicker in Flutter

## 📖 Introduction
In standard mobile forms—such as a 12-field user registration or flight reservation screen—embedding an entire month-view calendar widget directly into the scrollable body consumes immense screen real estate and pushes action buttons off the screen.

The standard mobile design pattern is to display a clean, compact text field or button (e.g. `Select Departure Date: Oct 24, 2026`). Tapping that field triggers [showDatePicker()](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/19_ShowDatePicker.md), which suspends execution, pushes a modal dialog over the current screen, allows the user to tap their desired date, and returns the selected [DateTime](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/17_DatePicker.md) (or `null` if canceled) through a Dart [Future](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/19_ShowDatePicker.md). Flutter also provides the companion [showDateRangePicker()](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/19_ShowDatePicker.md) for choosing travel check-in and check-out periods in a single modal flow.

---

## 🤔 Why Do We Need It?
1. **Space Preservation**: The screen remains uncluttered; the calendar appears only on demand.
2. **Asynchronous Resolution**: By returning a `Future<DateTime?>`, handling user selection integrates cleanly with standard `async/await` code.
3. **Cancellation Handling**: If the user taps "Cancel" or touches outside the dialog barrier, `showDatePicker()` yields `null`, allowing safe cancellation handling.
4. **Range Support**: `showDateRangePicker()` renders a full-screen Material modal with automatic multi-month continuous scrolling and start-to-end highlighting.

---

## 🌍 Real-Life Analogy
Think of [showDatePicker()](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/19_ShowDatePicker.md) as a **Waiter Handing You a Dessert Menu**:

```
+-------------------------------------------------------------+
|                     DINING AT A RESTAURANT                  |
|                                                             |
|   Table: You are eating dinner (Current Screen).            |
|                                                             |
|   You tap "Order Dessert" (Trigger showDatePicker()):       |
|   -> The waiter brings out the full Dessert Tray (Dialog).  |
|                                                             |
|   Outcome A:                                                |
|   You pick the "Chocolate Soufflé" (Tap Oct 24 & Confirm)   |
|   -> The tray is taken away; your order is placed.          |
|                                                             |
|   Outcome B:                                                |
|   You say "No thanks, I'm full" (Tap Cancel / Barrier)      |
|   -> The tray is taken away; nothing is ordered (null).     |
+-------------------------------------------------------------+
```

---

## 🧠 Internal Working
When you call `showDatePicker()`, Flutter's routing system activates:

```
                      User Taps "Select Date"
                                │
                                ▼
        final DateTime? picked = await showDatePicker(...)
                                │
                                ▼
           Navigator.push(DialogRoute<DateTime>)
                                │
                                ▼
             Modal Barrier Blocks Background Touches
                                │
                                ▼
         User Interacts With Calendar Dialog
         ┌──────────────────────┴──────────────────────┐
         ▼                                             ▼
    Taps "CANCEL"                                Taps "OK"
         │                                             │
         ▼                                             ▼
  Navigator.pop(null)                     Navigator.pop(selectedDate)
         │                                             │
         └──────────────────────┬──────────────────────┘
                                │
                                ▼
                    Future Completes In Code
                    picked = selectedDate OR null
                                │
                                ▼
                Check if (context.mounted && picked != null)
```

1. **Dialog Route**: `showDatePicker` invokes `showDialog()` with a specialized `_DatePickerDialog` widget.
2. **Context Resolution**: The dialog inherits themes, text scales, and localizations from the provided `BuildContext`.
3. **Custom Builder**: If you need custom styling (e.g. dark calendar popup on a light screen), you can pass a `builder: (context, child) => Theme(...)`.

---

## ⚙ Syntax

### Single Date Picker
```dart
final DateTime? pickedDate = await showDatePicker(
  context: context,
  initialDate: DateTime.now(),
  firstDate: DateTime(2020),
  lastDate: DateTime(2030),
  helpText: 'Select Departure Date',
  cancelText: 'Back',
  confirmText: 'Confirm Date',
);

if (pickedDate != null) {
  // User confirmed a date
}
```

### Date Range Picker
```dart
final DateTimeRange? pickedRange = await showDateRangePicker(
  context: context,
  firstDate: DateTime.now(),
  lastDate: DateTime.now().add(const Duration(days: 365)),
  helpText: 'Select Reservation Dates',
);

if (pickedRange != null) {
  print('Start: ${pickedRange.start}, End: ${pickedRange.end}');
}
```

---

## 📋 Important Parameters

| Parameter | Type | Description | Default |
| :--- | :--- | :--- | :--- |
| `context` | `BuildContext` | The build context used to resolve themes and navigation. | *Required* |
| `initialDate` | `DateTime` | The date highlighted when dialog opens. Must satisfy `firstDate <= initialDate <= lastDate`. | *Required* |
| `firstDate` | `DateTime` | The earliest selectable date boundary. | *Required* |
| `lastDate` | `DateTime` | The latest selectable date boundary. | *Required* |
| `builder` | `TransitionBuilder?` | Wrap the dialog in a custom `Theme` or `Localizations` widget. | `null` |
| `helpText` | `String?` | Header title displayed at the top of the dialog. | `'Select date'` |
| `cancelText` | `String?` | Label for the dismiss button. | `'Cancel'` |
| `confirmText` | `String?` | Label for the approval button. | `'OK'` |
| `initialDatePickerMode` | `DatePickerMode` | Opens directly to `DatePickerMode.day` or `DatePickerMode.year`. | `DatePickerMode.day` |
| `selectableDayPredicate` | `SelectableDayPredicate?` | Function to disable specific dates (e.g. holidays or weekends). | `null` |

---

## 💻 Complete Runnable Example
The following production-grade application demonstrates:
1. A single date picker with custom themed dialog builder.
2. A full-screen [showDateRangePicker()](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/19_ShowDatePicker.md) for hotel check-in and check-out reservation.
3. Safe asynchronous navigation checks (`if (!context.mounted) return;`).

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ShowDatePickerMasterclassApp());
}

class ShowDatePickerMasterclassApp extends StatelessWidget {
  const ShowDatePickerMasterclassApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'showDatePicker Guide',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const HotelReservationScreen(),
    );
  }
}

class HotelReservationScreen extends StatefulWidget {
  const HotelReservationScreen({super.key});

  @override
  State<HotelReservationScreen> createState() =>
      _HotelReservationScreenState();
}

class _HotelReservationScreenState extends State<HotelReservationScreen> {
  DateTime? _flightDate;
  DateTimeRange? _hotelStayRange;

  String _formatDate(DateTime dt) {
    const months = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'];
    return '${months[dt.month - 1]} ${dt.day}, ${dt.year}';
  }

  // 1. Single Date Picker with Custom Styling
  Future<void> _pickFlightDate() async {
    final now = DateTime.now();
    final today = DateTime(now.year, now.month, now.day);

    final DateTime? selected = await showDatePicker(
      context: context,
      initialDate: _flightDate ?? today.add(const Duration(days: 7)),
      firstDate: today,
      lastDate: today.add(const Duration(days: 365)),
      helpText: 'Select Flight Departure Date',
      confirmText: 'Choose Date',
      cancelText: 'Cancel',
      builder: (BuildContext context, Widget? child) {
        return Theme(
          data: Theme.of(context).copyWith(
            colorScheme: ColorScheme.fromSeed(
              seedColor: Colors.deepPurple,
              primary: Colors.deepPurple,
              onPrimary: Colors.white,
              surface: Colors.white,
            ),
          ),
          child: child!,
        );
      },
    );

    // CRITICAL: Ensure widget is still mounted across async gap
    if (!mounted) return;

    if (selected != null) {
      setState(() {
        _flightDate = selected;
      });
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          content: Text('Flight scheduled for: ${_formatDate(selected)}'),
          backgroundColor: Colors.deepPurple,
        ),
      );
    }
  }

  // 2. Hotel Check-In / Check-Out Range Picker
  Future<void> _pickHotelRange() async {
    final now = DateTime.now();
    final today = DateTime(now.year, now.month, now.day);

    final DateTimeRange? range = await showDateRangePicker(
      context: context,
      firstDate: today,
      lastDate: today.add(const Duration(days: 365)),
      initialDateRange: _hotelStayRange ??
          DateTimeRange(
            start: today.add(const Duration(days: 7)),
            end: today.add(const Duration(days: 12)),
          ),
      helpText: 'Select Check-In & Check-Out Dates',
      saveText: 'Confirm Stay',
    );

    if (!mounted) return;

    if (range != null) {
      setState(() {
        _hotelStayRange = range;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Travel Booking Portal'),
        centerTitle: true,
      ),
      body: SafeArea(
        child: ListView(
          padding: const EdgeInsets.all(20.0),
          children: [
            const Text(
              'Trip Itinerary Planner',
              style: TextStyle(fontSize: 22, fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 6),
            const Text(
              'Pick your flight and hotel dates using modal pickers.',
              style: TextStyle(color: Colors.grey),
            ),
            const SizedBox(height: 24),

            // CARD 1: Flight Departure Date (Single Picker)
            Card(
              elevation: 0,
              shape: RoundedRectangleBorder(
                borderRadius: BorderRadius.circular(16),
                side: BorderSide(color: Colors.grey.shade300),
              ),
              child: ListTile(
                contentPadding: const EdgeInsets.all(16),
                leading: const CircleAvatar(
                  backgroundColor: Colors.deepPurple,
                  child: Icon(Icons.flight_takeoff, color: Colors.white),
                ),
                title: const Text('Flight Departure Date', style: TextStyle(fontWeight: FontWeight.bold)),
                subtitle: Text(
                  _flightDate != null
                      ? _formatDate(_flightDate!)
                      : 'No date selected (Tap to pick)',
                  style: TextStyle(
                    color: _flightDate != null ? Colors.deepPurple : Colors.grey,
                    fontWeight: _flightDate != null ? FontWeight.w600 : FontWeight.normal,
                  ),
                ),
                trailing: const Icon(Icons.calendar_month, color: Colors.deepPurple),
                onTap: _pickFlightDate,
              ),
            ),
            const SizedBox(height: 16),

            // CARD 2: Hotel Reservation (Date Range Picker)
            Card(
              elevation: 0,
              shape: RoundedRectangleBorder(
                borderRadius: BorderRadius.circular(16),
                side: BorderSide(color: Colors.grey.shade300),
              ),
              child: ListTile(
                contentPadding: const EdgeInsets.all(16),
                leading: const CircleAvatar(
                  backgroundColor: Colors.teal,
                  child: Icon(Icons.hotel, color: Colors.white),
                ),
                title: const Text('Hotel Stay Period', style: TextStyle(fontWeight: FontWeight.bold)),
                subtitle: Text(
                  _hotelStayRange != null
                      ? '${_formatDate(_hotelStayRange!.start)}  →  ${_formatDate(_hotelStayRange!.end)} '
                          '(${_hotelStayRange!.duration.inDays} Nights)'
                      : 'Select check-in & check-out dates',
                  style: TextStyle(
                    color: _hotelStayRange != null ? Colors.teal.shade800 : Colors.grey,
                    fontWeight: _hotelStayRange != null ? FontWeight.w600 : FontWeight.normal,
                  ),
                ),
                trailing: const Icon(Icons.date_range, color: Colors.teal),
                onTap: _pickHotelRange,
              ),
            ),
            const SizedBox(height: 32),

            // Submit Summary Button
            FilledButton.icon(
              onPressed: (_flightDate != null && _hotelStayRange != null)
                  ? () {
                      showDialog(
                        context: context,
                        builder: (ctx) => AlertDialog(
                          icon: const Icon(Icons.check_circle, color: Colors.green, size: 48),
                          title: const Text('Itinerary Confirmed!'),
                          content: Text(
                            'Flight Departure: ${_formatDate(_flightDate!)}\n\n'
                            'Hotel Check-In: ${_formatDate(_hotelStayRange!.start)}\n'
                            'Hotel Check-Out: ${_formatDate(_hotelStayRange!.end)}\n'
                            'Duration: ${_hotelStayRange!.duration.inDays} Nights',
                          ),
                          actions: [
                            TextButton(
                              onPressed: () => Navigator.pop(ctx),
                              child: const Text('Great'),
                            ),
                          ],
                        ),
                      );
                    }
                  : null,
              icon: const Icon(Icons.confirmation_num_outlined),
              label: const Text('Confirm Travel Itinerary', style: TextStyle(fontSize: 16)),
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
- `final DateTime? selected = await showDatePicker(...)`:
  - **What it does**: Pushes the modal calendar dialog and suspends the function until the user confirms or dismisses the dialog.
  - **Return type**: Returns a nullable `DateTime?`. If the user taps outside the dialog or hits "Cancel", it resolves to `null`.
- `if (!mounted) return;`:
  - **What it does**: Verifies the `State` object is still in the widget tree before calling `setState()` or displaying a SnackBar.
  - **Why it exists**: If the user leaves the screen while the picker dialog is open, continuing without this check causes a `setState() called after dispose()` memory leak exception.
- `builder: (BuildContext context, Widget? child) { return Theme(...); }`:
  - **What it does**: Wraps the internal dialog tree with custom theme overrides.
  - **Why it exists**: Lets you customize dialog surface colors, typography, or calendar accent colors without mutating global app-wide themes.
- `showDateRangePicker(...)`:
  - **What it does**: Launches a full-screen Material modal configured specifically for picking start and end dates with continuous scrolling.
  - **Output**: Returns a `DateTimeRange` object containing `.start`, `.end`, and `.duration.inDays`.

---

## 📱 ASCII Phone Screen Preview
```
+------------------------------------------+
|          Travel Booking Portal           |
+------------------------------------------+
| Trip Itinerary Planner                   |
| Pick your flight and hotel dates...      |
|                                          |
| +--------------------------------------+ |
| | (✈) Flight Departure Date        [📅]| |
| |     Oct 24, 2026                     | |
| +--------------------------------------+ |
|                                          |
| +--------------------------------------+ |
| | (🏨) Hotel Stay Period           [📅]| |
| |     Oct 24, 2026 -> Oct 29, 2026     | |
| |     (5 Nights)                       | |
| +--------------------------------------+ |
|                                          |
|      ┌─────────────────────────────┐     |
|      │ Select Departure Date       │     |
|      │ Sat, Oct 24                 │     |
|      │-----------------------------│     |
|      │ <        October 2026     > │     |
|      │ Su Mo Tu We Th Fr Sa        │     |
|      │  -  -  -  1  2  3  4        │     |
|      │ 18 19 20 21 22 23 (24)      │     |
|      │                             │     |
|      │      [CANCEL]  [CHOOSE DATE]│     |
|      └─────────────────────────────┘     |
|                                          |
| +--------------------------------------+ |
| |   [🎫] Confirm Travel Itinerary      | |
| +--------------------------------------+ |
+------------------------------------------+
```

---

## 🌳 Widget Tree
```
MaterialApp
 └── HotelReservationScreen (StatefulWidget)
      └── Scaffold
           ├── AppBar
           └── SafeArea
                └── ListView
                     ├── Text ("Trip Itinerary Planner")
                     ├── Card -> ListTile (Flight Date, onTap: _pickFlightDate)
                     ├── Card -> ListTile (Hotel Range, onTap: _pickHotelRange)
                     └── FilledButton ("Confirm Travel Itinerary")

[ Overlay Layer during Dialog ]:
 ModalBarrier
  └── DialogRoute<DateTime>
       └── _DatePickerDialog (Pushed onto Navigator)
```

---

## ⚙ Flutter Execution Flow
1. **User Tap**: User taps the "Flight Departure Date" list tile.
2. **Function Execution**: `_pickFlightDate()` calls `showDatePicker()`.
3. **Route Push**: Flutter pushes a `DialogRoute` with a modal backdrop barrier.
4. **User Action**:
   - **Case A (Confirmation)**: User clicks October 24th and taps "Choose Date". `Navigator.pop(selectedDate)` is called.
   - **Case B (Cancellation)**: User taps "Cancel" or taps the barrier. `Navigator.pop(null)` is called.
5. **Resume**: `await` unblocks, returning `selectedDate` or `null`.
6. **Guard**: `if (!mounted) return;` ensures safe execution.
7. **UI Update**: `setState()` saves the date and updates the card subtitle.

---

## ⚠ Common Beginner Mistakes

### 1. Forgetting to Handle Null on Cancellation
```dart
// ❌ CRASH: Assuming user ALWAYS selects a date!
final picked = await showDatePicker(...);
print(picked.year); // Null check operator or NoSuchMethodError if user clicked Cancel!

// ✅ GOOD: Always null-check the return value
if (picked != null) {
  print(picked.year);
}
```

### 2. Calling `setState` Across Async Gap Without `mounted` Check
```dart
// ❌ MEMORY LEAK WARNING:
final picked = await showDatePicker(...);
setState(() { // Linter warning: "Don't use 'BuildContext' or 'setState' across async gaps"
  _date = picked;
});

// ✅ GOOD: Guard with if (!mounted) return;
final picked = await showDatePicker(...);
if (!mounted) return;
if (picked != null) {
  setState(() => _date = picked);
}
```

### 3. Setting `initialDate` Before `firstDate`
```dart
// ❌ RUNTIME ASSERTION: initialDate is before firstDate!
final picked = await showDatePicker(
  context: context,
  initialDate: DateTime(2024, 1, 1),
  firstDate: DateTime(2025, 1, 1), // Invalid!
  lastDate: DateTime(2030, 1, 1),
);
```

---

## 🚀 Professional Usage
- **Airlines & Flights (e.g., Delta, Ryanair)**: One-way bookings trigger `showDatePicker()`; round-trip bookings trigger `showDateRangePicker()`.
- **E-Commerce Returns (e.g., Amazon)**: Scheduling pickup return windows for package couriers.
- **Human Resources & Payroll (e.g., Workday, Gusto)**: Filtering paystubs and timesheet records across custom quarterly date ranges.

---

## 💡 Best Practices
1. **Always check `if (!mounted) return;`**: Critical in asynchronous UI workflows to protect against screen-dismissal crashes.
2. **Use `showDateRangePicker()` for paired dates**: Never make users open two separate single date pickers for check-in and check-out; range pickers validate start-before-end ordering automatically.
3. **Customize buttons with `confirmText` and `cancelText`**: Clear verbs (e.g. `"Select Date"`, `"Confirm Stay"`) improve accessibility over generic `"OK"`.
4. **Normalize dates to midnight**: Strip hours/minutes from `firstDate` and `lastDate`.

---

## 🧪 Try It Yourself
1. Change `initialDatePickerMode: DatePickerMode.year` inside `showDatePicker()` to make the dialog open in year-picking mode.
2. Add a `selectableDayPredicate` that disables all past days and upcoming Mondays.
3. Use the `builder` parameter to give the date picker a dark mode color palette.

---

## 🎯 Challenge
Build an automated "Age Gate / Birthdate Verification" dialog:
- A button: *"Verify Age (18+ Only)"*.
- The picker's `lastDate` must be calculated as `DateTime.now() - 18 Years`.
- When a valid date is picked, calculate the exact age in years and display: *"Verified: Age XX"*.

---

## ❓ Practice Questions

### Q1. What does `showDatePicker()` return when the user taps "Cancel" or outside the dialog?
- A) `DateTime.now()`
- B) `null`
- C) An error exception
- D) `DateTime(0)`
*Answer: B* — Tapping cancel resolves the `Future` with `null`.

### Q2. Why MUST you check `if (!mounted) return;` after `await showDatePicker(...)`?
- A) To clear text controllers
- B) Because the user might have navigated away from the screen while the dialog was open, and calling `setState()` on an unmounted state object causes an exception
- C) It is required by Dart syntax
- D) To dismiss the keyboard
*Answer: B* — Protects against calling `setState()` on disposed widgets across asynchronous gaps.

### Q3. Which companion function should you use when selecting both a Check-In and Check-Out date?
- A) `showMultiDatePicker()`
- B) `showDateRangePicker()`
- C) `showTwoDatesPicker()`
- D) `showCalendarList()`
*Answer: B* — `showDateRangePicker()` is specialized for selecting continuous ranges.

### Q4. What object type is returned by `showDateRangePicker()` upon confirmation?
- A) `List<DateTime>`
- B) `DateTimeRange?`
- C) `Map<String, DateTime>`
- D) `RangeValues`
*Answer: B* — Returns a nullable `DateTimeRange?` containing `.start` and `.end`.

### Q5. How can you customize the text displayed on the confirmation button in `showDatePicker()`?
- A) `okLabel: 'Done'`
- B) `confirmText: 'Choose Date'`
- C) `buttonText: 'Done'`
- D) `submitText: 'Done'`
*Answer: B* — `confirmText` customizes the label on the positive action button.

### Q6. How do you apply a custom theme specifically to the popup calendar without modifying the whole app theme?
*Answer*: Use the `builder: (BuildContext context, Widget? child)` parameter to wrap `child!` in a `Theme` widget with a custom `ThemeData`.

### Q7. What is the value of `dateTimeRange.duration.inDays` for a range from October 1 to October 6?
*Answer*: `5`.

### Q8. Output Prediction: What will this code print if the user clicks "Cancel"?
```dart
final date = await showDatePicker(...);
print(date ?? 'User Cancelled');
```
*Answer*: `'User Cancelled'`.

### Q9. Can you disable specific days of the week in `showDatePicker()`?
*Answer*: Yes, by passing a callback function to the `selectableDayPredicate` parameter.

### Q10. Write a minimal code snippet launching `showDatePicker()` for picking birthdates up to today.
*Answer*:
```dart
final birthDate = await showDatePicker(
  context: context,
  initialDate: DateTime(2000),
  firstDate: DateTime(1900),
  lastDate: DateTime.now(),
);
```

---

## 📝 Summary
- [showDatePicker()](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/19_ShowDatePicker.md) presents a modal date picker dialog, keeping the main screen clean.
- It returns a `Future<DateTime?>`; `null` indicates user cancellation.
- Always check `if (!mounted) return;` after `await` calls.
- Use `showDateRangePicker()` for booking hotel stays, vacation dates, and date intervals.
- The `builder` parameter allows injecting custom themes or localizations into the modal dialog.
