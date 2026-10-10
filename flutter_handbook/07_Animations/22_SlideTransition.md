# Introduction

`SlideTransition` moves a child along an offset over time.

It is often used for:

- drawers sliding in
- modal content appearing from the bottom
- menu panels moving into view
- cards entering from the side

It is a very common and effective way to signal movement across the screen.

# Why this animation exists

Users often better understand movement when they see a clear path. A child sliding across the screen tells the story of transition more clearly than a jump or a fade alone.

This makes `SlideTransition` highly useful for navigation, alerts, and layered UI.

# Real-world analogy

Imagine a curtain sliding open. The motion is visible and intentional, and it tells you something is changing.

A slide animation works in the same way for widgets.

# Internal Working

`SlideTransition` receives an animation that represents a position offset over time. The child is translated by an `Offset`, which can move horizontally, vertically, or both.

Common pattern:

```
AnimationController -> Tween<Offset> -> SlideTransition -> child moves across frame
```

This allows you to specify the start and end offset of the child over the course of the animation.

# Syntax

```dart
SlideTransition(
  position: Tween<Offset>(
    begin: const Offset(1, 0),
    end: Offset.zero,
  ).animate(_controller),
  child: const MyWidget(),
)
```

Important values:

- `position`: animation that determines offset
- `child`: the widget sliding
- `transformHitTests`: optional interaction behavior for transformed widgets

# Complete Example 1

```dart
import 'package:flutter/material.dart';

void main() => runApp(const SlideTransitionDemo());

class SlideTransitionDemo extends StatefulWidget {
  const SlideTransitionDemo({super.key});

  @override
  State<SlideTransitionDemo> createState() => _SlideTransitionDemoState();
}

class _SlideTransitionDemoState extends State<SlideTransitionDemo>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;
  late final Animation<Offset> _offset;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 700),
    );
    _offset = Tween<Offset>(
      begin: const Offset(1.0, 0.0),
      end: Offset.zero,
    ).chain(CurveTween(curve: Curves.easeOutCubic)).animate(_controller);
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
              SlideTransition(
                position: _offset,
                child: Container(
                  width: 220,
                  height: 120,
                  color: Colors.orange,
                  child: const Center(
                    child: Text(
                      'Slide in',
                      style: TextStyle(color: Colors.white, fontSize: 28),
                    ),
                  ),
                ),
              ),
              const SizedBox(height: 20),
              ElevatedButton(
                onPressed: () => _controller.forward(from: 0),
                child: const Text('Animate'),
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

- `Tween<Offset>` defines start and end positions.
- `begin: Offset(1,0)` means the child starts off-screen to the right.
- `end: Offset.zero` means it settles in its natural location.
- `SlideTransition` translates the child according to the current offset.
- the button restarts the slide.

## Widget tree

```text
MaterialApp
└── Scaffold
    └── Center
        └── Column
            ├── SlideTransition
            │   └── Container
            └── ElevatedButton
```

## Execution flow

```text
Button tapped
   |
   v
Controller starts animation
   |
   v
Tween computes current offset between start and end
   |
   v
SlideTransition updates child position each frame
   |
   v
Child slides into place smoothly
```

## ASCII phone preview

```text
┌────────────────────────────┐
authored by : slide       │
│                            │
│       Slide in            │
│                            │
│        [Animate]          │
│                            │
│                            │
│                            │
└────────────────────────────┘
```

## Example explanation

This is a very common and highly readable transition: the view enters from the right and settles into place. It clearly communicates direction and movement.

## Common mistakes

- forgetting to start the animation controller
- using `Offset.zero` incorrectly for the final position
- choosing a direction that conflicts with the app layout

## Professional use case

Used in navigation pages, drawer content, onboarding steps, and contextual panels.

# Complete Example 2

```dart
import 'package:flutter/material.dart';

void main() => runApp(const SlideMenuApp());

class SlideMenuApp extends StatefulWidget {
  const SlideMenuApp({super.key});

  @override
  State<SlideMenuApp> createState() => _SlideMenuAppState();
}

