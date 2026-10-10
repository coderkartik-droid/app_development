# showTimePicker in Flutter

## 📖 Introduction
Setting a daily wake-up alarm, scheduling an appointment slot, or defining a business opening hour requires selecting a specific point on the clock. While typing digits like `0` `8` `3` `0` into a text box works, users expect interactive clock controls that make setting hours and minutes intuitive.

In Flutter, [showTimePicker()](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/20_ShowTimePicker.md) is the framework's official modal dialog for picking a [TimeOfDay](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/18_TimePicker.md). It displays an interactive analog clock dial or digital keyboard input boxes, allows toggling between AM and PM, respects the device's 12-hour or 24-hour system locale, and asynchronously returns the selected [TimeOfDay](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/18_TimePicker.md) (or `null` if dismissed).

---

## 🤔 Why Do We Need It?
1. **Interactive Clock Face**: Users can drag the circular dial hand directly to their desired hour and minute markings.
2. **Dual Input Modes**: Seamlessly switches between an analog **Clock Dial** mode and a direct numeric **Keyboard Input** mode via a toggle button in the dialog header.
3. **Locale-Aware Formatting**: Automatically adapts to 24-hour military time in regions that use it (e.g. Europe) and 12-hour AM/PM format in others (e.g. United States).
4. **Clean Asynchronous Pattern**: Returns a `Future<TimeOfDay?>`, allowing clean integration with `async/await` syntax.

---

## 🌍 Real-Life Analogy
Think of [showTimePicker()](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/20_ShowTimePicker.md) as **Adjusting the Hands on a Traditional Bedside Alarm Clock**:

```
+-------------------------------------------------------------+
|                     BEDSIDE CLOCK DIAL                      |
|                                                             |
|                          12                                 |
|                     11        1                             |
|                  10              2                          |
|                 9        (•)      3                         |
|                  8      /        4                          |
|                     7        5                              |
|                          6                                  |
|                                                             |
|   You physically drag the hour hand to 7, and then the     |
|   minute hand to 30. If you prefer, you can flip to the    |
|   back and type the digits directly on the digital keypad! |
+-------------------------------------------------------------+
```

---

## 🧠 Internal Working
When `showTimePicker()` is invoked:

```
                     User Taps "Set Alarm Time"
                                 │
                                 ▼
         final TimeOfDay? picked = await showTimePicker(...)
                                 │
                                 ▼
              Navigator.push(_TimePickerDialogRoute)
                                 │
                                 ▼
                     TimePickerEntryMode Check
                     ├── dial  ──► Paints circular canvas with DragGesture
                     └── input ──► Paints dual TextFormField boxes (HH : MM)
                                 │
                                 ▼
                      User Confirms or Cancels
         ┌───────────────────────┴───────────────────────┐
         ▼                                               ▼
    Taps "CANCEL"                                   Taps "OK"
  Navigator.pop(null)                      Navigator.pop(selectedTime)
         │                                               │
         └───────────────────────┬───────────────────────┘
                                 │
                                 ▼
                     Future Completes In Code
                     picked = selectedTime OR null
                                 │
                                 ▼
                 Guarded by if (context.mounted)
```

1. **State Machine**: When the dialog opens in `dial` mode, it first displays the 12/24 hour circle. Once the user selects an hour, it automatically animates into the 60-minute circle.
2. **Platform 24-Hour Detection**: By default, Flutter checks `MediaQuery.alwaysUse24HourFormatOf(context)` to decide whether to render the `AM / PM` toggle segmented control.
3. **Customization via Theme**: The appearance (hand color, dial background, digit font size) can be configured globally via `TimePickerThemeData` or scoped using the `builder` parameter.

---

## ⚙ Syntax

### Standard Invocation
```dart
final TimeOfDay? pickedTime = await showTimePicker(
  context: context,
  initialTime: TimeOfDay.now(),
  initialEntryMode: TimePickerEntryMode.dial, // or .input
  helpText: 'Set Wake-Up Alarm',
  confirmText: 'Set Alarm',
  cancelText: 'Dismiss',
);

if (pickedTime != null) {
  print('Hour: ${pickedTime.hour}, Minute: ${pickedTime.minute}');
}
```

### Forcing 24-Hour Military Format
```dart
final TimeOfDay? picked = await showTimePicker(
  context: context,
  initialTime: const TimeOfDay(hour: 14, minute: 0),
  builder: (BuildContext context, Widget? child) {
    return MediaQuery(
      data: MediaQuery.of(context).copyWith(alwaysUse24HourFormat: true),
      child: child!,
    );
  },
);
```

