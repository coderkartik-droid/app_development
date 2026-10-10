# ValueListenableBuilder in Flutter

## 📖 Introduction

In the previous chapter, we explored `ValueNotifier<T>`, which encapsulates a single piece of mutable data and automatically notifies listeners when its value changes. But how do we actually consume that value in our widget tree with surgical precision?

Enter **`ValueListenableBuilder<T>`**.

`ValueListenableBuilder<T>` is a built-in Flutter widget that listens to a `ValueListenable<T>` (most commonly a `ValueNotifier<T>`) and rebuilds **only its own builder callback** whenever the value changes.

Instead of converting an entire screen into a `StatefulWidget` and calling `setState()`—which marks the whole screen dirty and rebuilds widgets that never changed—`ValueListenableBuilder` confines rebuilds to the exact leaf node displaying the data.

Even better: `ValueListenableBuilder` includes an optional **`child` parameter optimization**. If you have an expensive sub-tree (such as a complex vector graphic, heavy container, or large card) that does *not* depend on the changing value, you can pass it to the `child` argument. Flutter will build that sub-tree once and reuse it across every rebuild, saving immense CPU and GPU resources.

In this chapter, we will master the architecture of `ValueListenableBuilder`, dissect its internal subscription mechanism, understand the `child` performance trick, and build an interactive multi-gauge dashboard.

---

## 🤔 Why Do We Need It?

To appreciate why `ValueListenableBuilder` is so vital, let's examine what happens when you build a timer or a high-frequency slider using standard `setState()`:

```dart
// ❌ Anti-pattern: High-frequency rebuild of an entire screen
class _TimerScreenState extends State<TimerScreen> {
  int _seconds = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Workout Timer')), // REBUILDS EVERY SECOND!
      drawer: const AppDrawer(),                           // REBUILDS EVERY SECOND!
      body: Center(
        child: Column(
          children: [
            const HeavyHeaderIllustration(),              // REBUILDS EVERY SECOND!
            Text('Seconds: $_seconds'),                   // THE ONLY THING THAT CHANGED!
          ],
        ),
      ),
    );
  }
}
```

Every single second, `setState()` executes. Flutter re-runs the entire `build()` method, recreating the `AppBar`, the `AppDrawer`, the `HeavyHeaderIllustration`, and the `Column`. While Flutter is fast, doing this at 60 FPS or on deep widget trees drains the phone's battery and causes noticeable frame drops (jank).

### The `ValueListenableBuilder` Solution
```dart
// ✔ Surgical precision: ONLY the text rebuilds!
ValueListenableBuilder<int>(
  valueListenable: _secondsNotifier,
  builder: (context, seconds, _) {
    return Text('Seconds: $seconds');
  },
)
```

The `AppBar`, `AppDrawer`, and `HeavyHeaderIllustration` are built **once** and never touched again. Only the tiny `Text` widget repaints!

---

## 🌍 Real-Life Analogy

```
┌────────────────────────────────────────────────────────┐
│                   THE AIRPORT DEPARTURE GATE           │
│                                                        │
│  [setState() - Closing the Entire Airport]:            │
│  Flight 402 changes gates from Gate 12 to Gate 14.     │
│  The airport shuts down all terminals, evacuates all   │
│  passengers, re-prints every ticket, and turns on all  │
│  the runway lights again just to change one gate!      │
│                                                        │
│  [ValueListenableBuilder - The Digital Gate Monitor]:  │
│  Flight 402 changes gates. A small LCD screen above    │
│  Gate 12 refreshes its display from "12" to "14".      │
│  The rest of the airport continues operating without   │
│  interruption.                                         │
└────────────────────────────────────────────────────────┘
```

Think of a clock on the wall of a library:
- When the second hand ticks, you don't demolish the library and build a new one.
- You don't even ask the people reading books to look up.
- Only the physical gear turning the second hand moves. The building, bookshelves, and readers remain undisturbed.

`ValueListenableBuilder` is the clock on the wall: only its hands move when the value changes.

---

## 🧠 Internal Working: The Framework Implementation

How does `ValueListenableBuilder` work under the hood?

Let's look at a simplified excerpt of Flutter's `widgets/value_listenable_builder.dart`:

```dart
class ValueListenableBuilder<T> extends StatefulWidget {
  const ValueListenableBuilder({
    super.key,
    required this.valueListenable,
    required this.builder,
    this.child,
  });

  final ValueListenable<T> valueListenable;
  final ValueWidgetBuilder<T> builder;
  final Widget? child;

  @override
  State<StatefulWidget> createState() => _ValueListenableBuilderState<T>();
}

class _ValueListenableBuilderState<T> extends State<ValueListenableBuilder<T>> {
  late T value;

  @override
  void initState() {
    super.initState();
    value = widget.valueListenable.value;
    // 1. ATTACH LISTENER
    widget.valueListenable.addListener(_valueChanged);
  }

  @override
  void didUpdateWidget(ValueListenableBuilder<T> oldWidget) {
    super.didUpdateWidget(oldWidget);
    // 2. REBIND IF NOTIFIER INSTANCE CHANGED
    if (oldWidget.valueListenable != widget.valueListenable) {
      oldWidget.valueListenable.removeListener(_valueChanged);
      value = widget.valueListenable.value;
      widget.valueListenable.addListener(_valueChanged);
    }
  }

  @override
  void dispose() {
    // 3. CLEAN UP LISTENER AUTOMATICALLY
    widget.valueListenable.removeListener(_valueChanged);
    super.dispose();
  }

  void _valueChanged() {
    // 4. REBUILD ONLY THIS WIDGET!
    setState(() {
      value = widget.valueListenable.value;
    });
  }

  @override
  Widget build(BuildContext context) {
    // 5. PASS PRE-BUILT CHILD TO BUILDER
    return widget.builder(context, value, widget.child);
  }
}
```

### Key Framework Discoveries:
1. **It is a self-contained `StatefulWidget`**: You don't need to write stateful lifecycle boilerplate because `ValueListenableBuilder` encapsulates `initState`, `didUpdateWidget`, and `dispose` internally!
2. **Automatic Lifecycle Cleanup**: When `ValueListenableBuilder` unmounts, it removes its internal listener in `dispose()`.
3. **The `child` Parameter**: Notice line 47: `widget.builder(context, value, widget.child)`. The `child` is passed directly into the builder function. If you supply an expensive widget subtree as `child`, it is created **outside** the builder and simply slotted into the layout without re-executing its own build method!

---

## ⚙ Syntax

```dart
ValueListenableBuilder<T>(
  valueListenable: myValueNotifier, // 1. The ValueNotifier or ValueListenable
  builder: (BuildContext context, T value, Widget? child) {
    // 2. Returns UI that depends on `value`
    return Row(
      children: [
        Text('Value: $value'),
        if (child != null) child, // 3. Reuses pre-built child!
      ],
    );
  },
  child: const ExpensiveStaticWidget(), // 4. Built ONCE, never rebuilt!
)
```

### Explaining the 3 Parameters of the `builder` Callback:
- **`BuildContext context`**: The standard location of this widget in the element tree.
- **`T value`**: The strongly-typed current value of the listenable at the moment of building.
- **`Widget? child`**: The optional pre-constructed widget passed to the `child` argument of `ValueListenableBuilder`.

---

## 📋 Important Properties Table

| Property | Type | Mandatory? | Description |
| :--- | :--- | :---: | :--- |
| **`valueListenable`** | `ValueListenable<T>` | **Yes** | The source of truth (e.g., `ValueNotifier<T>`). |
| **`builder`** | `ValueWidgetBuilder<T>` | **Yes** | The callback that returns the widget hierarchy whenever `value` updates. |
| **`child`** | `Widget?` | No | An optional static subtree that does not depend on `value`. Passed to `builder` to prevent unnecessary rebuilds. |

---

## 💻 Complete Runnable Example

The following example builds a **Battery Health & Charging Monitor**:
- Maintains two `ValueNotifier`s: `_batteryLevel` (0–100) and `_isCharging` (bool).
- Uses `ValueListenableBuilder` to render the battery icon, percentage label, and charging status badge.
- Demonstrates the **`child` optimization**: An expensive custom decorative card background is passed to the `child` argument, proving that static decorations are never rebuilt when the percentage changes.

