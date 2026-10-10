# Introduction

`AnimationController` is one of the most important classes in Flutter animation.

It is the main object that drives animation timing. It manages the progression of a value over time, usually between 0.0 and 1.0.

A controller can:

- start an animation
- stop it
- reverse it
- repeat it
- drive custom curves and tweens

It is the foundation for explicit animation in Flutter.

# Why this animation exists

A widget cannot move smoothly if there is no time-based value. `AnimationController` gives the animation a timeline.

It is the heart of explicit animation because it decides when the animation is active and how far it has progressed.

# Real-world analogy

Think of a clock.

A clock measures time and helps you know how far through a process you are. The animation controller does the same: it measures how far the animation has moved through its timeline.

# Internal Working

`AnimationController` is attached to a `Ticker`, and the ticker calls back repeatedly while the animation runs.

Typical flow:

```
AnimationController created
   |
   v
forward(), reverse(), or repeat()
   |
   v
Ticker fires at each frame
   |
   v
value updates from 0.0 to 1.0 or vice versa
   |
   v
widgets rebuild with current animation value
```

This process is what makes smooth motion possible.

# Syntax

```dart
late final AnimationController _controller;

_controller = AnimationController(
  vsync: this,
  duration: const Duration(seconds: 1),
);

_controller.forward();
```

Common methods:

- `forward()` moves forward
- `reverse()` moves backward
- `repeat()` loops continuously
- `stop()` halts the animation
- `reset()` sets it back to the start
- `dispose()` cleans it up

# Complete Example 1

```dart
import 'package:flutter/material.dart';

void main() => runApp(const AnimationControllerDemo());

class AnimationControllerDemo extends StatefulWidget {
  const AnimationControllerDemo({super.key});

  @override
  State<AnimationControllerDemo> createState() => _AnimationControllerDemoState();
}

class _AnimationControllerDemoState extends State<AnimationControllerDemo>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 900),
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
        appBar: AppBar(title: const Text('AnimationController')),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              AnimatedBuilder(
                animation: _controller,
                builder: (context, child) {
                  return Opacity(
                    opacity: _controller.value,
                    child: Container(
                      width: 200,
                      height: 200,
                      color: Colors.orange,
                    ),
                  );
                },
              ),
              const SizedBox(height: 24),
              ElevatedButton(
                onPressed: () => _controller.forward(from: 0.0),
                child: const Text('Start'),
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

- `SingleTickerProviderStateMixin` gives the widget a ticker.
- `AnimationController` is created with a duration.
- `AnimatedBuilder` listens to the controller values.
- `Opacity` uses `controller.value` to fade the box.
- `forward(from: 0.0)` starts the animation from the beginning.

## Widget tree

```text
MaterialApp
└── Scaffold
    ├── AppBar
    └── Center
        └── Column
            ├── AnimatedBuilder
            │   └── Opacity
            │       └── Container
            └── ElevatedButton
```

## Execution flow

```text
Controller created in initState
   |
   v
Button tapped
   |
   v
forward(from: 0.0)
   |
   v
Ticker updates value over time
   |
   v
Opacity and render change smoothly
```

## ASCII phone preview

```text
┌────────────────────────────┐
authored by : controller   │
│                            │
│        [orange box]       │
│                            │
│        [Start]            │
│                            │
│                            │
│                            │
└────────────────────────────┘
```

## Example explanation

This example clearly shows that the controller is the time driver. It is not the visual effect itself—it creates the changing value that the widget uses to render the animation.

## Common mistakes

- forgetting `vsync`
- forgetting to dispose the controller
- creating a controller but never starting it

## Professional use case

This is the foundation of custom animations, list item movement, interactive transitions, and complex UI logic.

# Complete Example 2

```dart
import 'package:flutter/material.dart';

void main() => runApp(const RepeatControllerApp());

class RepeatControllerApp extends StatefulWidget {
  const RepeatControllerApp({super.key});

  @override
  State<RepeatControllerApp> createState() => _RepeatControllerAppState();
}

class _RepeatControllerAppState extends State<RepeatControllerApp>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 1),
    )..repeat();
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
              return Transform.rotate(
                angle: _controller.value * 2 * 3.14159,
                child: const FlutterLogo(size: 120),
              );
            },
          ),
        ),
      ),
    );
  }
}
```

This creates a rotating logo by repeating the controller continuously.

# Complete Example 3

```dart
import 'package:flutter/material.dart';

