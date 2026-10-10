# DatePicker in Flutter (CalendarDatePicker & CupertinoDatePicker)

## 📖 Introduction
Entering dates manually into text fields is notoriously error-prone. Users mix up date formats (`DD/MM/YYYY` vs `MM/DD/YYYY`), enter non-existent dates like February 31st, or choose past dates for future flight bookings.

To solve this, Flutter provides dedicated date-picking widgets that present a calendar grid or an iOS-style spinning wheel. While `showDatePicker()` displays a date picker inside a popup dialog (which we explore in chapter 19), Flutter also offers inline date picker widgets that can be embedded directly into your screen layout:
- [CalendarDatePicker](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/17_DatePicker.md): An inline Material 3 calendar grid widget.
- [CupertinoDatePicker](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/17_DatePicker.md): An iOS-style mechanical wheel picker.

---

## 🤔 Why Do We Need It?
1. **Guaranteed Valid Dates**: Users physically cannot pick February 30th or April 31st; the calendar mathematics eliminate invalid calendar days.
2. **Strict Range Boundaries**: With `firstDate` and `lastDate`, you can restrict selection to valid future windows (e.g. reserving a hotel within the next 90 days) or valid past windows (e.g. birthdates for users over 18).
3. **Business Rule Enforcement**: The `selectableDayPredicate` callback allows disabling specific days—such as graying out weekends, national holidays, or fully-booked appointment slots.
4. **Embedded Inline UX**: For tablet dashboards, kiosk apps, or scheduling screens, an inline calendar is much faster to use than repeatedly opening and closing modal dialogs.

---

## 🌍 Real-Life Analogy
Think of [CalendarDatePicker](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/17_DatePicker.md) as a **Desktop Wall Calendar with Red Sticky Notes**:

```
+-------------------------------------------------------------+
|                     DESKTOP WALL CALENDAR                   |
|                                                             |
|   <                   OCTOBER 2026                      >   |
|   Su   Mo   Tu   We   Th   Fr   Sa                          |
|                  1    2    3   [ 4] (Today)                 |
|    5    6    7    8    9   10   11                          |
|   12   13   14   15   16  (17)  18  <- (Selected: 17th)     |
|   19   20   21   22   23   24   25                          |
|   26   27   28   29   30   31                               |
|                                                             |
|   Sundays are grayed out because the clinic is closed.      |
|   You place a sticker directly on the 17th to book it!      |
+-------------------------------------------------------------+
```

---

## 🧠 Internal Working
Understanding how Flutter constructs calendar layouts:

```
               CalendarDatePicker Widget
                           │
             ┌─────────────┴─────────────┐
             ▼                           ▼
    _CalendarHeader             PageView of Months
  (Month/Year Title +         (MonthView with 7x6 Grid)
   Left/Right Nav Arrows)                │
                                         ▼
                            DayPicker (Custom Layout)
                                         │
                             ┌───────────┴───────────┐
                             ▼                       ▼
                     Valid Date Cell         Disabled Date Cell
                  (selectablePredicate)   (Weekends / Out-of-bounds)
                             │                       │
                             ▼                       ▼
                     Selectable Tap         Muted Gray / Unclickable
                             │
                             ▼
                 onDateChanged(DateTime)
```

1. **Grid Math**: Calculates the number of days in the current month, determines which day of the week the 1st lands on, and offsets leading blanks.
2. **Predicate Evaluation**: For every cell in the $7 \times 6$ grid, Flutter passes that date into `selectableDayPredicate(date)`. If it returns `false`, the day is painted in a muted color and touch events are discarded.
3. **Boundary Enforcements**: Any date before `firstDate` or after `lastDate` is automatically disabled.

---

## ⚙ Syntax

### CalendarDatePicker (Inline Material Grid)
```dart
CalendarDatePicker(
  initialDate: _selectedDate,
  firstDate: DateTime(2020),
  lastDate: DateTime(2030),
  onDateChanged: (DateTime newDate) {
    setState(() {
      _selectedDate = newDate;
    });
  },
  // Optional: Disable weekends
  selectableDayPredicate: (DateTime day) {
    return day.weekday != DateTime.saturday && day.weekday != DateTime.sunday;
  },
)
```