---

## 📋 Important Parameters

| Parameter | Type | Description | Default |
| :--- | :--- | :--- | :--- |
| `context` | `BuildContext` | Build context for theme and route resolution. | *Required* |
| `initialTime` | `TimeOfDay` | The starting time displayed when the dialog opens. | *Required* |
| `initialEntryMode` | `TimePickerEntryMode` | `.dial` (interactive clock) or `.input` (keyboard text boxes). | `TimePickerEntryMode.dial` |
| `builder` | `TransitionBuilder?` | Wrap the dialog in a custom `Theme` or `MediaQuery` widget. | `null` |
| `helpText` | `String?` | Header title label displayed at the top of the dialog. | `'Select time'` |
| `cancelText` | `String?` | Custom label for dismiss button. | `'Cancel'` |
| `confirmText` | `String?` | Custom label for confirmation button. | `'OK'` |
| `orientation` | `Orientation?` | Forces portrait or landscape layout inside the dialog. | `null` (auto) |

---

## 💻 Complete Runnable Example
The following production-ready application demonstrates:
1. Setting morning and evening daily alarms using `showTimePicker()`.
2. Toggling between **Analog Dial** and **Digital Keypad Input** modes.
3. Forcing a custom 24-hour military time theme override.
4. Converting a `TimeOfDay` into a fully scheduled `DateTime` instance.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ShowTimePickerMasterclassApp());
}

class ShowTimePickerMasterclassApp extends StatelessWidget {
  const ShowTimePickerMasterclassApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'showTimePicker Masterclass',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const AlarmManagerScreen(),
    );
  }
}

class AlarmManagerScreen extends StatefulWidget {
  const AlarmManagerScreen({super.key});

  @override
  State<AlarmManagerScreen> createState() => _AlarmManagerScreenState();
}

class _AlarmManagerScreenState extends State<AlarmManagerScreen> {
  TimeOfDay _morningAlarm = const TimeOfDay(hour: 7, minute: 30);
  TimeOfDay _eveningReminder = const TimeOfDay(hour: 21, minute: 0);
  bool _use24HourFormat = false;

