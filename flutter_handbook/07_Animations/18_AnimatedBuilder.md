# Introduction

`AnimatedBuilder` is one of the most useful widgets in Flutter animation because it lets you rebuild only the parts of the UI that depend on an animation.

This matters because animations often update only a small piece of the interface. If you rebuild the entire screen every frame, your app may become less efficient and more difficult to maintain.

`AnimatedBuilder` solves this by listening to an animation and rebuilding only the builder output that changes with time.

# Why this animation exists

In a normal Flutter widget, a rebuild can happen for many reasons. But animations need a special pattern: they should update repeatedly without recreating a large widget subtree unnecessarily.

`AnimatedBuilder` exists so you can separate:

- animation timing
- rendering logic
- widget subtree that should be rebuilt

This gives you a cleaner and more efficient setup.

# Real-world analogy

Imagine a factory line where only one machine needs updating while the rest of the line continues without interruption.

If the entire factory stops for every tiny change, the work slows down. `AnimatedBuilder` is like updating only the machine that needs the new value, while the rest remains stable.

# Internal Working

`AnimatedBuilder` listens to an animation object and rebuilds its builder closure whenever the animation value changes.

The general pattern is:

```
AnimationController / Animation -> AnimatedBuilder -> builder runs each frame -> UI updates
```

The important detail is that the `builder` is called repeatedly with the latest animation value, and the rest of the widget tree can remain static.

This keeps your app efficient and makes animation code easier to read.

# Syntax

```dart
AnimatedBuilder(
  animation: _controller,
  builder: (context, child) {
    return Transform.rotate(
      angle: _controller.value * 2 * 3.14,
      child: child,
    );
  },
  child: const FlutterLogo(size: 120),
)
```

Key pieces:

- `animation`: the animation object being listened to
- `builder`: runs as the animation value changes
- `child`: static content passed down to avoid rebuilding it every frame

# Complete Example 1

```dart
import 'package:flutter/material.dart';

void main() => runApp(const AnimatedBuilderDemo());

class AnimatedBuilderDemo extends StatefulWidget {
  const AnimatedBuilderDemo({super.key});

  @override
  State<AnimatedBuilderDemo> createState() => _AnimatedBuilderDemoState();
}

class _AnimatedBuilderDemoState extends State<AnimatedBuilderDemo>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 2),
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
        appBar: AppBar(title: const Text('AnimatedBuilder')),
        body: Center(
          child: AnimatedBuilder(
            animation: _controller,
            builder: (context, child) {
              return Transform.rotate(
                angle: _controller.value * 2 * 3.14159,
                child: child,
              );
            },
            child: const FlutterLogo(size: 160),
          ),
        ),
      ),
    );
  }
}
```

## Line-by-line explanation

- `AnimationController` is created with a 2-second duration.
- `..repeat()` keeps the animation running in a loop.
- `AnimatedBuilder` listens to the controller.
- every tick, the builder runs with the new controller value.
- `Transform.rotate` uses the current value to rotate the child.
- `child` is the static logo widget, so it does not rebuild unnecessarily.

## Widget tree

```text
MaterialApp
└── Scaffold
    ├── AppBar
    └── Center
        └── AnimatedBuilder
            └── Transform.rotate
                └── FlutterLogo
```

## Execution flow

```text
Controller repeats over time
   |
   v
AnimatedBuilder receives new animation value
   |
   v
Transform.rotate updates the rotation angle
   |
   v
Flutter paints the newly rotated frame
```

## ASCII phone preview

```text
┌────────────────────────────┐
authored by : builder      │
│                            │
│         [FlutterLogo]      │
│                            │
│                            │
│                            │
│                            │
│                            │
└────────────────────────────┘
```

## Example explanation

This is a classic animation pattern: the controller updates over time, and the widget uses that value to rotate the logo. The `child` is static, so you avoid unnecessary rebuild work.

## Common mistakes

- building huge widget trees inside the `builder`
- forgetting to attach the animation to the `AnimatedBuilder`
- not passing a `child` when the static subtree is large

## Professional use case

Used when a small animated transform is needed around a stable widget, such as a logo, icon, or card preview.

# Complete Example 2