void main() => runApp(const ReverseControllerApp());

class ReverseControllerApp extends StatefulWidget {
  const ReverseControllerApp({super.key});

  @override
  State<ReverseControllerApp> createState() => _ReverseControllerAppState();
}

class _ReverseControllerAppState extends State<ReverseControllerApp>
    with SingleTickerProviderStateMixin {
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

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              AnimatedBuilder(
                animation: _controller,
                builder: (context, child) {
                  return Transform.scale(
                    scale: 0.5 + _controller.value * 1.0,
                    child: Container(
                      width: 150,
                      height: 150,
                      color: Colors.green,
                    ),
                  );
                },
              ),
              const SizedBox(height: 18),
              ElevatedButton(
                onPressed: () {
                  if (_controller.isCompleted) {
                    _controller.reverse();
                  } else {
                    _controller.forward();
                  }
                },
                child: const Text('Toggle direction'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

This shows controlling both forward and reverse motion.

# Advanced Example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const AdvancedControllerApp());

class AdvancedControllerApp extends StatefulWidget {
  const AdvancedControllerApp({super.key});

  @override
  State<AdvancedControllerApp> createState() => _AdvancedControllerAppState();
}

class _AdvancedControllerAppState extends State<AdvancedControllerApp>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 1200),
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
              AnimatedBuilder(
                animation: _controller,
                builder: (context, child) {
                  final value = Curves.easeInOut.transform(_controller.value);
                  return Transform.translate(
                    offset: Offset(0, (1 - value) * 80),
                    child: Opacity(
                      opacity: value,
                      child: Container(
                        width: 200,
                        height: 80,
                        color: Colors.indigo,
                        child: const Center(
                          child: Text(
                            'Animated',
                            style: TextStyle(color: Colors.white, fontSize: 22),
                          ),
                        ),
                      ),
                    ),
                  );
                },
              ),
              const SizedBox(height: 20),
              ElevatedButton(
                onPressed: () => _controller.forward(from: 0.0),
                child: const Text('Run again'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

This shows the controller driving a more nuanced motion using a curve and offset.

# Under the Hood

The controller does not visually render anything itself. It provides the time-based value that your widgets consume. The real visuals come from `Transform`, `Opacity`, gradients, and other widgets that use the controller value to update their appearance.

This is why controller-driven animation is so powerful: the values are simple, but they are incredibly flexible.

# Performance Notes

- avoid heavy work inside every animation frame
- keep controllers to the widgets that actually need them
- dispose them to prevent leaks and unnecessary updates

# Best Practices

- keep controllers created only when needed
- prefer explicit controllers for custom sequences
- use `forward`, `reverse`, and `repeat` deliberately
- remember that values are usually from 0.0 to 1.0

# Common Mistakes

- forgetting `SingleTickerProviderStateMixin`
- not disposing the controller
- changing animation values without using them in the UI

# Debugging

If the animation does not run:

1. check whether `vsync` is provided
2. ensure a method like `forward()` or `repeat()` is called
3. verify the UI consumes `controller.value`
4. inspect if the controller was disposed too early

# Interview Questions

1. What is the role of `AnimationController`?
2. Why is `vsync` required?
3. What is the default range of animation values?
4. What do `forward`, `reverse`, and `repeat` do?
5. Why is it called an explicit animation system?
6. What is the relationship between the controller and the ticker?
7. What is the consequence of not disposing a controller?
8. What does `value` represent?
9. How is a controller useful for custom motion sequences?
10. How does it connect to `Tween` and `CurvedAnimation`?

# Try It Yourself

Create a button that triggers a controller to animate a box from small to big and back. Use `forward` and `reverse` to understand the controller lifecycle.

# Challenge Project

Build a custom animated modal that opens and closes using a controller-driven scale and fade combination.

# Quick Revision Sheet

- `AnimationController` gives the animation its timeline
- `vsync` ties it to the widget lifecycle
- `value` is usually between 0.0 and 1.0
- `forward`, `reverse`, `repeat`, and `stop` drive the motion
- it is the base of explicit animations
- the controller itself does not draw; widgets use it to render