### CupertinoDatePicker (iOS Rotating Wheel)
```dart
SizedBox(
  height: 200,
  child: CupertinoDatePicker(
    mode: CupertinoDatePickerMode.date,
    initialDateTime: _selectedDate,
    minimumDate: DateTime(2020),
    maximumDate: DateTime(2030),
    onDateTimeChanged: (DateTime newDate) {
      setState(() {
        _selectedDate = newDate;
      });
    },
  ),
)
```

---

## 📋 Important Properties

| Property | Type | Description | Default |
| :--- | :--- | :--- | :--- |
| `initialDate` | `DateTime` | The date currently selected when first displayed. Must fall between `firstDate` and `lastDate`. | *Required* |
| `firstDate` | `DateTime` | The earliest allowable selectable date. | *Required* |
| `lastDate` | `DateTime` | The latest allowable selectable date. | *Required* |
| `onDateChanged` | `ValueChanged<DateTime>` | Called immediately whenever a user taps a valid day cell. | *Required* |
| `selectableDayPredicate` | `SelectableDayPredicate?` | A boolean function `bool Function(DateTime day)` used to enable or disable specific days. | `null` |
| `initialCalendarMode` | `DatePickerMode` | Controls whether the picker opens showing the `day` grid or the `year` selection list. | `DatePickerMode.day` |

---

## 💻 Complete Runnable Example
The following production-ready application demonstrates:
1. An embedded, inline [CalendarDatePicker](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/17_DatePicker.md) inside an appointment booking card.
2. Disabling past dates and weekends using `selectableDayPredicate`.
3. Displaying a clean booking confirmation summary with formatted date output.

