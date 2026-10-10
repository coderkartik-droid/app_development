# TimePicker & CupertinoTimerPicker in Flutter

## 📖 Introduction
Just as selecting calendar dates benefits from dedicated visual tools, entering time durations and clock times demands specialized controls. Whether setting an alarm clock for `07:30 AM`, choosing a delivery arrival window, or configuring a kitchen cooking timer for `12 minutes and 45 seconds`, typing raw numbers into a text field produces syntax errors and validation headaches.

In Flutter, time selection is handled through two distinct concepts:
1. **Clock Time of Day** ([TimeOfDay](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/18_TimePicker.md)): Represents a specific moment on a 12-hour or 24-hour clock (e.g. `14:45` or `02:45 PM`). While `showTimePicker()` opens a modal clock dial (explored in Chapter 20), inline layouts can display and manipulate `TimeOfDay` directly.
2. **Elapsed Duration** ([CupertinoTimerPicker](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/18_TimePicker.md)): An inline iOS-style mechanical spinning wheel widget designed specifically for selecting hours, minutes, and seconds as a [Duration](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/18_TimePicker.md).

---

## 🤔 Why Do We Need It?
Understanding when to use clock time vs countdown duration is essential:
- **`TimeOfDay`**: Answers *"At what time does this event start?"* (e.g. Meeting starts at 10:00 AM).
- **`Duration` ([CupertinoTimerPicker](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/18_TimePicker.md))**: Answers *"How long will this activity last?"* (e.g. Meditate for 15 minutes and 30 seconds).

Why use [CupertinoTimerPicker](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/18_TimePicker.md)?
1. **Zero Text Typing**: Users roll drum wheels with tactile, physics-based inertial scrolling.
2. **Configurable Intervals**: Restrict selections to 5-minute or 15-minute intervals via `minuteInterval`.
3. **Flexible Modes**: Toggle between `hours/minutes/seconds`, `hours/minutes`, or pure `minutes/seconds`.

---

## 🌍 Real-Life Analogy
Think of [CupertinoTimerPicker](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/18_TimePicker.md) as a **Mechanical Egg Timer on a Kitchen Counter**:

```
+-------------------------------------------------------------+
|                     KITCHEN EGG TIMER                       |
|                                                             |
|           [ 00 hours ]   [ 12 mins ]   [ 30 secs ]          |
|                 ▲              ▲             ▲              |
|                 │              │             │              |
|   You spin the mechanical dial with your thumb until it     |
|   clicks on 12 minutes. You don't program a clock—you      |
|   set a countdown duration!                                 |
+-------------------------------------------------------------+
```

---

## 🧠 Internal Working
[CupertinoTimerPicker](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/18_TimePicker.md) uses cylindrical 3D projections via multiple synchronized `ListWheelScrollView` render objects:

```
                    CupertinoTimerPicker
                             │
         ┌───────────────────┼───────────────────┐
         ▼                   ▼                   ▼
    Hour Wheel          Minute Wheel        Second Wheel
  (0 to 23 items)     (0 to 59 items)     (0 to 59 items)
         │                   │                   │
         └───────────────────┼───────────────────┘
                             │
                             ▼
                Physics-based Inertial Drag
                             │
                             ▼
         Calculates Selected Indices (h, m, s)
                             │
                             ▼
         Duration(hours: h, minutes: m, seconds: s)
                             │
                             ▼
             onTimerDurationChanged(Duration)
```

1. **Cylindrical Wheel Rendering**: Each column is a `ListWheelScrollView` that renders items curved along a virtual 3D cylinder.
2. **Snapping Physics**: As the user flings the wheel, `FixedExtentScrollPhysics` snaps the wheel to the nearest integer index with haptic feedback.
3. **Duration Output**: Flutter combines the indices into a `Duration` and invokes `onTimerDurationChanged`.

---

## ⚙ Syntax

### CupertinoTimerPicker
```dart
SizedBox(
  height: 200, // Explicit height is REQUIRED!
  child: CupertinoTimerPicker(
    mode: CupertinoTimerPickerMode.hms, // hours, minutes, seconds
    initialTimerDuration: const Duration(minutes: 10),
    minuteInterval: 1, // Step interval for minutes
    secondInterval: 5, // Step interval for seconds
    onTimerDurationChanged: (Duration newDuration) {
      setState(() {
        _timerDuration = newDuration;
      });
    },
  ),
)
```