  Future<void> _pickMorningAlarm() async {
    final TimeOfDay? selected = await showTimePicker(
      context: context,
      initialTime: _morningAlarm,
      initialEntryMode: TimePickerEntryMode.dial,
      helpText: 'Set Morning Wake-Up Alarm',
      confirmText: 'Save Alarm',
      builder: (context, child) {
        return MediaQuery(
          data: MediaQuery.of(context).copyWith(
            alwaysUse24HourFormat: _use24HourFormat,
          ),
          child: child!,
        );
      },
    );

    if (!mounted) return;

    if (selected != null) {
      setState(() {
        _morningAlarm = selected;
      });
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          content: Text('Morning alarm set for ${selected.format(context)}'),
          backgroundColor: Colors.deepPurple,
        ),
      );
    }
  }

  Future<void> _pickEveningReminder() async {
    // Demonstrates opening directly in digital Keyboard Input mode
    final TimeOfDay? selected = await showTimePicker(
      context: context,
      initialTime: _eveningReminder,
      initialEntryMode: TimePickerEntryMode.input,
      helpText: 'Enter Bedtime Reminder Time',
      confirmText: 'Set Reminder',
      builder: (context, child) {
        return MediaQuery(
          data: MediaQuery.of(context).copyWith(
            alwaysUse24HourFormat: _use24HourFormat,
          ),
          child: child!,
        );
      },
    );

    if (!mounted) return;

    if (selected != null) {
      setState(() {
        _eveningReminder = selected;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Smart Alarm Manager'),
        centerTitle: true,
      ),
      body: SafeArea(
        child: ListView(
          padding: const EdgeInsets.all(20.0),
          children: [
            const Text(
              'Scheduled Alarms',
              style: TextStyle(fontSize: 22, fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 6),
            const Text(
              'Tap each card to configure alarm times using the modal time picker.',
              style: TextStyle(color: Colors.grey),
            ),
            const SizedBox(height: 16),

            // 24-Hour Format Switch
            SwitchListTile(
              title: const Text('Force 24-Hour Military Format'),
              subtitle: const Text('Overrides device locale in time pickers'),
              value: _use24HourFormat,
              onChanged: (bool val) => setState(() => _use24HourFormat = val),
            ),
            const Divider(height: 24),

            // Card 1: Morning Alarm (Opens Dial mode)
            Card(
              elevation: 0,
              shape: RoundedRectangleBorder(
                borderRadius: BorderRadius.circular(16),
                side: BorderSide(color: Colors.grey.shade300),
              ),
              child: ListTile(
                contentPadding: const EdgeInsets.all(16),
                leading: const CircleAvatar(
                  backgroundColor: Colors.amber,
                  child: Icon(Icons.wb_sunny, color: Colors.white),
                ),
                title: const Text('Wake-Up Alarm (Clock Dial)', style: TextStyle(fontWeight: FontWeight.bold)),
                subtitle: const Text('Tap to adjust with analog clock hand'),
                trailing: Text(
                  _morningAlarm.format(context),
                  style: const TextStyle(fontSize: 20, fontWeight: FontWeight.bold, color: Colors.deepPurple),
                ),
                onTap: _pickMorningAlarm,
              ),
            ),
            const SizedBox(height: 16),

            // Card 2: Evening Reminder (Opens Input mode)
            Card(
              elevation: 0,
              shape: RoundedRectangleBorder(
                borderRadius: BorderRadius.circular(16),
                side: BorderSide(color: Colors.grey.shade300),
              ),
              child: ListTile(
                contentPadding: const EdgeInsets.all(16),
                leading: const CircleAvatar(
                  backgroundColor: Colors.indigo,
                  child: Icon(Icons.bedtime, color: Colors.white),
                ),
                title: const Text('Bedtime Reminder (Keypad Input)', style: TextStyle(fontWeight: FontWeight.bold)),
                subtitle: const Text('Tap to enter digits directly via keyboard'),
                trailing: Text(
                  _eveningReminder.format(context),
                  style: const TextStyle(fontSize: 20, fontWeight: FontWeight.bold, color: Colors.indigo),
                ),
                onTap: _pickEveningReminder,
              ),
            ),
            const SizedBox(height: 32),

            // Next Alarm Banner
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.deepPurple.shade50,
                borderRadius: BorderRadius.circular(12),
                border: Border.all(color: Colors.deepPurple.shade200),
              ),
              child: Row(
                children: [
                  const Icon(Icons.alarm_on, color: Colors.deepPurple, size: 36),
                  const SizedBox(width: 12),
                  Expanded(
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        const Text(
                          'NEXT ACTIVE ALARM',
                          style: TextStyle(fontSize: 11, fontWeight: FontWeight.bold, color: Colors.deepPurple),
                        ),
                        Text(
                          'Ring tomorrow morning at ${_morningAlarm.format(context)}',
                          style: const TextStyle(fontWeight: FontWeight.w600, fontSize: 14),
                        ),
                      ],
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
- `final TimeOfDay? selected = await showTimePicker(...)`:
  - **What it does**: Pushes the modal time picker dialog and waits asynchronously for user confirmation or cancellation.
  - **Why it exists**: Provides an out-of-the-box Material 3 dialog with accessibility hooks, keyboard shortcuts, and animation transitions.
- `initialEntryMode: TimePickerEntryMode.input`:
  - **What it does**: Directs the dialog to open showing digital number boxes instead of the clock face.
  - **Why it exists**: Users on desktop or those with motor control challenges often prefer typing numbers over circular dragging.
- `alwaysUse24HourFormat: _use24HourFormat`:
  - **What it does**: Overrides the `MediaQuery` passed down to the dialog.
  - **Why it exists**: If `true`, the clock dial renders 24-hour numbers (`00` through `23`) and hides the AM/PM selector.
- `_morningAlarm.format(context)`:
  - **What it does**: Formats `TimeOfDay` into a localized string (`"7:30 AM"` or `"07:30"`).
  - **Fatal mistake if avoided**: Calling `_morningAlarm.toString()` returns raw debug text like `"TimeOfDay(07:30)"`, which is unsuitable for user-facing UI.

---

## 📱 ASCII Phone Screen Preview
```
+------------------------------------------+
|           Smart Alarm Manager            |
+------------------------------------------+
| Scheduled Alarms                         |
| Tap each card to configure...            |
|                                          |
| Force 24-Hour Military Format    [===O ] |
| ---------------------------------------- |
|                                          |
| +--------------------------------------+ |
| | (☀) Wake-Up Alarm (Clock Dial)       | |
| |     Tap to adjust...       07:30 AM  | |
| +--------------------------------------+ |
|                                          |
|      ┌─────────────────────────────┐     |
|      │ Set Morning Alarm           │     |
|      │ 07 : 30        (AM)  |  PM  │     |
|      │-----------------------------│     |
|      │              12             │     |
|      │          11       1         │     |
|      │       10             2      │     |
|      │      9        (•)     3     │     |
|      │       8      /        4     │     |
|      │         (7)       5         │     |
|      │               6             │     |
|      │                             │     |
|      │       [DISMISS]  [SAVE ALARM]│    |
|      └─────────────────────────────┘     |
|                                          |
| +--------------------------------------+ |
| | (🌙) Bedtime Reminder                | |
| |     Tap to enter...        09:00 PM  | |
| +--------------------------------------+ |
|                                          |
| +--------------------------------------+ |
| | [⏰] NEXT ACTIVE ALARM               | |
| |      Ring tomorrow morning at 7:30 AM| |
| +--------------------------------------+ |
+------------------------------------------+
```

---

## 🌳 Widget Tree
```
MaterialApp
 └── AlarmManagerScreen (StatefulWidget)
      └── Scaffold
           ├── AppBar
           └── SafeArea
                └── ListView
                     ├── Text ("Scheduled Alarms")
                     ├── SwitchListTile (24-Hour Format)
                     ├── Divider
                     ├── Card -> ListTile (Morning Alarm, onTap: _pickMorningAlarm)
                     ├── Card -> ListTile (Bedtime Reminder, onTap: _pickEveningReminder)
                     └── Container (Next Active Alarm banner)

[ Overlay Layer during Dialog ]:
 ModalBarrier
  └── DialogRoute<TimeOfDay>
       └── _TimePickerDialog (Pushed onto Navigator)
```

---

## ⚙ Flutter Execution Flow
1. **User Action**: User taps the Morning Alarm card.
2. **Launch**: `showTimePicker()` is invoked with `initialTime: TimeOfDay(hour: 7, minute: 30)`.
3. **Route Mount**: Modal dialog opens displaying the hour circle with `7` selected.
4. **Hour Selection**: User drags the dial to `6`. The dialog immediately animates into the minute circle.
5. **Minute Selection**: User drags the hand to `45`.
6. **Confirmation**: User taps "Save Alarm".
7. **Resolution**: `Navigator.pop(TimeOfDay(hour: 6, minute: 45))` completes the `Future`.
8. **Safety Check**: `if (!mounted) return;` confirms the widget is still alive.
9. **State Mutation**: `setState()` saves the new time, and the screen repaints with `"6:45 AM"`.

---

## ⚠ Common Beginner Mistakes

### 1. Using `.toString()` Instead of `.format(context)`
```dart
// ❌ UGLY UI: Displays debug text
Text('Time: ${_time.toString()}'); // Renders: "Time: TimeOfDay(07:30)"!

// ✅ GOOD: Use localized format method
Text('Time: ${_time.format(context)}'); // Renders: "Time: 7:30 AM" or "07:30"
```

### 2. Assuming `showTimePicker` Always Returns a Value
```dart
// ❌ CRASH: Assuming non-null without checking
final picked = await showTimePicker(context: context, initialTime: TimeOfDay.now());
_saveToDatabase(picked.hour); // Null check operator crash if user clicked Cancel!

// ✅ GOOD: Always null-check
if (picked != null) {
  _saveToDatabase(picked.hour);
}
```

### 3. Missing `mounted` Check Across Async Gap
```dart
// ❌ WARNING:
final picked = await showTimePicker(...);
setState(() => _time = picked); // Linter error: async gap

// ✅ GOOD:
final picked = await showTimePicker(...);
if (!mounted) return;
if (picked != null) {
  setState(() => _time = picked);
}
```

---

## 🚀 Professional Usage
- **Healthcare & Doctor Consultations (e.g., Zocdoc)**: Selecting specific 15-minute appointment arrival times.
- **Restaurant & Food Delivery (e.g., DoorDash, OpenTable)**: Choosing table reservation times or "Deliver at specific time" windows.
- **Flight & Transit Schedules**: Displaying departure and arrival clock times in 24-hour military format.

---

## 💡 Best Practices
1. **Always use `time.format(context)`**: Never manually assemble strings with string concatenation; `format(context)` respects device preferences and language locales.
2. **Combine with `DateTime` properly**: When saving to an API or database, combine today's `DateTime` with `TimeOfDay`:
   ```dart
   final now = DateTime.now();
   final fullDate = DateTime(now.year, now.month, now.day, time.hour, time.minute);
   ```
3. **Choose `initialEntryMode` thoughtfully**: Use `.input` for data-heavy desktop environments and `.dial` for touchscreen mobile interfaces.
4. **Guard with `if (!mounted) return;`**: Always guarantee widget lifecycle safety following `await`.

---

## 🧪 Try It Yourself
1. Add a second button that forces `TimePickerEntryMode.input` and observe how the keyboard immediately pops up for digit entry.
2. Wrap the `showTimePicker` builder in a custom `TimePickerThemeData` to change the dial color to emerald green.
3. Calculate the time remaining until the next alarm (e.g. *"Alarm rings in 8 hours and 15 minutes"*).

---

## 🎯 Challenge
Create a "Work Shift Scheduler":
- Pick a Shift Start Time (e.g. 09:00 AM).
- Pick a Shift End Time (e.g. 05:30 PM).
- Calculate and display the total paid shift hours (e.g. `8.5 Hours`), accounting for overnight shifts that wrap past midnight.

---

## ❓ Practice Questions

### Q1. What does `showTimePicker()` return if the user dismisses the dialog without selecting a time?
- A) `TimeOfDay.now()`
- B) `TimeOfDay(hour: 0, minute: 0)`
- C) `null`
- D) An exception
*Answer: C* — Dismissing the dialog resolves the Future with `null`.

### Q2. How can you display a `TimeOfDay` object formatted according to the user's phone locale?
- A) `time.toString()`
- B) `time.format(context)`
- C) `time.toIso8601String()`
- D) `time.toJson()`
*Answer: B* — `format(context)` handles 12h/24h and localized AM/PM symbols.