```dart
import 'package:flutter/cupertino.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(const DatePickerMasterclassApp());
}

class DatePickerMasterclassApp extends StatelessWidget {
  const DatePickerMasterclassApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'DatePicker Masterclass',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.teal),
        useMaterial3: true,
      ),
      home: const ClinicBookingScreen(),
    );
  }
}

class ClinicBookingScreen extends StatefulWidget {
  const ClinicBookingScreen({super.key});

  @override
  State<ClinicBookingScreen> createState() => _ClinicBookingScreenState();
}

class _ClinicBookingScreenState extends State<ClinicBookingScreen> {
  // Initial date set to tomorrow at 00:00:00
  late DateTime _selectedDate;
  late final DateTime _today;
  late final DateTime _maxBookingDate;

  @override
  void initState() {
    super.initState();
    final now = DateTime.now();
    _today = DateTime(now.year, now.month, now.day);
    // Default to tomorrow if today is Sunday, else tomorrow
    _selectedDate = _today.add(const Duration(days: 1));
    if (_selectedDate.weekday == DateTime.sunday) {
      _selectedDate = _selectedDate.add(const Duration(days: 1));
    }
    // Booking window: 60 days ahead
    _maxBookingDate = _today.add(const Duration(days: 60));
  }

  // Clinic rule: Closed on Sundays; Saturdays open half-day
  bool _isDayAvailable(DateTime day) {
    if (day.weekday == DateTime.sunday) {
      return false; // Closed on Sundays
    }
    return true;
  }

  String _formatDate(DateTime dt) {
    const weekdays = ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun'];
    const months = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'];
    return '${weekdays[dt.weekday - 1]}, ${months[dt.month - 1]} ${dt.day}, ${dt.year}';
  }

  void _showCupertinoDatePicker() {
    showModalBottomSheet(
      context: context,
      builder: (ctx) => Container(
        height: 280,
        color: Colors.white,
        child: Column(
          children: [
            Container(
              color: Colors.grey.shade100,
              padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
              child: Row(
                mainAxisAlignment: MainAxisAlignment.spaceBetween,
                children: [
                  const Text('Select Date', style: TextStyle(fontWeight: FontWeight.bold)),
                  TextButton(
                    onPressed: () => Navigator.pop(ctx),
                    child: const Text('Done'),
                  ),
                ],
              ),
            ),
            Expanded(
              child: CupertinoDatePicker(
                mode: CupertinoDatePickerMode.date,
                initialDateTime: _selectedDate,
                minimumDate: _today,
                maximumDate: _maxBookingDate,
                onDateTimeChanged: (DateTime newDate) {
                  setState(() {
                    _selectedDate = newDate;
                  });
                },
              ),
            ),
          ],
        ),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Appointment Scheduler'),
        centerTitle: true,
      ),
      body: SafeArea(
        child: SingleChildScrollView(
          padding: const EdgeInsets.all(20.0),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: [
              const Text(
                'Schedule a Doctor Consultation',
                style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
              ),
              const SizedBox(height: 6),
              const Text(
                'Sundays are unavailable. Appointments available up to 60 days ahead.',
                style: TextStyle(color: Colors.grey),
              ),
              const SizedBox(height: 20),

              // Embedded Material 3 Calendar Date Picker
              Card(
                elevation: 2,
                shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(16)),
                child: Padding(
                  padding: const EdgeInsets.all(8.0),
                  child: CalendarDatePicker(
                    initialDate: _selectedDate,
                    firstDate: _today,
                    lastDate: _maxBookingDate,
                    selectableDayPredicate: _isDayAvailable,
                    onDateChanged: (DateTime newDate) {
                      setState(() {
                        _selectedDate = newDate;
                      });
                    },
                  ),
                ),
              ),
              const SizedBox(height: 16),

              // Alternative: iOS Cupertino Wheel Picker Button
              OutlinedButton.icon(
                onPressed: _showCupertinoDatePicker,
                icon: const Icon(Icons.phone_iphone),
                label: const Text('Or Select with iOS Cupertino Wheel'),
              ),
              const SizedBox(height: 24),

              // Selected Appointment Confirmation Banner
              Container(
                padding: const EdgeInsets.all(16),
                decoration: BoxDecoration(
                  color: Colors.teal.shade50,
                  borderRadius: BorderRadius.circular(12),
                  border: Border.all(color: Colors.teal.shade300),
                ),
                child: Row(
                  children: [
                    const Icon(Icons.event_available, color: Colors.teal, size: 36),
                    const SizedBox(width: 14),
                    Expanded(
                      child: Column(
                        crossAxisAlignment: CrossAxisAlignment.start,
                        children: [
                          const Text(
                            'Selected Appointment Date:',
                            style: TextStyle(fontSize: 12, color: Colors.teal, fontWeight: FontWeight.w600),
                          ),
                          Text(
                            _formatDate(_selectedDate),
                            style: const TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
                          ),
                        ],
                      ),
                    ),
                  ],
                ),
              ),
              const SizedBox(height: 20),

              FilledButton(
                onPressed: () {
                  ScaffoldMessenger.of(context).showSnackBar(
                    SnackBar(
                      content: Text('Consultation booked for ${_formatDate(_selectedDate)}!'),
                      backgroundColor: Colors.teal,
                    ),
                  );
                },
                style: FilledButton.styleFrom(padding: const EdgeInsets.symmetric(vertical: 16)),
                child: const Text('Confirm Appointment', style: TextStyle(fontSize: 16)),
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
- `firstDate: _today, lastDate: _maxBookingDate`:
  - **What it does**: Enforces the valid booking window.
  - **Why it exists**: Prevents users from selecting dates in the past or unreasonably far in the future.
  - **Internal constraint**: If `initialDate` is before `firstDate` or after `lastDate`, Flutter throws an immediate assertion exception.
- `selectableDayPredicate: _isDayAvailable`:
  - **What it does**: Executes for every day displayed in the month view.
  - **Why it exists**: If `day.weekday == DateTime.sunday`, it returns `false`, causing Flutter to render Sunday dates in disabled gray and ignore user taps.
- `_today = DateTime(now.year, now.month, now.day);`:
  - **What it does**: Strips hours, minutes, seconds, and milliseconds from the current date.
  - **Why it exists**: If you compare `DateTime.now()` directly against a day cell that represents midnight `00:00:00`, `firstDate` may be considered *after* today's date, causing bugs with selecting today!
- `CupertinoDatePicker(mode: CupertinoDatePickerMode.date, ...)`:
  - **What it does**: Renders three mechanical rotating drum wheels (Month, Day, Year).
  - **Why it exists**: Provides the native iOS date picker experience.

---

## 📱 ASCII Phone Screen Preview
```
+------------------------------------------+
|          Appointment Scheduler           |
+------------------------------------------+
| Schedule a Doctor Consultation           |
| Sundays are unavailable.                 |
|                                          |
| +--------------------------------------+ |
| | <            OCTOBER 2026          > | |
| | Su   Mo   Tu   We   Th   Fr   Sa     | |
| |       -    -    -    1    2    3     | |
| | [4]   5    6    7    8    9   10     | |
| | [11] 12   13   14   15   16  (17)    | |
| | [18] 19   20   21   22   23   24     | |
| | [25] 26   27   28   29   30   31     | |
| | (Sundays [X] are disabled / grayed)  | |
| +--------------------------------------+ |
|                                          |
| [ 📱 Or Select with iOS Wheel ]          |
|                                          |
| +--------------------------------------+ |
| | [📅] Selected Appointment Date:      | |
| |      Sat, Oct 17, 2026               | |
| +--------------------------------------+ |
|                                          |
| +--------------------------------------+ |
| |         Confirm Appointment          | |
| +--------------------------------------+ |
+------------------------------------------+
```

---

## 🌳 Widget Tree
```
MaterialApp
 └── ClinicBookingScreen (StatefulWidget)
      └── Scaffold
           ├── AppBar
           └── SafeArea
                └── SingleChildScrollView
                     └── Column
                          ├── Text ("Schedule Consultation")
                          ├── Card
                          │    └── CalendarDatePicker (Material 3 embedded calendar)
                          ├── OutlinedButton ("iOS Cupertino Wheel")
                          ├── Container (Appointment confirmation card)
                          └── FilledButton ("Confirm Appointment")
