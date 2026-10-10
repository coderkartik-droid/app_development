# Introduction

`AnimatedPositioned` is a very useful widget when a child needs to move smoothly within a `Stack`.

Unlike `AnimatedAlign`, which animates alignment inside a general parent, `AnimatedPositioned` works primarily with `Stack` layout and animates the child's `left`, `right`, `top`, and `bottom` positions.

This is a very common animation technique for mobile UI such as:

- bottom sheets
- floating panels
- toast notifications
- decorative overlays
- sliding cards

# Why this animation exists

In a `Stack`, widgets are layered on top of each other. Sometimes one widget needs to move from one area to another smoothly.

`AnimatedPositioned` provides that motion without requiring custom animation code.

It improves interface clarity because the user can see that the element is moving to a new place rather than abruptly appearing somewhere else.

# Real-world analogy

Imagine a card sliding from the corner of a desk into the center.

The movement is easy to track because it follows a visible path. A sudden teleport feels harder to follow.

`AnimatedPositioned` gives that same clear motion to widgets layered inside a `Stack`.

# Internal Working

`AnimatedPositioned` is designed for `Stack` layout. Each child in a `Stack` can be positioned using an explicit `left`, `right`, `top`, or `bottom` value.

When these values change, Flutter interpolates the difference and re-renders the stack in each frame. The result is a smooth transition between old and new positions.

This is an implicit animation widget.

# Syntax

```dart
Stack(
  children: [
    AnimatedPositioned(
      duration: const Duration(milliseconds: 500),
      curve: Curves.easeInOut,
      left: 20,
      top: 30,
      width: 100,
      height: 100,
      child: const FlutterLogo(),
    ),
  ],
)
```

Important properties:

- `left`, `right`, `top`, `bottom`: position edges
- `width`, `height`: size of the child
- `duration`: movement time
- `curve`: motion style

# Complete Example 1

```dart
import 'package:flutter/material.dart';

void main() => runApp(const AnimatedPositionedDemo());

class AnimatedPositionedDemo extends StatefulWidget {
  const AnimatedPositionedDemo({super.key});

  @override
  State<AnimatedPositionedDemo> createState() => _AnimatedPositionedDemoState();
}

class _AnimatedPositionedDemoState extends State<AnimatedPositionedDemo> {
  bool _moved = false;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('AnimatedPositioned')),
        body: Center(
          child: Container(
            width: 320,
            height: 320,
            color: Colors.grey[200],
            child: Stack(
              children: [
                AnimatedPositioned(
                  duration: const Duration(milliseconds: 600),
                  curve: Curves.easeInOut,
                  left: _moved ? 180 : 20,
                  top: _moved ? 180 : 20,
                  width: 100,
                  height: 100,
                  child: Container(
                    color: Colors.blue,
                    child: const Center(
                      child: Text(
                        'Move',
                        style: TextStyle(color: Colors.white, fontSize: 18),
                      ),
                    ),
                  ),
                ),
              ],
            ),
          ),
        ),
        floatingActionButton: FloatingActionButton(
          onPressed: () => setState(() => _moved = !_moved),
          child: const Icon(Icons.play_arrow),
        ),
      ),
    );
  }
}
```

## Line-by-line explanation

- `Stack` allows layered child positioning.
- `_moved` toggles the target location.
- `left` and `top` animate between two values.
- `AnimatedPositioned` smoothly transitions the child between the old and new positions.
- `FloatingActionButton` triggers the move.

## Widget tree

```text
MaterialApp
└── Scaffold
    ├── AppBar
    ├── Center
    │   └── Container
    │       └── Stack
    │           └── AnimatedPositioned
    │               └── Container
    └── FloatingActionButton
```

## Execution flow

```text
Button tapped
   |
   v
_moved changes
   |
   v
AnimatedPositioned receives new coordinates
   |
   v
Flutter interpolates old and new left/top values
   |
   v
Child moves smoothly within the stack
```

## ASCII phone preview

```text
┌────────────────────────────┐
authored by : stackmove    │
│                            │
│   [blue box]               │
│                            │
│                            │
│                            │
│                            │
│                 [play]      │
└────────────────────────────┘
```

## Example explanation

The inner box moves from one corner to another smoothly. This is a pristine example of how `AnimatedPositioned` is used for layered motion on screens.

## Common mistakes

- using `AnimatedPositioned` without a `Stack`
- forgetting to set `left` or `top` properly
- expecting smooth movement if the parent has no size constraints

## Professional use case

Common in floating bottom nav bars, draggable cards, contextual menus, and layered onboarding states.

# Complete Example 2

```dart
import 'package:flutter/material.dart';

void main() => runApp(const PositionedPopupApp());

class PositionedPopupApp extends StatefulWidget {
  const PositionedPopupApp({super.key});

  @override
  State<PositionedPopupApp> createState() => _PositionedPopupAppState();
}

class _PositionedPopupAppState extends State<PositionedPopupApp> {
  bool _open = false;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        body: Stack(
          children: [
            Center(
              child: ElevatedButton(
                onPressed: () => setState(() => _open = !_open),
                child: const Text('Open panel'),
              ),
            ),
            AnimatedPositioned(
              duration: const Duration(milliseconds: 500),
              curve: Curves.easeOutBack,
              left: 30,
              right: 30,
              bottom: _open ? 30 : -150,
              child: Container(
                height: 120,
                decoration: BoxDecoration(
                  color: Colors.white,
                  borderRadius: BorderRadius.circular(18),
                  boxShadow: const [
                    BoxShadow(
                      color: Colors.black12,
                      blurRadius: 12,
                      offset: Offset(0, 8),
                    ),
                  ],
                ),
                child: const Center(
                  child: Text('Sliding panel', style: TextStyle(fontSize: 24)),
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

This creates a floating panel that slides in and out of view.

# Complete Example 3

```dart
import 'package:flutter/material.dart';

