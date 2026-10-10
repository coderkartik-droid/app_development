# Introduction

`SizeTransition` animates the size of a child between a small and larger value.

It is useful when a widget expands or collapses, for example:

- reveal details in an accordion panel
- show hidden content section
- expand a filter menu
- animate panel size in a dashboard card

This widget is especially helpful when the content changes from compressed to expanded while staying in the same general location.

# Why this animation exists

An abrupt size change can feel harsh and inconsistent with the surrounding layout. If an item grows gradually, the user can understand the new content space better.

A size transition is often more readable than a separate fade or slide when content actually expands within the same area.

# Real-world analogy

Picture a room expanding as more furniture is added. The movement is not a teleport; it is a reconnection of space.

A size transition works similarly in UI: it smoothly updates the available space for a widget.

# Internal Working

`SizeTransition` uses an `Animation<double>` to animate the child’s size. It takes a `sizeFactor` or similar value and applies it to a `Align` or `SizeTransition` around the child.

This is usually used with a controller and a curve to create a smooth resize behavior.

# Syntax

```dart
SizeTransition(
  sizeFactor: _animation,
  axisAlignment: -1.0,
  child: const MyWidget(),
)
```

Important values:

- `sizeFactor`: animation determining how large the child becomes
- `axisAlignment`: where the child aligns while transitioning
- `axis`: optional axis of animation
- `child`: content being resized

# Complete Example 1

```dart
import 'package:flutter/material.dart';

void main() => runApp(const SizeTransitionDemo());

class SizeTransitionDemo extends StatefulWidget {
  const SizeTransitionDemo({super.key});

  @override
  State<SizeTransitionDemo> createState() => _SizeTransitionDemoState();
}

class _SizeTransitionDemoState extends State<SizeTransitionDemo>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;
  late final Animation<double> _size;
  bool _expanded = false;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 600),
    );
    _size = CurvedAnimation(
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
              SizeTransition(
                sizeFactor: _size,
                axis: Axis.vertical,
                axisAlignment: -1.0,
                child: Container(
                  height: 120,
                  width: 220,
                  color: Colors.green,
                  child: const Center(
                    child: Text(
                      'Expanded content',
                      style: TextStyle(color: Colors.white, fontSize: 24),
                    ),
                  ),
                ),
              ),
              const SizedBox(height: 20),
              ElevatedButton(
                onPressed: () {
                  setState(() => _expanded = !_expanded);
                  if (_expanded) {
                    _controller.forward();
                  } else {
                    _controller.reverse();
                  }
                },
                child: const Text('Toggle size'),
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

- `_expanded` toggles the size state.
- `sizeFactor` changes over time according to the controller.
- `axis: Axis.vertical` means the child grows vertically.
- the content is inside a `Container` of fixed size and expands within the size transition.
- the button toggles the controller direction.

## Widget tree

```text
MaterialApp
└── Scaffold
    └── Center
        └── Column
            ├── SizeTransition
            │   └── Container
            └── ElevatedButton
```

## Execution flow

```text
Button tapped
   |
   v
_expanded toggles and controller direction changes
   |
   v
SizeTransition animates sizeFactor
   |
   v
Container grows or shrinks over time
```

## ASCII phone preview

```text
┌────────────────────────────┐
authored by : size        │
│                            │
│  Expanded content          │
│                            │
│         [Toggle size]      │
│                            │
│                            │
│                            │
└────────────────────────────┘
```

## Example explanation

Instead of suddenly appearing with a fixed height, the content grows smoothly. This makes the expansion feel intentional and less abrupt.

## Common mistakes

- not controlling the axis appropriately
- not toggling the controller direction when collapse is needed
- using size transition when the child should simply fade in instead

## Professional use case

Used for accordion sections, expandable product descriptions, and expanding filter panels.

# Complete Example 2

```dart
import 'package:flutter/material.dart';

void main() => runApp(const SizeFilterApp());

class SizeFilterApp extends StatefulWidget {
  const SizeFilterApp({super.key});

  @override
  State<SizeFilterApp> createState() => _SizeFilterAppState();
}

