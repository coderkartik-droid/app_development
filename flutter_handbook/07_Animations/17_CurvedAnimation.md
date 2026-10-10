# Introduction

`CurvedAnimation` is used when you want motion to feel natural instead of mechanical.

A straight linear change from 0.0 to 1.0 can feel too rigid. Curves make the motion slower at the start, faster in the middle, or softer at the end.

This is why `CurvedAnimation` is often used in real apps when a movement should feel polished and human.

# Why this animation exists

People do not interact with apps in a perfectly linear way. Movement feels more natural if it follows a curve.

Curves help with:

- slow reveal
- quick lift-in effect
- soft bounce
- elastic return
- controlled deceleration

# Real-world analogy

Think of a ball rolling down a slope.

It does not move at a constant speed because gravity changes its acceleration. The same is true for movements in UI. Curves mimic natural motion patterns.

# Internal Working

`CurvedAnimation` wraps a parent animation and applies a `Curve` to it.

The general flow is:

```dart
controller -> CurvedAnimation -> value with curve -> widget renders
```

The curve is a function that transforms the output of the controller into a different progression.

For example, `Curves.easeInOut` makes the animation slow at the beginning and end but faster in the middle.

# Syntax

```dart
final controller = AnimationController(vsync: this, duration: const Duration(seconds: 1));
final animation = CurvedAnimation(
  parent: controller,
  curve: Curves.easeInOut,
);
```

Common curves:

- `Curves.easeIn`
- `Curves.easeOut`
- `Curves.easeInOut`
- `Curves.bounceOut`
- `Curves.elasticOut`
- `Curves.fastOutSlowIn`

# Complete Example 1

```dart
import 'package:flutter/material.dart';

void main() => runApp(const CurvedAnimationDemo());

class CurvedAnimationDemo extends StatefulWidget {
  const CurvedAnimationDemo({super.key});

  @override
  State<CurvedAnimationDemo> createState() => _CurvedAnimationDemoState();
}

class _CurvedAnimationDemoState extends State<CurvedAnimationDemo>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;
  late final Animation<double> _animation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 900),
    );
    _animation = CurvedAnimation(
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
              AnimatedBuilder(
                animation: _animation,
                builder: (context, child) {
                  return Transform.scale(
                    scale: 0.6 + _animation.value * 0.8,
                    child: Container(
                      width: 160,
                      height: 160,
                      color: Colors.teal,
                    ),
                  );
                },
              ),
              const SizedBox(height: 20),
              ElevatedButton(
                onPressed: () => _controller.forward(from: 0),
                child: const Text('Run curve'),
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

- `CurvedAnimation` wraps the controller.
- `Curves.easeInOut` slows the start and end while keeping motion smooth.
- `AnimatedBuilder` listens to the curve-adjusted animation.
- `Transform.scale` uses the animation value to create a smooth scale effect.

## Widget tree

```text
MaterialApp
└── Scaffold
    └── Center
        └── Column
            ├── AnimatedBuilder
            │   └── Transform.scale
            │       └── Container
            └── ElevatedButton
```

## Execution flow

```text
Button pressed
   |
   v
Controller begins animation
   |
   v
CurvedAnimation shapes the timeline
   |
   v
Widget reads the new value and updates visual scale
```

## ASCII phone preview

```text
┌────────────────────────────┐
authored by : curve       │
│                            │
│      [teal box]            │
│                            │
│        [Run curve]         │
│                            │
│                            │
│                            │
└────────────────────────────┘
```

## Example explanation

Without a curve, the square would grow linearly. With a curve, the growth feels more natural and polished. This can make a big difference in the perceived quality of the interface.

## Common mistakes

- forgetting to wrap the controller in a curve
- using too many heavy curves at once
- expecting random and unnatural motion to feel professional

## Professional use case

Used in modal transitions, card expansions, page entrance motion, and app onboarding animations.

# Complete Example 2

```dart
import 'package:flutter/material.dart';

void main() => runApp(const BounceCurveDemo());

class BounceCurveDemo extends StatefulWidget {
  const BounceCurveDemo({super.key});

  @override
  State<BounceCurveDemo> createState() => _BounceCurveDemoState();
}