### TimeOfDay Construction & Formatting
```dart
// 1. Create a TimeOfDay instance
const TimeOfDay meetingTime = TimeOfDay(hour: 14, minute: 30); // 2:30 PM

// 2. Format to 12-hour or 24-hour string according to device locale
final String formatted = meetingTime.format(context); // e.g. "2:30 PM"
```

---

## 📋 Important Properties (CupertinoTimerPicker)

| Property | Type | Description | Default |
| :--- | :--- | :--- | :--- |
| `mode` | `CupertinoTimerPickerMode` | Display columns: `hms` (hours/mins/secs), `hm` (hours/mins), or `ms` (mins/secs). | `CupertinoTimerPickerMode.hms` |
| `initialTimerDuration` | `Duration` | The duration selected when first rendered. | `Duration.zero` |
| `onTimerDurationChanged` | `ValueChanged<Duration>` | Called continuously as the wheels scroll and settle. | *Required* |
| `minuteInterval` | `int` | Granularity of minutes wheel. Must divide 60 evenly (1, 2, 3, 4, 5, 6, 10, 12, 15, 20, 30). | `1` |
| `secondInterval` | `int` | Granularity of seconds wheel. Must divide 60 evenly (1, 2, 3, 4, 5, 6, 10, 12, 15, 20, 30). | `1` |
| `alignment` | `AlignmentGeometry` | Alignment of the picker wheels within its parent. | `Alignment.center` |

---

## 💻 Complete Runnable Example
The following production-ready application demonstrates:
1. An embedded [CupertinoTimerPicker](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/18_TimePicker.md) used to configure a high-intensity interval training (HIIT) workout timer.
2. Switching picker modes between `Hours:Minutes:Seconds` and `Minutes:Seconds`.
3. Live formatted countdown display badge with clean two-digit padding.