class _SizeFilterAppState extends State<SizeFilterApp>
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
        body: Center(
          child: Column(
            children: [
              ElevatedButton(
                onPressed: () {
                  setState(() => _open = !_open);
                  if (_open) {
                    _controller.forward();
                  } else {
                    _controller.reverse();
                  }
                },
                child: const Text('Open filters'),
              ),
              SizeTransition(
                sizeFactor: CurvedAnimation(
                  parent: _controller,
                  curve: Curves.easeInOut,
                ),
                axisAlignment: -1.0,
                child: Container(
                  width: 260,
                  height: 150,
                  color: Colors.purple,
                  child: const Center(
                    child: Text(
                      'Filter options',
                      style: TextStyle(color: Colors.white, fontSize: 24),
                    ),
                  ),
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

This creates a filter panel that expands vertically when opened.

# Complete Example 3

```dart
import 'package:flutter/material.dart';

void main() => runApp(const SizeInfoApp());

class SizeInfoApp extends StatefulWidget {
  const SizeInfoApp({super.key});

  @override
  State<SizeInfoApp> createState() => _SizeInfoAppState();
}

class _SizeInfoAppState extends State<SizeInfoApp>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;
  bool _showExtra = false;

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
        body: Center(
          child: Column(
            children: [
              Container(
                width: 220,
                decoration: BoxDecoration(
                  color: Colors.grey[200],
                  borderRadius: BorderRadius.circular(12),
                ),
                child: Column(
                  children: [
                    TextButton(
                      onPressed: () {
                        setState(() => _showExtra = !_showExtra);
                        if (_showExtra) {
                          _controller.forward();
                        } else {
                          _controller.reverse();
                        }
                      },
                      child: const Text('More info'),
                    ),
                    SizeTransition(
                      sizeFactor: CurvedAnimation(
                        parent: _controller,
                        curve: Curves.easeOut,
                      ),
                      child: const Padding(
                        padding: EdgeInsets.all(12),
                        child: Text('This is the extra info that opens and closes.'),
                      ),
                    ),
                  ],
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

This is a common expandable content pattern used in product details or FAQs.

# Advanced Example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const AdvancedSizeTransitionApp());

class AdvancedSizeTransitionApp extends StatefulWidget {
  const AdvancedSizeTransitionApp({super.key});

  @override
  State<AdvancedSizeTransitionApp> createState() => _AdvancedSizeTransitionAppState();
}

class _AdvancedSizeTransitionAppState extends State<AdvancedSizeTransitionApp>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;
  bool _expanded = false;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 800),
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
            children: [
              MaterialButton(
                onPressed: () {
                  setState(() => _expanded = !_expanded);
                  if (_expanded) {
                    _controller.forward();
                  } else {
                    _controller.reverse();
                  }
                },
                child: const Text('Expand panel'),
              ),
              SizeTransition(
                sizeFactor: CurvedAnimation(
                  parent: _controller,
                  curve: Curves.fastOutSlowIn,
                ),
                axis: Axis.vertical,
                axisAlignment: -1.0,
                child: Container(
                  width: 280,
                  decoration: BoxDecoration(
                    color: Colors.blueGrey,
                    borderRadius: BorderRadius.circular(16),
                  ),
                  child: const Padding(
                    padding: EdgeInsets.all(20),
                    child: Text(
                      'This advanced panel expands with a premium motion curve and fixed area.',
                      style: TextStyle(color: Colors.white),
                    ),
                  ),
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

This is a more complete card-like expansion panel with a refined motion style.

# Under the Hood

A `SizeTransition` works by animating the size factor of the child while the framework maintains the appropriate layout. This is different from simply toggling visibility because the content remains in the layout but changes its space gradually.

This makes size transitions ideal for expandable content that should not feel like a hard cut.

# Performance Notes

- size transitions are efficient for simple expansion/collapse behavior
- avoid extremely large content expansions if the result is not material to the user
- keep the child layout light and targeted

# Best Practices

- use size transitions when content genuinely expands within the same area
- set a meaningful axis and alignment
- prefer this over abrupt re-layouts when space is meaningful

# Common Mistakes

- using size transitions for content that should simply appear/disappear
- failing to reverse the controller when collapsing
- not considering axis alignment in the final look

# Debugging

If the panel does not expand:

1. confirm `sizeFactor` is active
2. inspect whether the controller is reversed when closing
3. verify parent layout can accommodate the new size
4. ensure the child has a real height or width to animate

# Interview Questions

1. What does `SizeTransition` animate?
2. When is it more useful than a fade?
3. Why is the axis important?
4. What is `sizeFactor` controlling?
5. What is a common use case for expandable panels?
6. Why does it feel more professional than a sudden resize?
7. What does `axisAlignment` affect?
8. When should you prefer `SizeTransition` over `AnimatedContainer`?
9. How does it differ from `AnimatedOpacity`?
10. How is it useful in real product UIs?

# Try It Yourself

Build a small FAQ section where each answer expands when the user taps the question. Animate the size and observe how the layout feels.

# Challenge Project

Create an expandable product details panel where each section grows when selected and collapses when closed.

# Quick Revision Sheet

- `SizeTransition` animates the size of a child
- useful for reveal/collapse behaviors and panels
- `sizeFactor` controls the growth amount
- choose the axis and alignment carefully
- great for meaningful space changes that should feel smooth
- use it when content genuinely expands and contracts in place
