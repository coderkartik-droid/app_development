# Introduction

`AnimatedAlign` is an animation widget used to change the alignment of a child over time.

It is especially useful when you want a widget to move from one side of a container to another smoothly. Common examples include:

- a button moving to the center
- content shifting from left to right
- onboarding elements aligning to focus points
- map or panel elements repositioning

# Why this animation exists

Positions often change because the UI responds to a user action or system state. If that move is abrupt, it can feel awkward.

`AnimatedAlign` communicates movement cleanly because the child travels smoothly between alignments.

# Real-world analogy

Imagine a person walking from one side of a room to the other.

If they teleport, it feels strange. If they move naturally, the transition is easy to understand.

`AnimatedAlign` does the same for a widget: it moves the child between positions with a smooth visual transition.

# Internal Working

`AnimatedAlign` animates between two alignment values such as:

- `Alignment.topLeft`
- `Alignment.center`
- `Alignment.bottomRight`

The child is placed according to the current alignment and size constraints. Flutter interpolates the alignment progress over time and repaints each frame.

This is another implicit animation widget, which means it handles timing automatically.

# Syntax

```dart
AnimatedAlign(
  duration: const Duration(milliseconds: 500),
  curve: Curves.easeInOut,
  alignment: Alignment.centerRight,
  child: const FlutterLogo(size: 80),
)
```

Important parameters:

- `alignment`: the target placement inside the parent
- `duration`: time of movement
- `curve`: smoothness of movement
- `child`: the widget being aligned

# Complete Example 1

```dart
import 'package:flutter/material.dart';

void main() => runApp(const AnimatedAlignDemo());

class AnimatedAlignDemo extends StatefulWidget {
  const AnimatedAlignDemo({super.key});

  @override
  State<AnimatedAlignDemo> createState() => _AnimatedAlignDemoState();
}

class _AnimatedAlignDemoState extends State<AnimatedAlignDemo> {
  bool _moved = false;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('AnimatedAlign')),
        body: Center(
          child: Container(
            width: 300,
            height: 300,
            color: Colors.grey[200],
            child: AnimatedAlign(
              duration: const Duration(milliseconds: 600),
              curve: Curves.easeInOut,
              alignment: _moved ? Alignment.bottomRight : Alignment.topLeft,
              child: Container(
                width: 80,
                height: 80,
                color: Colors.red,
              ),
            ),
          ),
        ),
        floatingActionButton: FloatingActionButton(
          onPressed: () => setState(() => _moved = !_moved),
          child: const Icon(Icons.swap_horiz),
        ),
      ),
    );
  }
}
```

## Line-by-line explanation

- `_moved` toggles the target alignment.
- The parent container defines the available space.
- `AnimatedAlign` animates the child from one alignment to another.
- `FloatingActionButton` triggers the state change.
- The red box smoothly moves across the container.

## Widget tree

```text
MaterialApp
└── Scaffold
    ├── AppBar
    ├── Center
    │   └── Container
    │       └── AnimatedAlign
    │           └── Container
    └── FloatingActionButton
```

## Execution flow

```text
Button tapped
   |
   v
_moved toggles
   |
   v
AnimatedAlign compares old and new alignment
   |
   v
Flutter interpolates position over time
   |
   v
Child moves smoothly inside the container
```

## ASCII phone preview

```text
┌────────────────────────────┐
authored by : align       │
│                            │
│  [red box]                 │
│                            │
│                            │
│                            │
│                            │
│                [FAB]        │
└────────────────────────────┘
```

## Example explanation

This example shows how alignment changes can animate a widget from one corner to another. The effect makes intent very clear and feels much more professional than instant repositioning.

## Common mistakes

- forgetting to provide a parent container with enough size
- changing the alignment but not the state value
- using alignment changes for text without considering layout constraints

## Professional use case

This is often used in onboarding screens, panel transitions, and floating action layouts.

# Complete Example 2

```dart
import 'package:flutter/material.dart';

void main() => runApp(const AnimatedAlignCenterApp());

class AnimatedAlignCenterApp extends StatefulWidget {
  const AnimatedAlignCenterApp({super.key});

  @override
  State<AnimatedAlignCenterApp> createState() => _AnimatedAlignCenterAppState();
}

class _AnimatedAlignCenterAppState extends State<AnimatedAlignCenterApp> {
  bool _centered = false;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        body: Center(
          child: Container(
            width: 320,
            height: 250,
            color: Colors.blue[100],
            child: AnimatedAlign(
              duration: const Duration(milliseconds: 700),
              curve: Curves.easeInOutBack,
              alignment: _centered ? Alignment.center : Alignment.topCenter,
              child: Container(
                width: 120,
                height: 120,
                color: Colors.green,
              ),
            ),
          ),
        ),
        floatingActionButton: FloatingActionButton(
          onPressed: () => setState(() => _centered = !_centered),
          child: const Icon(Icons.center_focus_strong),
        ),
      ),
    );
  }
}
```