class _SlideMenuAppState extends State<SlideMenuApp>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;
  bool _showMenu = false;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 500),
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
        body: Stack(
          children: [
            Center(
              child: ElevatedButton(
                onPressed: () {
                  setState(() => _showMenu = !_showMenu);
                  if (_showMenu) {
                    _controller.forward();
                  } else {
                    _controller.reverse();
                  }
                },
                child: const Text('Toggle menu'),
              ),
            ),
            Align(
              alignment: Alignment.centerRight,
              child: SlideTransition(
                position: Tween<Offset>(
                  begin: const Offset(1, 0),
                  end: Offset.zero,
                ).animate(
                  CurvedAnimation(
                    parent: _controller,
                    curve: Curves.easeOutCubic,
                  ),
                ),
                child: Container(
                  width: 220,
                  height: double.infinity,
                  color: Colors.teal,
                  child: const Center(
                    child: Text(
                      'Menu',
                      style: TextStyle(color: Colors.white, fontSize: 28),
                    ),
                  ),
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

This creates a sliding side menu, one of the most practical uses of `SlideTransition`.

# Complete Example 3

```dart
import 'package:flutter/material.dart';

void main() => runApp(const SlideBottomSheetApp());

class SlideBottomSheetApp extends StatefulWidget {
  const SlideBottomSheetApp({super.key});

  @override
  State<SlideBottomSheetApp> createState() => _SlideBottomSheetAppState();
}

class _SlideBottomSheetAppState extends State<SlideBottomSheetApp>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;
  bool _open = false;

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
        body: Stack(
          children: [
            Center(
              child: ElevatedButton(
                onPressed: () {
                  setState(() => _open = !_open);
                  if (_open) {
                    _controller.forward();
                  } else {
                    _controller.reverse();
                  }
                },
                child: const Text('Open sheet'),
              ),
            ),
            Positioned(
              left: 0,
              right: 0,
              bottom: 0,
              child: SlideTransition(
                position: Tween<Offset>(
                  begin: const Offset(0, 1),
                  end: Offset.zero,
                ).animate(
                  CurvedAnimation(
                    parent: _controller,
                    curve: Curves.easeOutBack,
                  ),
                ),
                child: Container(
                  height: 180,
                  color: Colors.indigo,
                  child: const Center(
                    child: Text(
                      'Bottom sheet',
                      style: TextStyle(color: Colors.white, fontSize: 24),
                    ),
                  ),
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

This creates a sliding bottom sheet pattern.

# Advanced Example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const AdvancedSlideApp());

class AdvancedSlideApp extends StatefulWidget {
  const AdvancedSlideApp({super.key});

  @override
  State<AdvancedSlideApp> createState() => _AdvancedSlideAppState();
}

class _AdvancedSlideAppState extends State<AdvancedSlideApp>
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
        body: Center(
          child: SlideTransition(
            position: Tween<Offset>(
              begin: const Offset(0.0, 1.0),
              end: Offset.zero,
            ).chain(CurveTween(curve: Curves.fastOutSlowIn)).animate(_controller),
            child: Container(
              width: 260,
              height: 180,
              decoration: BoxDecoration(
                color: Colors.deepPurple,
                borderRadius: BorderRadius.circular(24),
              ),
              child: const Center(
                child: Text(
                  'Premium reveal',
                  style: TextStyle(color: Colors.white, fontSize: 30),
                ),
              ),
            ),
          ),
        ),
        floatingActionButton: FloatingActionButton(
          onPressed: () => _controller.forward(from: 0),
          child: const Icon(Icons.keyboard_arrow_up),
        ),
      ),
    );
  }
}
```

This is a premium panel reveal using a strong slide motion and curve.

# Under the Hood

The offset animation uses a 2D vector to decide how much the child should move in x and y directions. Flutter uses the internal transform matrix to translate the child during each frame.

Unlike `FadeTransition`, the child actually changes position in the layout transform, so it feels like a movement rather than a fading or size change.

# Performance Notes

- slide transitions are still lightweight
- use them for navigation and overlay interactions
- avoid turning heavy layouts into animated translated content if the movement is not needed

# Best Practices

- use slide direction to reinforce meaning
- keep the motion route intuitive
- combine with opacity for stronger transitions

# Common Mistakes

- wrong start/end offset values
- moving UI in a direction that feels inconsistent with context
- overusing slide when a smaller fade would be cleaner

# Debugging

If the widget does not slide:

1. verify the offset path is correct
2. confirm the controller is moving through the animation
3. inspect whether the transform is applied to the right widget
4. confirm the animation is using the correct `Offset` values

# Interview Questions

1. What does `SlideTransition` animate?
2. Why is offset movement useful for UI transitions?
3. What is `Tween<Offset>` used for here?
4. Why is a start offset often off-screen?
5. What is the difference between slide and fade transitions?
6. How do you create a bottom sheet animation with this widget?
7. Why is direction meaningful in UI movement?
8. What is meant by a transform-based animation?
9. When would you choose this over a `ScaleTransition`?
10. How does a curve change the feel of the sliding motion?

# Try It Yourself

Build a side panel that slides in from the right. Then try sliding from the bottom and from the left to compare how the user perceives each direction.

# Challenge Project

Create a mini app with a bottom sheet and a right-side drawer that both slide in with different curves and durations.

# Quick Revision Sheet

- `SlideTransition` moves a child using an offset animation
- `Tween<Offset>` defines the start and end positions
- ideal for drawers, sheets, and layered transitions
- direction matters a lot for clarity and UX
- combine with fade or scale to create more polished transitions
- good movement uses both purpose and direction