```

---

## ⚙ Flutter Execution Flow
1. **Sanitization**: `initState()` strips hours/minutes, calculating normalized dates `_today` and `_maxBookingDate`.
2. **Month Render**: `CalendarDatePicker` renders the current month.
3. **Predicate Filtering**: It calls `selectableDayPredicate` for every date cell. Sundays evaluate to `false` and receive disabled styling.
4. **Day Click**: User taps Saturday, October 17th.
5. **Callback**: `onDateChanged(DateTime(2026, 10, 17))` fires.
6. **State Update**: `setState()` assigns `_selectedDate = newDate`.
7. **Repaint**: The 17th is encircled with the active teal theme color, and the bottom confirmation banner updates its formatted text.

---

## ⚠ Common Beginner Mistakes

### 1. `initialDate` Out of Bounds Assertion Crash
```dart
// ❌ CRASH: initialDate is BEFORE firstDate!
CalendarDatePicker(
  initialDate: DateTime(2025, 1, 1),
  firstDate: DateTime(2026, 1, 1), // firstDate is later than initialDate!
  lastDate: DateTime(2030, 1, 1),
  onDateChanged: (d) {},
)
// Result: Crash! Assertion failed: "initialDate.isAfter(firstDate) || isAtSameMomentAs"

// ✅ GOOD: Ensure firstDate <= initialDate <= lastDate
```

### 2. Timezone and Hour-Offset Comparison Bugs
```dart
// ❌ BUG: Using raw DateTime.now() with hours and seconds
final firstDate = DateTime.now(); // e.g. 14:35:20
// Today's calendar cell is at 00:00:00!
// 00:00:00 is BEFORE 14:35:20, so today is considered disabled!

// ✅ GOOD: Strip time components to midnight
final today = DateTime(now.year, now.month, now.day);
```

### 3. Missing Height Constraint for `CupertinoDatePicker`
```dart
// ❌ LAYOUT ERROR: CupertinoDatePicker has unbounded height
Column(
  children: [
    CupertinoDatePicker(...), // Crashes with vertical unbounded viewport error!
  ],
)