```dart
import 'dart:async';
import 'package:flutter/material.dart';

void main() {
  runApp(const ValueListenableMasteryApp());
}

class ValueListenableMasteryApp extends StatelessWidget {
  const ValueListenableMasteryApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'ValueListenableBuilder Mastery',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorSchemeSeed: Colors.teal,
        useMaterial3: true,
      ),
      home: const BatteryMonitorScreen(),
    );
  }
}

class BatteryMonitorScreen extends StatefulWidget {
  const BatteryMonitorScreen({super.key});

  @override
  State<BatteryMonitorScreen> createState() => _BatteryMonitorScreenState();
}

class _BatteryMonitorScreenState extends State<BatteryMonitorScreen> {
  // 1. TWO INDEPENDENT VALUE NOTIFIERS
  final ValueNotifier<int> _batteryLevel = ValueNotifier<int>(65);
  final ValueNotifier<bool> _isCharging = ValueNotifier<bool>(false);
  Timer? _simulationTimer;

  @override
  void dispose() {
    _simulationTimer?.cancel();
    _batteryLevel.dispose();
    _isCharging.dispose();
    super.dispose();
  }

  void _toggleCharging() {
    _isCharging.value = !_isCharging.value;
  }

  void _adjustBattery(int delta) {
    final next = (_batteryLevel.value + delta).clamp(0, 100);
    _batteryLevel.value = next;
  }

  void _startLiveDrainSimulation() {
    if (_simulationTimer != null && _simulationTimer!.isActive) {
      _simulationTimer!.cancel();
      return;
    }

    _simulationTimer = Timer.periodic(const Duration(milliseconds: 400), (timer) {
      if (_batteryLevel.value > 0) {
        _batteryLevel.value--;
      } else {
        timer.cancel();
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Hardware Telemetry Monitor'),
        centerTitle: true,
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(24.0),
        child: Column(
          children: [
            // ================================================================
            // VALUE LISTENBLE BUILDER WITH CHILD OPTIMIZATION
            // ================================================================
            ValueListenableBuilder<int>(
              valueListenable: _batteryLevel,
              // The expensive static container is passed to `child`
              child: const Padding(
                padding: EdgeInsets.only(top: 8.0),
                child: Text(
                  'Lithium-Ion 4500mAh • Optimal Temperature 28°C',
                  style: TextStyle(color: Colors.grey, fontSize: 12),
                ),
              ),
              builder: (BuildContext context, int level, Widget? staticChild) {
                // Determine color based on level
                final Color barColor = level > 50
                    ? Colors.green
                    : level > 20
                        ? Colors.amber
                        : Colors.red;

                return Card(
                  elevation: 4,
                  shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(20)),
                  child: Padding(
                    padding: const EdgeInsets.all(24.0),
                    child: Column(
                      children: [
                        Row(
                          mainAxisAlignment: MainAxisAlignment.spaceBetween,
                          children: [
                            Text(
                              '$level%',
                              style: TextStyle(
                                fontSize: 44,
                                fontWeight: FontWeight.bold,
                                color: barColor,
                              ),
                            ),
                            // Nested builder listening to charging flag
                            ValueListenableBuilder<bool>(
                              valueListenable: _isCharging,
                              builder: (context, charging, _) {
                                return Chip(
                                  avatar: Icon(
                                    charging ? Icons.bolt : Icons.battery_std,
                                    color: charging ? Colors.amber.shade900 : Colors.grey,
                                  ),
                                  label: Text(charging ? 'Fast Charging' : 'Discharging'),
                                  backgroundColor: charging ? Colors.amber.shade100 : Colors.grey.shade100,
                                );
                              },
                            ),
                          ],
                        ),
                        const SizedBox(height: 20),

                        // Animated visual progress bar
                        ClipRRect(
                          borderRadius: BorderRadius.circular(10),
                          child: LinearProgressIndicator(
                            value: level / 100.0,
                            minHeight: 16,
                            backgroundColor: Colors.grey.shade200,
                            color: barColor,
                          ),
                        ),

                        // REUSING THE STATIC PRE-BUILT CHILD!
                        if (staticChild != null) staticChild,
                      ],
                    ),
                  ),
                );
              },
            ),
            const SizedBox(height: 32),

            // CONTROLS PANEL
            Card(
              elevation: 2,
              child: Padding(
                padding: const EdgeInsets.all(20.0),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.stretch,
                  children: [
                    const Text(
                      'Manual Telemetry Controls',
                      style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
                    ),
                    const SizedBox(height: 16),
                    Row(
                      mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                      children: [
                        FilledButton.tonalIcon(
                          onPressed: () => _adjustBattery(-5),
                          icon: const Icon(Icons.remove),
                          label: const Text('-5%'),
                        ),
                        FilledButton.tonalIcon(
                          onPressed: () => _adjustBattery(5),
                          icon: const Icon(Icons.add),
                          label: const Text('+5%'),
                        ),
                      ],
                    ),
                    const SizedBox(height: 12),
                    FilledButton.icon(
                      onPressed: _toggleCharging,
                      icon: const Icon(Icons.power),
                      label: const Text('Toggle Charger Plug'),
                    ),
                    const SizedBox(height: 8),
                    OutlinedButton.icon(
                      onPressed: _startLiveDrainSimulation,
                      icon: const Icon(Icons.speed),
                      label: const Text('Start/Stop Continuous Drain Test'),
                    ),
                  ],
                ),
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

- **Lines 31–32 (`ValueNotifier` declarations)**:
  - `_batteryLevel`: Holds an integer between 0 and 100.
  - `_isCharging`: Holds a boolean tracking power plug status.
- **Lines 36–42 (`dispose`)**: Always cancel timers and dispose both notifiers to ensure zero memory leaks.
- **Lines 80–87 (`child: const Padding(...)`)**: Notice the **`child` parameter optimization**. This `Text` widget displaying `"Lithium-Ion 4500mAh..."` is static. By passing it to `child`, it is instantiated once and passed as `staticChild` to the builder. It is **never recreated** during the rapid battery level ticks!
- **Lines 88–95 (`builder: (context, level, staticChild)`)**: Receives the strongly-typed `int level`. When `_batteryLevel` changes from 65 to 64, this builder executes, dynamically recalculates `barColor` (Green $\rightarrow$ Amber $\rightarrow$ Red), and renders the new percentage.
- **Lines 111–125 (Nested `ValueListenableBuilder<bool>`)**: Listens to `_isCharging`. Notice how clean composition is: when you toggle charging, only the `Chip` updates. The parent battery percentage card does not rebuild!
- **Lines 142–143 (`if (staticChild != null) staticChild`)**: Renders the pre-built child directly into the column.

---

## 📱 ASCII Phone Screen Preview

```
┌──────────────────────────────────────┐
│     Hardware Telemetry Monitor       │
├──────────────────────────────────────┤
│ ┌──────────────────────────────────┐ │
│ │ 65%             [ ⚡ Fast Charge]│ │  <-- Rebuilt by Level & Charging
│ │                                  │ │
│ │ [████████████████░░░░░░░░░░]     │ │  <-- LinearProgressIndicator
│ │                                  │ │
│ │ Lithium-Ion 4500mAh • Temp 28°C  │ │  <-- Static pre-built child!
│ └──────────────────────────────────┘ │
│                                      │
│ ┌──────────────────────────────────┐ │
│ │ Manual Telemetry Controls        │ │
│ │   [ -5% ]           [ +5% ]      │ │
│ │                                  │ │
│ │ [ 🔌 Toggle Charger Plug       ] │ │
│ │ [ ⏱️ Start/Stop Drain Test     ] │ │
│ └──────────────────────────────────┘ │
└──────────────────────────────────────┘
```

---

## 🌳 Widget Tree

```
BatteryMonitorScreen (StatefulWidget)
 └── Scaffold
      ├── AppBar
      └── SingleChildScrollView
           └── Column
                ├── ValueListenableBuilder<int> (_batteryLevel)
                │    └── Card
                │         └── Column
                │              ├── Row (Text: $level%, Nested Builder)
                │              │    └── ValueListenableBuilder<bool> (_isCharging)
                │              │         └── Chip (Bolt Icon, Text)
                │              ├── LinearProgressIndicator
                │              └── staticChild (Lithium-Ion specs Text)
                └── Card (Manual Controls)