```dart
import 'package:flutter/material.dart';

void main() => runApp(const AnimatedBuilderScaleDemo());

class AnimatedBuilderScaleDemo extends StatefulWidget {
  const AnimatedBuilderScaleDemo({super.key});

  @override
  State<AnimatedBuilderScaleDemo> createState() => _AnimatedBuilderScaleDemoState();
}

class _AnimatedBuilderScaleDemoState extends State<AnimatedBuilderScaleDemo>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 1000),
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
                child: child,
              );
            },
            child: Container(
              width: 180,
              height: 180,
              color: Colors.orange,
              child: const Center(
                child: Text(
                  'Pulse',
                  style: TextStyle(color: Colors.white, fontSize: 24),
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

This creates a pulse animation using a child that stays static while the scale changes.

# Complete Example 3

```dart
import 'package:flutter/material.dart';

void main() => runApp(const AnimatedBuilderOffsetDemo());

class AnimatedBuilderOffsetDemo extends StatefulWidget {
  const AnimatedBuilderOffsetDemo({super.key});

  @override
  State<AnimatedBuilderOffsetDemo> createState() => _AnimatedBuilderOffsetDemoState();
}

class _AnimatedBuilderOffsetDemoState extends State<AnimatedBuilderOffsetDemo>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 800),
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
              return Transform.translate(
                offset: Offset(0, -30 + _controller.value * 60),
                child: child,
              );
            },
            child: Container(
              width: 150,
              height: 150,
              decoration: const BoxDecoration(
                color: Colors.green,
                borderRadius: BorderRadius.all(Radius.circular(20)),
              ),
            ),
          ),
        ),
      ),
    );
  }
}
```

This moves a card up and down with a smooth oscillating motion.

# Advanced Example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const AnimatedBuilderAdvancedDemo());

class AnimatedBuilderAdvancedDemo extends StatefulWidget {
  const AnimatedBuilderAdvancedDemo({super.key});

  @override
  State<AnimatedBuilderAdvancedDemo> createState() => _AnimatedBuilderAdvancedDemoState();
}

class _AnimatedBuilderAdvancedDemoState extends State<AnimatedBuilderAdvancedDemo>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 2),
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
              final opacity = 0.3 + _controller.value * 0.7;
              final scale = 0.8 + _controller.value * 0.5;
              final angle = _controller.value * 2 * 3.14159;

              return Transform.rotate(
                angle: angle,
                child: Opacity(
                  opacity: opacity,
                  child: Transform.scale(
                    scale: scale,
                    child: child,
                  ),
                ),
              );
            },
            child: Container(
              width: 220,
              height: 220,
              decoration: BoxDecoration(
                color: Colors.indigo,
                borderRadius: BorderRadius.circular(30),
              ),
              child: const Center(
                child: Text(
                  'Live motion',
                  style: TextStyle(color: Colors.white, fontSize: 26),
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

This combines multiple animated properties while keeping the child subtree stable.

# Under the Hood

`AnimatedBuilder` is a separation-of-concerns tool. It allows the animation state to remain at the animation layer while the UI rendering stays in the builder.

This is why it is so efficient: the expensive child is not rebuilt when the animation value changes unless the builder says so. The builder can transform the child around the current animation value.

# Performance Notes

- use a static `child` when the subtree is expensive
- avoid computing heavy logic inside the `builder` every frame
- keep the builder focused on only the animated part

# Best Practices

- use `AnimatedBuilder` when you need custom transforms but want to avoid unnecessary rebuilds
- pass static children whenever possible
- keep animation logic separate from the UI structure

# Common Mistakes

- rebuilding too much inside the builder
- forgetting the `child` property for heavy widgets
- mixing large layout logic into the animation closure

# Debugging

If the animation is wrong:

1. confirm the builder is listening to the correct animation
2. confirm the value is changing as expected
3. inspect whether the child is static or being rebuilt unnecessarily
4. verify the controller is started and not disposed early

# Interview Questions

1. What is the purpose of `AnimatedBuilder`?
2. Why is it more efficient than rebuilding a large widget tree?
3. What is the role of the `child` parameter?
4. How does it differ from `setState`-driven animation?
5. When would you choose `AnimatedBuilder`?
6. Why is it useful in custom motion design?
7. What does the builder receive?
8. How does the static child help performance?
9. What is a practical case where it is used?
10. How does it help separation of concerns?

# Try It Yourself

Create an animation where a card rotates and scales using `AnimatedBuilder`, then move the expensive static widget into the `child` parameter and observe the code structure.

# Challenge Project

Build a custom motion card where the child remains static and only the transform changes over time using `AnimatedBuilder`.

# Quick Revision Sheet

- `AnimatedBuilder` listens to an animation and rebuilds the builder output
- it reduces unnecessary rebuilds
- the `child` parameter keeps static content efficient
- ideal for custom transforms, rotation, scale, and position
- this is a professional way to build efficient animations in Flutter