class _BounceCurveDemoState extends State<BounceCurveDemo>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;
  late final Animation<double> _animation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 1200),
    );
    _animation = CurvedAnimation(
      parent: _controller,
      curve: Curves.bounceOut,
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
                animation: _animation,
                builder: (context, child) {
                  return Transform.scale(
                    scale: 0.5 + _animation.value * 0.9,
                    child: Container(
                      width: 180,
                      height: 180,
                      color: Colors.orange,
                    ),
                  );
                },
              ),
              const SizedBox(height: 20),
              ElevatedButton(
                onPressed: () => _controller.forward(from: 0),
                child: const Text('Bounce'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

This creates a playful bounce effect using a curve designed for energetic motion.

# Complete Example 3

```dart
import 'package:flutter/material.dart';

void main() => runApp(const ElasticCurveDemo());

class ElasticCurveDemo extends StatefulWidget {
  const ElasticCurveDemo({super.key});

  @override
  State<ElasticCurveDemo> createState() => _ElasticCurveDemoState();
}

class _ElasticCurveDemoState extends State<ElasticCurveDemo>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;
  late final Animation<double> _animation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 1000),
    );
    _animation = CurvedAnimation(
      parent: _controller,
      curve: Curves.elasticOut,
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
          child: AnimatedBuilder(
            animation: _animation,
            builder: (context, child) {
              return Transform.translate(
                offset: Offset(0, (1 - _animation.value) * 90),
                child: Container(
                  width: 180,
                  height: 80,
                  color: Colors.purple,
                ),
              );
            },
          ),
        ),
        floatingActionButton: FloatingActionButton(
          onPressed: () => _controller.forward(from: 0),
          child: const Icon(Icons.play_arrow),
        ),
      ),
    );
  }
}
```

This uses a spring-like elastic curve for a rich movement.

# Advanced Example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const CurveComparisonApp());

class CurveComparisonApp extends StatefulWidget {
  const CurveComparisonApp({super.key});

  @override
  State<CurveComparisonApp> createState() => _CurveComparisonAppState();
}

class _CurveComparisonAppState extends State<CurveComparisonApp>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;
  late final Animation<double> _animation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 800),
    );
    _animation = CurvedAnimation(
      parent: _controller,
      curve: Curves.fastOutSlowIn,
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
          child: AnimatedBuilder(
            animation: _animation,
            builder: (context, child) {
              return Transform.translate(
                offset: Offset(0, (1 - _animation.value) * 80),
                child: Opacity(
                  opacity: _animation.value,
                  child: Container(
                    width: 220,
                    height: 120,
                    color: Colors.indigo,
                    child: const Center(
                      child: Text(
                        'Smooth entrance',
                        style: TextStyle(color: Colors.white, fontSize: 24),
                      ),
                    ),
                  ),
                ),
              );
            },
          ),
        ),
        floatingActionButton: FloatingActionButton(
          onPressed: () => _controller.forward(from: 0),
          child: const Icon(Icons.start),
        ),
      ),
    );
  }
}
```

This is a polished, natural entry animation used in many mobile UI screens.

# Under the Hood

The curve is a mathematical mapping function. The controller provides a value that goes from 0.0 to 1.0; the curve reshapes that value according to the motion behavior you want. For example, a bounce or elastic curve makes the change feel more playful or more physical.

# Performance Notes

- curves are lightweight and useful
- avoid overusing extreme curves that make the animation feel chaotic
- choose a curve that matches the screen's purpose

# Best Practices

- use subtle curves for functional UI transitions
- use bouncy or elastic curves for playful touches
- use smooth, slower curves for important onboarding or arrival moments

# Common Mistakes

- making every transition bounce or elastic
- using curves without understanding their effect
- setting durations and curves that feel inconsistent with the app's style

# Debugging

If the motion feels wrong:

1. change only the curve first
2. inspect whether the values are increasing correctly
3. check if the animation is too fast or too slow
4. choose a more conventional curve if the design appears noisy

# Interview Questions

1. What is the purpose of `CurvedAnimation`?
2. Why is a linear animation often not ideal?
3. What does `Curves.easeInOut` do?
4. When would you use `Curves.bounceOut`?
5. What is the difference between a curve and a tween?
6. How does `CurvedAnimation` relate to `AnimationController`?
7. What does `CurveTween` do in a tween chain?
8. Why are curves important for mobile UX?
9. What is a typical use case for `Curves.fastOutSlowIn`?
10. How does a curve change the animation perception?

# Try It Yourself

Run the same transition with `Curves.linear`, `Curves.easeInOut`, and `Curves.elasticOut` and compare the experience. This gives you a direct understanding of curve behavior.

# Challenge Project

Build a simple splash screen where a card enters with a curve and a loading icon rotates with a different motion profile. Explain how each curve changes the feel.

# Quick Revision Sheet

- `CurvedAnimation` applies a curve to a parent animation
- curves reshape the motion profile to feel more natural
- common curves include `easeInOut`, `bounceOut`, and `elasticOut`
- use them to make motion feel intentional and polished
- the controller gives time; the curve changes the rhythm
- good curves improve user comfort and perceived quality
