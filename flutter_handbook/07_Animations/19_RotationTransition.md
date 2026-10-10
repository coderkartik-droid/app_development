# Introduction

`RotationTransition` is a Flutter widget that rotates a child over time according to an animation.

It is often used for:

- loading indicators
- refresh states
- notification icons
- feature toggles
- menu action controls

It is simple, expressive, and widely used in UI design because rotation is easy to understand visually.

# Why this animation exists

Some actions feel more complete when the user sees a visual turn or spin. A rotated icon can communicate progress or change of state without extra text.

A smooth rotation says, "This is active, progressing, or changing mode." It is a simple but powerful way to communicate state.

# Real-world analogy

Imagine a spinner wheel or a mechanical dial rotating to indicate operation. The user sees motion and understands that activity is underway.

`RotationTransition` gives your widget a similar communication language.

# Internal Working

`RotationTransition` takes an animation object that defines the rotation angle. By default, the child rotates around its center.

The value usually ranges between 0.0 and 1.0, scaled into radians when used with the transformation.

A common pattern is:

```
AnimationController -> RotationTransition -> child rotates according to animation value
```

The widget does not create the controller for you; you supply the animation and decide when it starts, stops, or repeats.

# Syntax

```dart
RotationTransition(
  turns: _animation,
  child: const Icon(Icons.refresh),
)
```

Important pieces:

- `turns`: the animation value controlling rotation
- `child`: the widget that will rotate
- `alignment`: optional center alignment

# Complete Example 1

```dart
import 'package:flutter/material.dart';

void main() => runApp(const RotationTransitionDemo());

class RotationTransitionDemo extends StatefulWidget {
  const RotationTransitionDemo({super.key});

  @override
  State<RotationTransitionDemo> createState() => _RotationTransitionDemoState();
}

class _RotationTransitionDemoState extends State<RotationTransitionDemo>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;
  late final Animation<double> _turns;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 2),
    )..repeat();
    _turns = CurvedAnimation(
      parent: _controller,
      curve: Curves.linear,
    );
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
        appBar: AppBar(title: const Text('RotationTransition')),
        body: Center(
          child: RotationTransition(
            turns: _turns,
            child: const Icon(Icons.refresh, size: 90),
          ),
        ),
      ),
    );
  }
}
```

## Line-by-line explanation

- `_controller` provides the time progression.
- `CurvedAnimation` with `Curves.linear` means the turn value increases evenly.
- `RotationTransition` uses the animation to rotate the icon.
- `..repeat()` keeps the animation moving continuously.

## Widget tree

```text
MaterialApp
└── Scaffold
    ├── AppBar
    └── Center
        └── RotationTransition
            └── Icon
```

## Execution flow

```text
Controller updates over time
   |
   v
RotationTransition receives turn value
   |
   v
Widget rotates around center
   |
   v
Flutter repaints each frame with new angle
```

## ASCII phone preview

```text
┌────────────────────────────┐
authored by : rotate       │
│                            │
│          [⟳]               │
│                            │
│                            │
│                            │
│                            │
│                            │
└────────────────────────────┘
```

## Example explanation

This is a loading-style spinner. The icon rotates continuously, clearly communicating that an action is underway or the app is processing data.

## Common mistakes

- forgetting to provide a valid animation controller
- setting the wrong `turns` value range
- not repeating the animation when continuous motion is desired

## Professional use case

Used for refresh behaviors, sync states, processing indicators, and utility icons that need to indicate activity.

# Complete Example 2

```dart
import 'package:flutter/material.dart';

void main() => runApp(const RotationButtonApp());

class RotationButtonApp extends StatefulWidget {
  const RotationButtonApp({super.key});

  @override
  State<RotationButtonApp> createState() => _RotationButtonAppState();
}

class _RotationButtonAppState extends State<RotationButtonApp>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;
  late final Animation<double> _turns;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 800),
    );
    _turns = CurvedAnimation(
      parent: _controller,
      curve: Curves.easeInOut,
    );
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
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              RotationTransition(
                turns: _turns,
                child: const Icon(Icons.settings, size: 90),
              ),
              const SizedBox(height: 20),
              ElevatedButton(
                onPressed: () => _controller.forward(from: 0),
                child: const Text('Rotate once'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

This rotates the settings icon once when the user presses the button.

# Complete Example 3

```dart
import 'package:flutter/material.dart';

