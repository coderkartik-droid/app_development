# Introduction

Flutter gives two major animation styles: implicit and explicit.

The difference is simple:

- implicit animation: you tell Flutter what the final property should be, and Flutter animates it automatically
- explicit animation: you control the timing and animation behavior yourself

A beginner often starts with implicit animations because they are easy to understand. As the app becomes more custom, explicit animations become more useful.

# Why this animation exists

Flutter provides both approaches because different situations need different control.

If you need a box to smoothly move from one color to another when a flag changes, implicit animation is perfect. If you need a custom rotation, bounce, or layered motion, an explicit animation gives you more power.

The tool choice depends on the amount of control you need.

# Real-world analogy

Imagine a driver using a car with automatic mode vs a manual transmission.

- automatic mode: the car handles the details
- manual mode: the driver controls each change directly

Implicit animations are automatic transforms. Explicit animations are manual control of movement over time.

# Internal Working

Implicit animations are built using widgets like `AnimatedContainer`, `AnimatedOpacity`, and `AnimatedAlign`.

These widgets internally:

- track a value
- compare old and new states
- animate between them
- rebuild the UI smoothly

Explicit animations use an `AnimationController` and custom animation values. They give you a direct handle over timing, curves, and triggers.

The main difference is ownership of the animation lifecycle:

- implicit: Flutter owns and manages the animation internally
- explicit: you own the controller and timing

# Syntax

## Implicit syntax

```dart
AnimatedContainer(
  duration: const Duration(milliseconds: 500),
  width: 200,
  height: 100,
  color: Colors.blue,
)
```

## Explicit syntax

```dart
AnimationController controller = AnimationController(
  vsync: this,
  duration: const Duration(seconds: 1),
);

CurvedAnimation(
  parent: controller,
  curve: Curves.easeInOut,
);
```

# Complete Example 1

```dart
import 'package:flutter/material.dart';

void main() => runApp(const ImplicitVsExplicitApp());

class ImplicitVsExplicitApp extends StatefulWidget {
  const ImplicitVsExplicitApp({super.key});

  @override
  State<ImplicitVsExplicitApp> createState() => _ImplicitVsExplicitAppState();
}

class _ImplicitVsExplicitAppState extends State<ImplicitVsExplicitApp>
    with SingleTickerProviderStateMixin {
  bool _expanded = false;
  late final AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 600),
    );
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  void _toggle() {
    setState(() {
      _expanded = !_expanded;
    });
    if (_expanded) {
      _controller.forward();
    } else {
      _controller.reverse();
    }
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('Implicit vs Explicit')),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              AnimatedContainer(
                duration: const Duration(milliseconds: 500),
                curve: Curves.easeInOut,
                width: _expanded ? 220 : 120,
                height: _expanded ? 220 : 120,
                decoration: BoxDecoration(
                  color: _expanded ? Colors.green : Colors.blue,
                  borderRadius: BorderRadius.circular(_expanded ? 30 : 12),
                ),
              ),
              const SizedBox(height: 24),
              ScaleTransition(
                scale: _controller,
                child: const Text(
                  'Explicit animation is also active',
                  style: TextStyle(fontSize: 18),
                ),
              ),
              const SizedBox(height: 24),
              ElevatedButton(
                onPressed: _toggle,
                child: const Text('Toggle'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

## Line-by-line explanation

- `AnimatedContainer` uses implicit animation.
- `width`, `height`, `color`, and radius all animate automatically when the state changes.
- `_controller` drives an explicit scale animation.
- `ScaleTransition` listens to the controller and smoothly scales the text.
- `setState` updates the expansion state.
- `forward()` and `reverse()` trigger explicit timing.

## Widget tree

```text
MaterialApp
└── Scaffold
    ├── AppBar
    └── Center
        └── Column
            ├── AnimatedContainer
            ├── ScaleTransition
            │   └── Text
            └── ElevatedButton
```

## Execution flow

```text
Button tapped
   |
   v
Expanded state toggles
   |
   v
AnimatedContainer animates to new values
   |
   v
Controller triggers ScaleTransition
   |
   v
UI shows smooth state change
```

## ASCII phone preview

```text
┌────────────────────────────┐
authored by : compare      │
│                            │
│   [   box   ]               │
│                            │
│ Explicit animation is      │
│ also active                │
│                            │
│          [Toggle]          │
│                            │
│                            │
└────────────────────────────┘
```

## Example explanation

This example mixes both styles. The card is animated implicitly because its shape, size, and color change automatically. The text is animated explicitly because it uses a controller and scale transition. The combination shows why both systems are useful.

## Common mistakes

- using explicit animation when implicit animation would be simpler
- forgetting `vsync` in explicit animations
- expecting implicit widgets to support custom curves without specifying them

## Professional use case

This pattern is common in complex screens where one widget changes automatically but another needs custom motion or repetition.

# Complete Example 2

```dart
import 'package:flutter/material.dart';

void main() => runApp(const ImplicitOnlyApp());

class ImplicitOnlyApp extends StatefulWidget {
  const ImplicitOnlyApp({super.key});

  @override
  State<ImplicitOnlyApp> createState() => _ImplicitOnlyAppState();
}