void main() => runApp(const AnimatedPositionedProfileApp());

class AnimatedPositionedProfileApp extends StatefulWidget {
  const AnimatedPositionedProfileApp({super.key});

  @override
  State<AnimatedPositionedProfileApp> createState() => _AnimatedPositionedProfileAppState();
}

class _AnimatedPositionedProfileAppState extends State<AnimatedPositionedProfileApp> {
  bool _selected = false;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        body: Center(
          child: SizedBox(
            width: 360,
            height: 260,
            child: Stack(
              children: [
                Positioned(
                  left: 20,
                  top: 20,
                  child: Container(
                    width: 120,
                    height: 120,
                    color: Colors.amber,
                  ),
                ),
                AnimatedPositioned(
                  duration: const Duration(milliseconds: 700),
                  curve: Curves.easeInOut,
                  left: _selected ? 200 : 80,
                  top: 70,
                  width: 80,
                  height: 80,
                  child: Container(
                    color: Colors.red,
                  ),
                ),
              ],
            ),
          ),
        ),
        floatingActionButton: FloatingActionButton(
          onPressed: () => setState(() => _selected = !_selected),
          child: const Icon(Icons.change_circle),
        ),
      ),
    );
  }
}
```

This demonstrates the ability to move a small overlay over a layout.

# Advanced Example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const AdvancedStackMotionApp());

class AdvancedStackMotionApp extends StatefulWidget {
  const AdvancedStackMotionApp({super.key});

  @override
  State<AdvancedStackMotionApp> createState() => _AdvancedStackMotionAppState();
}

class _AdvancedStackMotionAppState extends State<AdvancedStackMotionApp> {
  bool _expanded = false;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        body: Center(
          child: SizedBox(
            width: 400,
            height: 400,
            child: Stack(
              children: [
                Positioned.fill(
                  child: Container(color: Colors.grey[200]),
                ),
                AnimatedPositioned(
                  duration: const Duration(milliseconds: 700),
                  curve: Curves.fastOutSlowIn,
                  left: _expanded ? 50 : 20,
                  top: _expanded ? 50 : 20,
                  right: _expanded ? 50 : 20,
                  bottom: _expanded ? 50 : 20,
                  child: Container(
                    decoration: BoxDecoration(
                      color: Colors.indigo,
                      borderRadius: BorderRadius.circular(_expanded ? 30 : 10),
                    ),
                  ),
                ),
              ],
            ),
          ),
        ),
        floatingActionButton: FloatingActionButton(
          onPressed: () => setState(() => _expanded = !_expanded),
          child: const Icon(Icons.aspect_ratio),
        ),
      ),
    );
  }
}
```

This is a broad layered transition simulating a card opening or expanding within a background.

# Under the Hood

A `Stack` is a layout strategy that allows children to occupy the same area with explicit positioning. `AnimatedPositioned` interpolates between old and new coordinates so the child appears to travel smoothly from one location to another.

The result is visually intuitive and extremely useful in complex layered interfaces.

# Performance Notes

- use it when a `Stack` layout is already needed
- keep the child count limited
- avoid unrelated large animations in the same stack

# Best Practices

- use `AnimatedPositioned` instead of rebuilding many layout widgets when position is the main change
- keep the movement natural and readable
- combine with shadows or opacity for stronger emphasis

# Common Mistakes

- using it without a `Stack`
- failing to define the parent size
- trying to animate many children simultaneously without reason

# Debugging

If the child is not moving:

1. make sure the parent is a `Stack`
2. verify the `left` or `top` values changed
3. check whether `setState` is called
4. confirm the parent has a definite size

# Interview Questions

1. What is the main use case for `AnimatedPositioned`?
2. Why does it work best inside a `Stack`?
3. What properties are typically animated?
4. How is this different from `AnimatedAlign`?
5. What does the `curve` parameter affect?
6. Why is a defined parent size important?
7. Name a real app feature that could use `AnimatedPositioned`.
8. Why is smooth position change better than a jump?
9. What are the benefits of layered UI motion in mobile apps?
10. When should you avoid using `AnimatedPositioned`?

# Try It Yourself

Create a `Stack` with a small floating panel and animate it from offscreen to on-screen using `AnimatedPositioned`. Then play with `left`, `right`, `bottom`, and `top` values.

# Challenge Project

Build a floating mini-player or bottom sheet that slides up into place with a soft movement and shadow.

# Quick Revision Sheet

- `AnimatedPositioned` moves a child within a `Stack`
- it animates `left`, `right`, `top`, and `bottom` values
- it is an implicit animation widget
- good for layered UI and overlay transitions
- parent size and stack structure matter
- use it for clean and understandable movement between positions