### Q3. Which `TimePickerEntryMode` displays interactive circular clock hands?
- A) `TimePickerEntryMode.input`
- B) `TimePickerEntryMode.dial`
- C) `TimePickerEntryMode.analog`
- D) `TimePickerEntryMode.clock`
*Answer: B* — `.dial` shows the interactive circular clock dial.

### Q4. How do you force a 24-hour military clock layout inside `showTimePicker`?
- A) Set `is24Hour: true`
- B) Set `militaryTime: true`
- C) Override `MediaQueryData.alwaysUse24HourFormat = true` in the `builder` callback
- D) Set `hourRange: 24`
*Answer: C* — Injecting `alwaysUse24HourFormat: true` via `MediaQuery` forces 24-hour time.

### Q5. What is the range of values for `TimeOfDay.hour`?
- A) 1 to 12
- B) 0 to 23
- C) 0 to 24
- D) 1 to 24
*Answer: B* — Standard 24-hour internal representation (0 represents midnight, 23 represents 11 PM).

### Q6. How do you combine a `DateTime date` and a `TimeOfDay time` into a single `DateTime` instance?
*Answer*:
```dart
DateTime(date.year, date.month, date.day, time.hour, time.minute);
```

### Q7. What happens when the user finishes dragging the hour hand on the clock dial?
*Answer*: The clock face automatically animates to the minute selection ring.

### Q8. Output Prediction: What will `const TimeOfDay(hour: 13, minute: 5).format(context)` display in US English locale?
*Answer*: `"1:05 PM"`.

### Q9. Can the user switch between Dial and Keypad modes while the dialog is open?
*Answer*: Yes, tapping the small keyboard or clock icon in the bottom corner of the dialog toggles dynamically between `dial` and `input` modes.

### Q10. Write a minimal code snippet to launch `showTimePicker()` starting at 5:00 PM.
*Answer*:
```dart
final time = await showTimePicker(
  context: context,
  initialTime: const TimeOfDay(hour: 17, minute: 0),
);
```

---

## 📝 Summary
- [showTimePicker()](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/20_ShowTimePicker.md) launches a modal dialog for selecting clock times via interactive dial or direct keyboard input.
- Returns a `Future<TimeOfDay?>`; `null` indicates cancellation.
- Always use `time.format(context)` for clean, localized string presentation.
- Check `if (!mounted) return;` across the asynchronous dialog gap.
- Override `alwaysUse24HourFormat` via the `builder` parameter when military time is strictly required.