```dart
import 'package:flutter/cupertino.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(const TimePickerMasterclassApp());
}

class TimePickerMasterclassApp extends StatelessWidget {
  const TimePickerMasterclassApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Time & Duration Picker',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepOrange),
        useMaterial3: true,
      ),
      home: const WorkoutTimerConfigScreen(),
    );
  }
}

class WorkoutTimerConfigScreen extends StatefulWidget {
  const WorkoutTimerConfigScreen({super.key});

  @override
  State<WorkoutTimerConfigScreen> createState() =>
      _WorkoutTimerConfigScreenState();
}

class _WorkoutTimerConfigScreenState extends State<WorkoutTimerConfigScreen> {
  // Configured duration state
  Duration _workoutDuration = const Duration(minutes: 5, seconds: 30);
  CupertinoTimerPickerMode _pickerMode = CupertinoTimerPickerMode.ms;

  String _formatDuration(Duration d) {
    final hours = d.inHours;
    final minutes = d.inMinutes.remainder(60);
    final seconds = d.inSeconds.remainder(60);

    if (hours > 0) {
      return '${hours.toString().padLeft(2, '0')}:'
          '${minutes.toString().padLeft(2, '0')}:'
          '${seconds.toString().padLeft(2, '0')}';
    }
    return '${minutes.toString().padLeft(2, '0')}:${seconds.toString().padLeft(2, '0')}';
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Workout Interval Timer'),
        centerTitle: true,
      ),
      body: SafeArea(
        child: SingleChildScrollView(
          padding: const EdgeInsets.all(20.0),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: [
              const Text(
                'Configure Exercise Duration',
                style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
              ),
              const SizedBox(height: 6),
              const Text(
                'Spin the wheels to set your countdown timer target.',
                style: TextStyle(color: Colors.grey),
              ),
              const SizedBox(height: 20),

              // Mode Selector Segmented Button
              SegmentedButton<CupertinoTimerPickerMode>(
                segments: const [
                  ButtonSegment(
                    value: CupertinoTimerPickerMode.ms,
                    label: Text('Min : Sec'),
                    icon: Icon(Icons.timer_outlined),
                  ),
                  ButtonSegment(
                    value: CupertinoTimerPickerMode.hms,
                    label: Text('Hr : Min : Sec'),
                    icon: Icon(Icons.hourglass_bottom),
                  ),
                ],
                selected: {_pickerMode},
                onSelectionChanged: (Set<CupertinoTimerPickerMode> newSelection) {
                  setState(() {
                    _pickerMode = newSelection.first;
                  });
                },
              ),
              const SizedBox(height: 20),

              // Embedded Cupertino Timer Picker inside an aesthetic Card
              Card(
                elevation: 2,
                shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(16)),
                child: Padding(
                  padding: const EdgeInsets.symmetric(vertical: 16.0),
                  child: SizedBox(
                    height: 200, // CRITICAL: Explicit height constraint
                    child: CupertinoTimerPicker(
                      key: ValueKey(_pickerMode), // Forces rebuild when mode changes
                      mode: _pickerMode,
                      initialTimerDuration: _workoutDuration,
                      minuteInterval: 1,
                      secondInterval: 5, // Snaps every 5 seconds
                      onTimerDurationChanged: (Duration newDuration) {
                        setState(() {
                          _workoutDuration = newDuration;
                        });
                      },
                    ),
                  ),
                ),
              ),
              const SizedBox(height: 24),

              // Live Digital Display Card
              Container(
                padding: const EdgeInsets.all(20),
                decoration: BoxDecoration(
                  color: Colors.deepOrange.shade50,
                  borderRadius: BorderRadius.circular(16),
                  border: Border.all(color: Colors.deepOrange.shade200),
                ),
                child: Column(
                  children: [
                    const Text(
                      'TOTAL WORKOUT DURATION',
                      style: TextStyle(
                        fontSize: 12,
                        fontWeight: FontWeight.bold,
                        letterSpacing: 1.5,
                        color: Colors.deepOrange,
                      ),
                    ),
                    const SizedBox(height: 8),
                    Text(
                      _formatDuration(_workoutDuration),
                      style: TextStyle(
                        fontFamily: 'monospace',
                        fontSize: 48,
                        fontWeight: FontWeight.bold,
                        color: Colors.deepOrange.shade900,
                      ),
                    ),
                    const SizedBox(height: 4),
                    Text(
                      '(${_workoutDuration.inSeconds} Total Seconds)',
                      style: TextStyle(color: Colors.grey.shade700, fontSize: 13),
                    ),
                  ],
                ),
              ),
              const SizedBox(height: 24),

              FilledButton.icon(
                onPressed: _workoutDuration.inSeconds > 0
                    ? () {
                        ScaffoldMessenger.of(context).showSnackBar(
                          SnackBar(
                            content: Text(
                              'Timer started for ${_formatDuration(_workoutDuration)}!',
                            ),
                            backgroundColor: Colors.deepOrange,
                          ),
                        );
                      }
                    : null,
                icon: const Icon(Icons.play_arrow),
                label: const Text('Start Workout Session', style: TextStyle(fontSize: 16)),
                style: FilledButton.styleFrom(
                  padding: const EdgeInsets.symmetric(vertical: 16),
                ),
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
- `SizedBox(height: 200, child: CupertinoTimerPicker(...))`:
  - **What it does**: Constrains the spinning wheel widget to 200 logical pixels vertically.
  - **Why it exists**: `CupertinoTimerPicker` has unbounded intrinsic height. Omitting this height constraint will throw an immediate `RenderFlex children have non-zero flex but incoming height constraints are unbounded` error!
- `secondInterval: 5`:
  - **What it does**: Restricts the seconds wheel to multiples of 5 (`00, 05, 10, ..., 55`).
  - **Why it exists**: Saves users from unnecessary scrolling when fine-grained second precision isn't required.
  - **Rule**: `secondInterval` must divide 60 evenly. Passing `7` will trigger an assertion failure!
- `key: ValueKey(_pickerMode)`:
  - **What it does**: Assigns a unique key based on the active picker mode (`ms` vs `hms`).
  - **Why it exists**: When the segmented button changes modes, the key ensures the old wheel layout is destroyed and remounted with the correct column count.
- `_formatDuration(Duration d)`:
  - **What it does**: Extracts `inHours`, `remainder(60)` for minutes, and `remainder(60)` for seconds, applying `.padLeft(2, '0')`.
  - **Why it exists**: Guarantees consistent digital clock formatting like `05:30` rather than `5:30`.

---

## 📱 ASCII Phone Screen Preview
```
+------------------------------------------+
|          Workout Interval Timer          |
+------------------------------------------+
| Configure Exercise Duration              |
| Spin the wheels to set target...         |
|                                          |
| ( [Min : Sec] )   |   [Hr : Min : Sec]   |
|                                          |
| +--------------------------------------+ |
| |          04 min         25 sec       | |
| |       >  05 min   :  >  30 sec       | |
| |          06 min         35 sec       | |
| +--------------------------------------+ |
|                                          |
| +--------------------------------------+ |
| |       TOTAL WORKOUT DURATION         | |
| |                                      | |
| |              05:30                   | |
| |                                      | |
| |        (330 Total Seconds)           | |
| +--------------------------------------+ |
|                                          |
| +--------------------------------------+ |
| |      [▶] Start Workout Session       | |
| +--------------------------------------+ |
+------------------------------------------+
```

---

## 🌳 Widget Tree
```
MaterialApp
 └── WorkoutTimerConfigScreen (StatefulWidget)
      └── Scaffold
           ├── AppBar
           └── SafeArea
                └── SingleChildScrollView
                     └── Column
                          ├── Text ("Configure Exercise Duration")
                          ├── SegmentedButton (ms vs hms mode)
                          ├── Card
                          │    └── SizedBox [height: 200]
                          │         └── CupertinoTimerPicker
                          ├── Container (Duration digital display)
                          └── FilledButton ("Start Workout")