```

---

## ⚙ Flutter Execution Flow

```
User taps "-5%" Button
            │
            ▼
1. `_batteryLevel.value` decrements from 65 to 60
            │
            ▼
2. `ValueNotifier` fires internal `notifyListeners()`
            │
            ▼
3. `_ValueListenableBuilderState._valueChanged()` runs
            │
            ▼
4. Internal `setState()` marks ONLY the `ValueListenableBuilder` dirty
            │
            ▼
5. `builder(context, 60, staticChild)` runs:
   • Progress bar updates to 0.60
   • Text updates to "60%"
   • `staticChild` is reused without re-running build()!
            │
            ▼
6. RenderObject repaints the 60% text and green bar
   (Controls Card and Scaffold remain completely untouched!)
```

---

## ⚠ Common Beginner Mistakes

### 1. Creating the `ValueNotifier` Inside the `build()` Method

❌ **Wrong**:
```dart
@override
Widget build(BuildContext context) {
  final count = ValueNotifier<int>(0); // NEW NOTIFIER ON EVERY REBUILD!
  return ValueListenableBuilder<int>(
    valueListenable: count,
    builder: (context, val, _) => Text('$val'),
  );
}
```

Every time the parent widget rebuilds, a brand new `ValueNotifier` instance is allocated, resetting the value back to 0!

✔ **Correct**: Always declare `ValueNotifier` as a persistent field in your `State` class.

---

### 2. Not Utilizing the `child` Parameter for Expensive Subtrees

❌ **Sub-optimal**:
```dart
ValueListenableBuilder<int>(
  valueListenable: notifier,
  builder: (context, val, _) {
    return Column(
      children: [
        Text('$val'),
        const HugeComplexSvgIllustration(), // REBUILT UNNECESSARILY!
      ],
    );
  },
)
```

✔ **Optimized with `child`**:
```dart
ValueListenableBuilder<int>(
  valueListenable: notifier,
  child: const HugeComplexSvgIllustration(), // Built ONCE!
  builder: (context, val, staticChild) {
    return Column(
      children: [
        Text('$val'),
        staticChild!, // Reused!
      ],
    );
  },
)
```

---

### 3. Nesting Too Many Builders Horizontally

If you have 4 different `ValueNotifier`s and nest 4 `ValueListenableBuilder`s, your indentation becomes deeply nested:

```dart
// Pyramid of Doom
ValueListenableBuilder(
  valueListenable: a,
  builder: (ctx, valA, _) => ValueListenableBuilder(
    valueListenable: b,
    builder: (ctx, valB, _) => ValueListenableBuilder(...)
  )
)
```

✔ **Solution**: For multiple values, either combine them into a single `ChangeNotifier`, or use `Listenable.merge([a, b, c])` with a standard `ListenableBuilder`!

---

## 🚀 Professional Usage: Multi-Notifier Listening with `Listenable.merge`

When you want a widget to rebuild if **either** of two `ValueNotifier`s change, you don't need nested builders. Use `Listenable.merge`:

```dart
class DualSensorDisplay extends StatelessWidget {
  final ValueNotifier<double> tempNotifier;
  final ValueNotifier<double> humidityNotifier;