class _ImplicitOnlyAppState extends State<ImplicitOnlyApp> {
  bool _selected = false;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        body: Center(
          child: GestureDetector(
            onTap: () => setState(() => _selected = !_selected),
            child: AnimatedContainer(
              duration: const Duration(milliseconds: 600),
              curve: Curves.easeInOutBack,
              width: _selected ? 200 : 120,
              height: _selected ? 200 : 120,
              decoration: BoxDecoration(
                color: _selected ? Colors.red : Colors.blue,
                borderRadius: BorderRadius.circular(_selected ? 30 : 10),
                boxShadow: [
                  BoxShadow(
                    color: Colors.black.withOpacity(0.2),
                    blurRadius: 14,
                    offset: const Offset(0, 8),
                  ),
                ],
              ),
            ),
          ),
        ),
      ),
    );
  }
}
```

This is an implicit animation example: changing a single state triggers smooth property change.

# Complete Example 3

```dart
import 'package:flutter/material.dart';

void main() => runApp(const ExplicitOnlyApp());

class ExplicitOnlyApp extends StatefulWidget {
  const ExplicitOnlyApp({super.key});

  @override
  State<ExplicitOnlyApp> createState() => _ExplicitOnlyAppState();
}

class _ExplicitOnlyAppState extends State<ExplicitOnlyApp>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 700),
    )..repeat(reverse: true);
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        body: Center(
          child: AnimatedBuilder(
            animation: _controller,
            builder: (context, child) {
              return Transform.scale(
                scale: 0.7 + _controller.value * 0.6,
                child: Container(
                  width: 150,
                  height: 150,
                  decoration: const BoxDecoration(
                    shape: BoxShape.circle,
                    color: Colors.orange,
                  ),
                ),
              );
            },
          ),
        ),
      ),
    );
  }
}
```

This is the explicit path: custom value mapping and custom motion logic.

# Advanced Example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const AdvancedComparisonApp());

class AdvancedComparisonApp extends StatefulWidget {
  const AdvancedComparisonApp({super.key});

  @override
  State<AdvancedComparisonApp> createState() => _AdvancedComparisonAppState();
}

class _AdvancedComparisonAppState extends State<AdvancedComparisonApp>
    with SingleTickerProviderStateMixin {
  bool _isOpen = false;
  late final AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 700),
    );
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  void _toggle() {
    setState(() => _isOpen = !_isOpen);
    if (_isOpen) {
      _controller.forward();
    } else {
      _controller.reverse();
    }
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              AnimatedContainer(
                duration: const Duration(milliseconds: 500),
                width: _isOpen ? 260 : 160,
                height: _isOpen ? 180 : 90,
                decoration: BoxDecoration(
                  color: _isOpen ? Colors.teal : Colors.purple,
                  borderRadius: BorderRadius.circular(_isOpen ? 20 : 8),
                ),
              ),
              const SizedBox(height: 20),
              ScaleTransition(
                scale: CurvedAnimation(
                  parent: _controller,
                  curve: Curves.elasticOut,
                ),
                child: const Text('Custom explicit details'),
              ),
              const SizedBox(height: 20),
              ElevatedButton(
                onPressed: _toggle,
                child: const Text('Switch state'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

This combines both systems to show how a practical app can use them together.

# Under the Hood

Implicit animations are a convenience layer built over explicit animation principles. Flutter handles the `AnimationController`, values, and rebuilds behind the scenes.

Explicit animations are more transparent. You create the motion logic and decide exactly how the animation should behave over time.

This is why explicit animation gives more control but also more responsibility.

# Performance Notes

- implicit animations are easier and often more efficient for simple UI changes
- explicit animations are useful when you need custom control
- define animation durations and curves deliberately
- avoid using too many `Animated*` widgets at once

# Best Practices

- use implicit animations for simple property transitions
- use explicit animations when logic is custom or advanced
- keep your curves consistent with your app feel
- group animations when possible for better perceived quality

# Common Mistakes

- assuming explicit animation is always better
- using implicit widgets in situations needing custom behavior
- ignoring `vsync` in explicit animation controllers

# Debugging

If something is not animating:

- check whether the state changed
- verify that the animated property actually changed
- ensure `AnimatedContainer` is replacing a different value
- confirm the `AnimationController` is running and not disposed

# Interview Questions

1. What is the main difference between implicit and explicit animations?
2. Which widget is a common implicit animation example?
3. Why do explicit animations need `vsync`?
4. When would you choose implicit animation?
5. When would you choose explicit animation?
6. How does Flutter manage implicit animations internally?
7. What is the role of `CurvedAnimation` in explicit animation?
8. Why are implicit animations easier for beginners?
9. What is the advantage of custom explicit animation?
10. Can both styles be combined in one screen?

# Try It Yourself

Create two versions of the same card:

- one version using `AnimatedContainer`
- one version using `AnimationController` and `AnimatedBuilder`

Compare the code complexity and flexibility.

# Challenge Project

Build a card that expands when tapped.

Requirements:

- one part should animate implicitly
- one part should animate explicitly
- use a smooth transition and polished spacing

# Quick Revision Sheet

- implicit animation = automatic, easy, ideal for basic property transitions
- explicit animation = manual control, ideal for custom motion
- implicit widgets are simpler but less flexible
- explicit controllers allow timing, curves, reverse, repeat, and custom mapping
- both styles can be combined in a real app
- choose the simpler one that fits the job