```

---

## ⚙ Flutter Execution Flow
1. **Initial Mount**: `CupertinoTimerPicker` renders with 2 spinning wheels (minutes and seconds) initialized to 5 min, 30 sec.
2. **User Gesture**: User scrolls the minutes wheel from `05` down to `10`.
3. **Inertial Physics**: The drum wheel spins and snaps smoothly to `10`.
4. **Callback**: `onTimerDurationChanged(Duration(minutes: 10, seconds: 30))` fires.
5. **State Update**: `setState()` assigns `_workoutDuration = newDuration`.
6. **Repaint**: The digital clock display updates instantaneously to `10:30 (630 Total Seconds)`.

---

## ⚠ Common Beginner Mistakes

### 1. Unbounded Height Layout Exception
```dart
// ❌ LAYOUT CRASH: CupertinoTimerPicker has no vertical bound!
Column(
  children: [
    CupertinoTimerPicker(
      onTimerDurationChanged: (d) {},
    ),
  ],
)
// Result: Crash! RenderBox was not laid out: RenderCustomMultiChildLayoutBox

// ✅ GOOD: Always wrap in SizedBox or Container with explicit height
SizedBox(
  height: 200,
  child: CupertinoTimerPicker(
    onTimerDurationChanged: (d) {},
  ),
)
```

### 2. Passing Interval That Does Not Divide 60 Evenly
```dart
// ❌ CRASH: 7 does not divide 60 evenly!
CupertinoTimerPicker(
  minuteInterval: 7, // Assertion failure: 60 % minuteInterval == 0
  onTimerDurationChanged: (d) {},
)

// ✅ GOOD: Use factors of 60: 1, 2, 3, 4, 5, 6, 10, 12, 15, 20, 30
CupertinoTimerPicker(
  minuteInterval: 5,
  onTimerDurationChanged: (d) {},
)
```

### 3. Confusing `TimeOfDay` with `Duration`
```dart
// ❌ CONCEPTUAL MISTAKE:
// Using TimeOfDay for a 15-minute countdown timer:
TimeOfDay timer = TimeOfDay(hour: 0, minute: 15); // Represents 00:15 AM!