This version centers the element smoothly instead of snapping it into place.

# Complete Example 3

```dart
import 'package:flutter/material.dart';

void main() => runApp(const AnimatedAlignComplexApp());

class AnimatedAlignComplexApp extends StatefulWidget {
  const AnimatedAlignComplexApp({super.key});

  @override
  State<AnimatedAlignComplexApp> createState() => _AnimatedAlignComplexAppState();
}

class _AnimatedAlignComplexAppState extends State<AnimatedAlignComplexApp> {
  Alignment _alignment = Alignment.topLeft;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        body: Center(
          child: Container(
            width: 400,
            height: 400,
            color: Colors.grey[200],
            child: AnimatedAlign(
              duration: const Duration(milliseconds: 600),
              curve: Curves.easeInOut,
              alignment: _alignment,
              child: Container(
                width: 90,
                height: 90,
                decoration: const BoxDecoration(
                  color: Colors.deepPurple,
                  shape: BoxShape.circle,
                ),
              ),
            ),
          ),
        ),
        floatingActionButton: FloatingActionButton(
          onPressed: () {
            setState(() {
              _alignment = _alignment == Alignment.topLeft
                  ? Alignment.bottomRight
                  : Alignment.topLeft;
            });
          },
          child: const Icon(Icons.refresh),
        ),
      ),
    );
  }
}
```

This example animates between two corner positions and shows the control you can get from alignment transitions.

# Advanced Example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const AdvancedAlignApp());

class AdvancedAlignApp extends StatefulWidget {
  const AdvancedAlignApp({super.key});

  @override
  State<AdvancedAlignApp> createState() => _AdvancedAlignAppState();
}

class _AdvancedAlignAppState extends State<AdvancedAlignApp> {
  Alignment _alignment = Alignment.centerLeft;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        body: Center(
          child: Container(
            width: 360,
            height: 360,
            decoration: BoxDecoration(
              color: Colors.indigo[50],
              borderRadius: BorderRadius.circular(24),
            ),
            child: AnimatedAlign(
              duration: const Duration(milliseconds: 800),
              curve: Curves.fastOutSlowIn,
              alignment: _alignment,
              child: Container(
                margin: const EdgeInsets.all(12),
                width: 100,
                height: 100,
                decoration: const BoxDecoration(
                  color: Colors.orange,
                  borderRadius: BorderRadius.all(Radius.circular(18)),
                ),
              ),
            ),
          ),
        ),
        floatingActionButton: FloatingActionButton.extended(
          onPressed: () {
            setState(() {
              _alignment = _alignment == Alignment.centerLeft
                  ? Alignment.centerRight
                  : Alignment.centerLeft;
            });
          },
          label: const Text('Move'),
          icon: const Icon(Icons.compare_arrows),
        ),
      ),
    );
  }
}
```

This shows a more polished layout move with a distinctive material interaction.

# Under the Hood

`AnimatedAlign` is a convenience widget built on Flutter's implicit animation system. It calculates interpolation between the current and target alignment values and schedules rebuilds as the motion progresses.

This means the child not only changes position visually but does so in a way that feels controlled and continuous.

# Performance Notes

- use `AnimatedAlign` only when position changes are meaningful
- keep the child tree lightweight
- do not animate large widget trees repeatedly if a simple layout is enough

# Best Practices

- use it to communicate spatial movement clearly
- keep alignment changes purposeful and not excessive
- match movement with the screen's design language

# Common Mistakes

- placing the child in a container with no size constraints
- expecting alignment movement without a parent size
- using too many simultaneous alignment changes

# Debugging

If the element is not moving:

1. check if the parent container has a defined size
2. confirm that the alignment value actually changes
3. verify `setState` is called
4. inspect if the parent widget layout is preventing motion

# Interview Questions

1. What does `AnimatedAlign` do?
2. Why does it need a parent container with size?
3. What type of values are animated?
4. When would you use `AnimatedAlign` in an app?
5. What is the role of `curve` in this widget?
6. Why is this considered an implicit animation?
7. What happens when the alignment changes but the parent size is tiny?
8. Why is `AnimatedAlign` useful in design-driven apps?
9. What is different between alignment and position in Flutter layout?
10. How would you explain the widget to a beginner?

# Try It Yourself

Build a container with a small child and animate the child from top-left to bottom-right. Experiment with different curves and durations to observe the difference.

# Challenge Project

Create a custom notification panel that slides from the bottom or right using `AnimatedAlign` or a closely related position animation and polish it with a soft curve.

# Quick Revision Sheet

- `AnimatedAlign` moves a child smoothly between alignments
- it animates `Alignment` values inside a parent container
- a parent with a defined size is necessary
- `duration` and `curve` control the movement feel
- it is simple but effective for goal-directed motion
- use it when the user needs to understand a repositioning action