// ✅ GOOD: Wrap CupertinoDatePicker in a SizedBox with explicit height
SizedBox(
  height: 200,
  child: CupertinoDatePicker(...),
)
```

---

## 🚀 Professional Usage
- **Healthcare & Telemedicine (e.g., Teladoc, Zocdoc)**: In-app scheduling grids displaying doctor availability by blocking unbookable days.
- **Hotel & Vacation Rentals (e.g., Airbnb, Booking.com)**: Displaying unavailable booked dates in gray and available dates with price tags underneath.
- **Flight Booking (e.g., Skyscanner, United)**: Selecting outbound and return dates with minimum stay intervals.

---

## 💡 Best Practices
1. **Always normalize `DateTime` instances to midnight `(year, month, day)`**: Eliminates subtle bugs caused by hours, minutes, and time zones.
2. **Use `selectableDayPredicate` for business rules**: Restrict selection to valid operating days rather than validating after the user makes an invalid pick.
3. **Use the `intl` package for internationalized date formatting**: Handles multi-language month names and localized date orders (`DateFormat.yMMMMd()`).
4. **Choose between Inline vs Dialog based on context**: Use inline `CalendarDatePicker` when the calendar is the primary focus of the screen; use `showDatePicker()` when date entry is just one field among many in a form.

---

## 🧪 Try It Yourself
1. Change `initialCalendarMode: DatePickerMode.year` and observe how the picker opens displaying the year selection grid first.
2. Add a rule to `selectableDayPredicate` that disables all dates falling on national holidays.
3. Wrap `CalendarDatePicker` in a `Theme` widget with a custom `ColorScheme` to change the calendar selection circle color.

---

## 🎯 Challenge
Create a dynamic "Leave Request / Vacation" booking widget:
- Embed a `CalendarDatePicker`.
- When the user selects a start date, calculate the total working days (excluding Saturdays and Sundays) for a 5-day leave request.
- Display a summary: *"Leave: Monday, Oct 12 to Friday, Oct 16 (Total: 5 working days)"*.

---

## ❓ Practice Questions

### Q1. What is the fundamental difference between `CalendarDatePicker` and `showDatePicker()`?
- A) `CalendarDatePicker` is an inline widget embedded directly in a screen's layout; `showDatePicker()` displays a calendar in a modal popup dialog
- B) `CalendarDatePicker` only works on Android
- C) `CalendarDatePicker` does not support `firstDate`
- D) `showDatePicker()` cannot pick years
*Answer: A* — `CalendarDatePicker` embeds into the widget tree; `showDatePicker()` opens a modal dialog route.

### Q2. What error occurs if `initialDate` is set to yesterday while `firstDate` is set to today?
- A) Flutter resets `initialDate` to today silently
- B) Flutter crashes with an assertion failure stating `initialDate` must fall within bounds
- C) The date picker displays negative numbers
- D) The calendar renders in black and white
*Answer: B* — Boundary checks are enforced via strict framework assertions.

### Q3. How can you disable all Sundays in a `CalendarDatePicker`?
- A) Set `sundays: false`
- B) Pass a function to `selectableDayPredicate` that returns `day.weekday != DateTime.sunday`
- C) Pass `disabledDays: [7]`
- D) It cannot be done in Flutter
*Answer: B* — `selectableDayPredicate` evaluates each day and disables any date where it returns `false`.

### Q4. Why should you strip the hour and minute components from `firstDate` when using `DateTime.now()`?
- A) To save memory
- B) Because calendar cells represent midnight `00:00:00`. If `firstDate` contains the current time (e.g. 3 PM), today's midnight is strictly before `firstDate` and becomes disabled
- C) Dart does not support hours
- D) Timezones will crash the app
*Answer: B* — Normalizing to midnight prevents today's date from being erroneously marked as in the past.

### Q5. Why does `CupertinoDatePicker` require an explicit height constraint (such as a `SizedBox`)?
- A) It has unbounded intrinsic height and will throw a layout exception inside unbounded parents like `Column`
- B) To change its font size
- C) To make it scroll faster
- D) It only works in full screen
*Answer: A* — Cupertino spinning drum wheels require an explicit height to calculate wheel perspective.

### Q6. Which property makes `CalendarDatePicker` open showing the year selector list first?
*Answer*: `initialCalendarMode: DatePickerMode.year`.

### Q7. What integer does `DateTime.monday` represent in Dart?
*Answer*: `1` (`DateTime.sunday` is `7`).

### Q8. Output Prediction: What will `_isDayAvailable(DateTime(2026, 10, 4))` return if October 4, 2026 is a Sunday and Sundays return `false`?
*Answer*: `false`.

### Q9. Which package is the industry standard for parsing and formatting `DateTime` objects into localized strings in Flutter?
*Answer*: The `intl` package (`DateFormat`).

### Q10. Write a minimal code snippet defining a normalized `DateTime` representing today at midnight.
*Answer*:
```dart
final now = DateTime.now();
final today = DateTime(now.year, now.month, now.day);
```

---

## 📝 Summary
- [CalendarDatePicker](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/17_DatePicker.md) embeds an interactive calendar directly into your screen layout.
- [CupertinoDatePicker](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/17_DatePicker.md) provides iOS-style spinning drum wheels.
- Always guarantee that `firstDate <= initialDate <= lastDate`.
- Strip time components `(year, month, day)` to avoid hour-offset comparison bugs.
- Use `selectableDayPredicate` to enforce business rules like disabling weekends or holidays.