// ✅ GOOD: Use Duration for elapsed countdowns, TimeOfDay for clock moments:
Duration timer = const Duration(minutes: 15);
```

---

## 🚀 Professional Usage
- **Fitness & Workout Apps (e.g., Nike Training Club, Strava)**: Configuring rest interval timers between heavy weightlifting sets.
- **Smart Home & Cooking (e.g., Philips Hue, Instant Pot)**: Setting smart light sleep timers and pressure cooker cook times.
- **Productivity & Focus (e.g., Forest, Pomodoro Timer)**: Customizing study intervals (25 min focus / 5 min break).

---

## 💡 Best Practices
1. **Always wrap `CupertinoTimerPicker` in a `SizedBox(height: 200)`**: Ensures reliable rendering across all platforms.
2. **Format with `.padLeft(2, '0')`**: Always display times with leading zeros (`09:05` instead of `9:5`).
3. **Select the appropriate `mode`**: Don't force users to scroll past an unnecessary hours wheel if the maximum duration is under an hour (`CupertinoTimerPickerMode.ms`).
4. **Use `ValueKey` when toggling modes dynamically**: Forces Flutter to cleanly recreate the internal wheel controllers.

---

## 🧪 Try It Yourself
1. Switch `secondInterval: 1` and notice the finer scrolling granularity on the seconds wheel.
2. Add a `CupertinoTimerPickerMode.hm` (Hours and Minutes) option to the segmented button.
3. Build a working timer using `dart:async`'s `Timer.periodic` that actually counts down to zero when the user taps "Start Workout".

---

## 🎯 Challenge
Create a "Pomodoro Focus Setup" screen:
- A `CupertinoTimerPicker` to configure Focus Time (e.g. 25 minutes).
- A second `CupertinoTimerPicker` to configure Short Break Time (e.g. 5 minutes).
- A button that validates both durations are greater than zero before launching a focus session.

---

## ❓ Practice Questions

### Q1. What is the fundamental difference between `TimeOfDay` and `Duration`?
- A) `TimeOfDay` is deprecated
- B) `TimeOfDay` represents a specific clock time on the wall (e.g. 2:30 PM), while `Duration` represents an elapsed amount of time (e.g. 2 hours and 30 minutes)
- C) `Duration` only works on Android
- D) `TimeOfDay` requires milliseconds
*Answer: B* — `TimeOfDay` is clock-bound, whereas `Duration` measures an elapsed interval.

### Q2. Why does `CupertinoTimerPicker` throw an exception if placed directly inside an unconstrained `Column`?
- A) It is an iOS widget running on Android
- B) It has unbounded intrinsic height and requires an explicit height boundary (like a `SizedBox`)
- C) It requires a GlobalKey
- D) It only works inside a Dialog
*Answer: B* — Spinning wheels require defined layout boundaries to calculate perspective.

### Q3. Which `minuteInterval` value will cause Flutter to crash with an assertion failure?
- A) `5`
- B) `10`
- C) `8`
- D) `15`
*Answer: C* — 8 does not divide 60 evenly (`60 % 8 != 0`), violating the framework assertion.

### Q4. Which `CupertinoTimerPickerMode` should be used for a 30-minute kitchen timer where hours are unnecessary?
- A) `CupertinoTimerPickerMode.hms`
- B) `CupertinoTimerPickerMode.hm`
- C) `CupertinoTimerPickerMode.ms`
- D) `CupertinoTimerPickerMode.seconds`
*Answer: C* — `ms` shows only the Minutes and Seconds wheels.

### Q5. What does `timeOfDay.format(context)` accomplish?
- A) Saves the time to SQLite
- B) Formats the `TimeOfDay` into a localized string (e.g. `2:30 PM` or `14:30`) according to the user's 12h/24h device locale settings
- C) Changes the device clock
- D) Sets a system alarm
*Answer: B* — It respects platform localization and device 12/24-hour preferences.

### Q6. How do you convert a 24-hour `TimeOfDay(hour: 15, minute: 0)` to total minutes from midnight?
*Answer*:
```dart
int totalMinutes = (time.hour * 60) + time.minute; // 900 minutes
```

### Q7. What property determines the starting position of the wheels in `CupertinoTimerPicker`?
*Answer*: `initialTimerDuration`.

### Q8. Output Prediction: What will `Duration(minutes: 65).inHours` return?
*Answer*: `1`. The `inHours` getter truncates division to complete whole hours.

### Q9. How do you extract the remaining minutes (0 to 59) from a duration that spans 2 hours and 15 minutes?
*Answer*: `duration.inMinutes.remainder(60)` returns `15`.

### Q10. Write a minimal code snippet creating a `TimeOfDay` instance representing 8:15 AM.
*Answer*:
```dart
const TimeOfDay breakfast = TimeOfDay(hour: 8, minute: 15);
```

---

## 📝 Summary
- Use `TimeOfDay` for clock times (alarms, appointments) and `Duration` for elapsed countdown intervals.
- [CupertinoTimerPicker](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/18_TimePicker.md) provides an iOS-style physical spinning wheel picker for durations.
- Always constrain `CupertinoTimerPicker` with an explicit height (e.g. `SizedBox(height: 200)`).
- Ensure `minuteInterval` and `secondInterval` divide 60 evenly to prevent assertion crashes.
- Use `time.format(context)` to guarantee localized 12-hour or 24-hour time formatting across devices.