  const DualSensorDisplay({
    super.key,
    required this.tempNotifier,
    required this.humidityNotifier,
  });

  @override
  Widget build(BuildContext context) {
    // Listens to BOTH notifiers with ZERO nesting!
    return ListenableBuilder(
      listenable: Listenable.merge([tempNotifier, humidityNotifier]),
      builder: (context, _) {
        return Text(
          'Temp: ${tempNotifier.value}°C | Humidity: ${humidityNotifier.value}%',
        );
      },
    );
  }
}
```

---

## 💡 Best Practices Checklist

- [ ] **Specify the generic type**: Always write `ValueListenableBuilder<int>`, not raw `ValueListenableBuilder`.
- [ ] **Leverage the `child` argument** to isolate static, expensive subtrees.
- [ ] **Keep builders as close to the leaf as possible**: Don't wrap an entire `Scaffold` in `ValueListenableBuilder`.
- [ ] **Combine multiple listenables** using `Listenable.merge` instead of nesting 3+ builders.
- [ ] **Always dispose the underlying notifier** in `dispose()`.

---

## 🧪 Try It Yourself

1. Tap **"Start/Stop Continuous Drain Test"** and watch the battery percentage count down rapidly.
2. Notice how silky smooth the animation is because only the percentage label and progress bar are rebuilding!
3. Tap **"Toggle Charger Plug"** during the drain test and confirm that the chip toggles between "Fast Charging" and "Discharging" independently.

---

## 🎯 Challenge

Build a **Real-Time Password Strength Meter**:
- Create a `ValueNotifier<String> passwordNotifier = ValueNotifier('');`.
- Attach a listener to a `TextField(obscureText: true)`.
- Use a `ValueListenableBuilder<String>` to display a password strength bar:
  - Red if length < 6
  - Orange if length >= 6 but no numbers
  - Green if length >= 8 with letters and numbers
- Ensure the `TextField` itself is NOT rebuilt by placing it outside the builder!

---

## ❓ Practice Questions

### Q1. What is `ValueListenableBuilder` in Flutter?
**Answer**: `ValueListenableBuilder<T>` is a built-in widget that listens to a `ValueListenable<T>` (such as `ValueNotifier<T>`) and executes its `builder` callback to rebuild only that specific subtree whenever the value changes.

---

### Q2. What are the three parameters passed to the `builder` callback?
**Answer**:
1. `BuildContext context`: The build context of the widget.
2. `T value`: The current strongly-typed value of the `ValueListenable`.
3. `Widget? child`: The optional pre-built widget passed into `ValueListenableBuilder.child`.

---

### Q3. What is the performance purpose of the `child` parameter in `ValueListenableBuilder`?
**Answer**: The `child` parameter allows you to supply an expensive widget subtree that does not depend on the changing value. Flutter instantiates this subtree once and passes it into the `builder` callback as a ready-made instance, preventing it from being rebuilt when the value changes.

---

### Q4. Does `ValueListenableBuilder` automatically remove its listener when unmounted?
**Answer**: Yes. Under the hood, `ValueListenableBuilder` is a `StatefulWidget` whose `State` object automatically calls `valueListenable.removeListener()` inside its `dispose()` method.

---

### Q5. Can `ValueListenableBuilder` listen to objects other than `ValueNotifier`?
**Answer**: Yes. It can listen to any class that implements the `ValueListenable<T>` interface, such as `Animation<T>` (e.g. `Animation<double>`) or custom implementations of `ValueListenable`.

---

### Q6. What is the difference between `ListenableBuilder` and `ValueListenableBuilder`?
**Answer**: `ListenableBuilder` listens to any generic `Listenable` (like `ChangeNotifier`) and its builder callback has the signature `(context, child)`. `ValueListenableBuilder<T>` specifically listens to a `ValueListenable<T>` and provides the strongly-typed `T value` directly as a parameter: `(context, value, child)`.

---

### Q7. What happens if you reassign `valueNotifier.value` to the exact same value it already holds?
**Answer**: `ValueNotifier`'s setter contains an equality guard (`if (_value == newValue) return;`). If the new value is equal to the old value, no notification is fired, and `ValueListenableBuilder` does not rebuild.

---

### Q8. How do you listen to multiple `ValueNotifier`s simultaneously without nesting builders?
**Answer**: Use `Listenable.merge([notifierA, notifierB])` with a standard `ListenableBuilder`. The builder will execute whenever any of the listed notifiers emit an update.

---

### Q9. Can you use `ValueListenableBuilder` inside a `StatelessWidget`?
**Answer**: Yes! `ValueListenableBuilder` is commonly used inside `StatelessWidget`s to provide reactive updates for local or injected notifiers without needing to convert the entire parent widget to a `StatefulWidget`.

---

### Q10. What is the recommended scope for `ValueListenableBuilder`?
**Answer**: It should be scoped as narrowly as possible around the exact leaf widgets that consume the value (e.g., around a single `Text` or `Icon`), rather than wrapping entire page layouts or parent `Scaffold`s.

---

## 📝 Summary

- **Surgical UI Updates**: Rebuilds only its builder callback when the listened value changes.
- **Strongly Typed**: Provides the current `T value` directly to the `builder`.
- **The `child` Optimization**: Eliminates redundant rebuilds of static, expensive subtrees.
- **Automated Teardown**: Automatically handles listener attachment in `initState` and removal in `dispose`.
- **Pure Flutter Native**: Delivers clean, reactive state management without installing third-party packages.