void main() => runApp(const RotationPickupApp());

class RotationPickupApp extends StatefulWidget {
  const RotationPickupApp({super.key});

  @override
  State<RotationPickupApp> createState() => _RotationPickupAppState();
}

class _RotationPickupAppState extends State<RotationPickupApp>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;
  late final Animation<double> _turns;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 700),
    )..repeat(reverse: true);
    _turns = CurvedAnimation(
      parent: _controller,
      curve: Curves.easeInOut,
    );
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
          child: RotationTransition(
            turns: _turns,
            child: Container(
              width: 180,
              height: 180,
              decoration: BoxDecoration(
                color: Colors.blue[100],
                borderRadius: BorderRadius.circular(30),
              ),
              child: const Center(
                child: Icon(Icons.sync, size: 70),
              ),
            ),
          ),
        ),
      ),
    );
  }
}
```

This creates a subtle sync icon rotation within a container.

# Advanced Example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const RotationHeroApp());

class RotationHeroApp extends StatefulWidget {
  const RotationHeroApp({super.key});

  @override
  State<RotationHeroApp> createState() => _RotationHeroAppState();
}

class _RotationHeroAppState extends State<RotationHeroApp>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;
  late final Animation<double> _turns;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 3),
    )..repeat();
    _turns = CurvedAnimation(
      parent: _controller,
      curve: Curves.easeInOutCubic,
    );
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
          child: RotationTransition(
            turns: _turns,
            child: Container(
              width: 240,
              height: 240,
              decoration: BoxDecoration(
                gradient: const LinearGradient(
                  colors: [Colors.purple, Colors.red],
                ),
                borderRadius: BorderRadius.circular(30),
              ),
              child: const Center(
                child: Text(
                  'Processing',
                  style: TextStyle(color: Colors.white, fontSize: 28),
                ),
              ),
            ),
          ),
        ),
      ),
    );
  }
}
```

This is a more visually rich processing indicator with a gradient and rotation.

# Under the Hood

`RotationTransition` is a direct wrapper around a rotational transform. It uses the supplied animation as an angle factor and rotates the child according to that value.

The typical unit for turns is one full rotation = `1.0`. So if the animation is 0.5, the widget rotates half a turn.

# Performance Notes

- rotation is lightweight compared to complex rebuilds
- use it for smaller visual details rather than huge tree rotations
- avoid unnecessary repetitive animations on content-heavy screens

# Best Practices

- use rotation for status or progress states, not for content gimmicks
- keep the rotation consistent and visible
- combine with color or opacity for bigger clarity

# Common Mistakes

- using a large repeated rotation for decorative-only content
- forgetting to supply a valid controller or animation
- using a rotation where a fade or slide might be more appropriate

# Debugging

If the icon is not rotating:

1. check whether the controller is running
2. confirm the animation is assigned to `turns`
3. verify `forward` or `repeat` is called
4. inspect whether the widget is not rebuilt due to the animation not being listened to

# Interview Questions

1. What does `RotationTransition` do?
2. Why is the `turns` value important?
3. What does a value of `1.0` mean?
4. Why is rotation effective for loading states?
5. What is the difference between rotation and scale?
6. How does this widget use an animation?
7. Why do we usually use `repeat()` for a spinner?
8. When is a reverse animation useful?
9. Why would a developer prefer this over manual transforms?
10. Which app interactions commonly use rotation?

# Try It Yourself

Create a refresh button that rotates once and then settles. Try the same animation with a curve to see how the motion feels.

# Challenge Project

Build a circular loading screen with a rotating icon and a status label that changes when the process completes.

# Quick Revision Sheet

- `RotationTransition` rotates a child based on an animation
- `turns` is the angle controller, where `1.0` is a full turn
- ideal for progress, settings, and refresh states
- works best when motion clearly communicates action
- simple, readable, and easy to combine with other animations
