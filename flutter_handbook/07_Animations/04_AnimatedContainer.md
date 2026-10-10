# Introduction

`AnimatedContainer` is one of the most beginner-friendly animation widgets in Flutter.

It gives you a smooth animation when container properties change, such as:

- width
- height
- color
- border radius
- padding
- alignment
- decoration

Instead of suddenly changing from one value to another, Flutter interpolates the changes and animates them over a given duration.

# Why this animation exists

Developers use `AnimatedContainer` to create responsive, polished UI without writing a custom animation controller each time.

It is useful when a button expands, a card transforms, or a widget changes appearance based on a state toggle.

This is one of the first animation widgets beginners should learn because it reduces code while improving user experience.

# Real-world analogy

Imagine a box being moved from one shelf to another.

If it teleports, it feels abrupt. If it glides smoothly between positions, the action feels natural.

`AnimatedContainer` does the same thing for a widget: it animates movement between property values.

# Internal Working

`AnimatedContainer` is a widget that listens to a target value and animates between the old and new property states.

When a property changes, Flutter compares the new state with the previous one and interpolates the difference over the specified duration.

The internal process is:

```
state changes
   |
   v
new values are compared
   |
   v
interpolation happens over time
   |
   v
frame is repainted with updated values
```

It is an implicit animation widget, which means Flutter manages the timing for you.

# Syntax

```dart
AnimatedContainer(
  duration: const Duration(milliseconds: 500),
  curve: Curves.easeInOut,
  width: 200,
  height: 100,
  color: Colors.blue,
  child: const Text('Hello'),
)
```

Important properties include:

- `duration`: how long the animation lasts
- `curve`: the motion style
- `width`, `height`: final size
- `color`: final background color
- `padding`: inner spacing
- `alignment`: child placement
- `decoration`: border, gradient, shadow, etc.

# Complete Example 1

```dart
import 'package:flutter/material.dart';

void main() => runApp(const AnimatedContainerDemo());

class AnimatedContainerDemo extends StatefulWidget {
  const AnimatedContainerDemo({super.key});

  @override
  State<AnimatedContainerDemo> createState() => _AnimatedContainerDemoState();
}

class _AnimatedContainerDemoState extends State<AnimatedContainerDemo> {
  bool _expanded = false;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('AnimatedContainer')),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              AnimatedContainer(
                duration: const Duration(milliseconds: 700),
                curve: Curves.easeInOutBack,
                width: _expanded ? 240 : 120,
                height: _expanded ? 180 : 100,
                padding: EdgeInsets.all(_expanded ? 20 : 10),
                decoration: BoxDecoration(
                  color: _expanded ? Colors.green : Colors.blue,
                  borderRadius: BorderRadius.circular(_expanded ? 28 : 14),
                  boxShadow: [
                    BoxShadow(
                      color: Colors.black.withOpacity(0.15),
                      offset: const Offset(0, 8),
                      blurRadius: 12,
                    ),
                  ],
                ),
                alignment: Alignment.center,
                child: const Text(
                  'Tap me',
                  style: TextStyle(
                    color: Colors.white,
                    fontSize: 22,
                    fontWeight: FontWeight.bold,
                  ),
                ),
              ),
              const SizedBox(height: 24),
              ElevatedButton(
                onPressed: () {
                  setState(() {
                    _expanded = !_expanded;
                  });
                },
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

- `_expanded` is the state that decides the final size and appearance.
- `AnimatedContainer` animates from the current values to the target values.
- `duration` sets the animation speed.
- `curve` changes how the animation moves.
- width and height change smoothy between 120x100 and 240x180.
- `padding` also animates with the new value.
- `decoration` changes its color and border radius.
- `setState` triggers the animation.

## Widget tree

```text
MaterialApp
└── Scaffold
    ├── AppBar
    └── Center
        └── Column
            ├── AnimatedContainer
            └── ElevatedButton
```

## Execution flow

```text
Button tapped
   |
   v
setState updates _expanded
   |
   v
AnimatedContainer compares old and new values
   |
   v
Flutter interpolates amount over duration
   |
   v
UI animates to the new size and style
```

## ASCII phone preview

```text
┌────────────────────────────┐
authored by : boxmotion     │
│                            │
│        [Tap me]            │
│                            │
│                            │
│           [Toggle]         │
│                            │
│                            │
│                            │
└────────────────────────────┘
```

## Example explanation

This example uses one flag to animate size, padding, color, and border radius together. The result feels natural because multiple properties animate in a coordinated way.

## Common mistakes

- forgetting to change the state variable
- setting the same value repeatedly and expecting motion
- using large durations for very short UI changes

## Professional use case

`AnimatedContainer` is used for expansion panels, profile cards, modal previews, action chips, and many small design interactions.

# Complete Example 2

```dart
import 'package:flutter/material.dart';

void main() => runApp(const AnimatedContainerColorDemo());

class AnimatedContainerColorDemo extends StatefulWidget {
  const AnimatedContainerColorDemo({super.key});

  @override
  State<AnimatedContainerColorDemo> createState() => _AnimatedContainerColorDemoState();
}

class _AnimatedContainerColorDemoState extends State<AnimatedContainerColorDemo> {
  bool _isDark = false;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        backgroundColor: _isDark ? Colors.black : Colors.white,
        body: Center(
          child: GestureDetector(
            onTap: () => setState(() => _isDark = !_isDark),
            child: AnimatedContainer(
              duration: const Duration(milliseconds: 600),
              curve: Curves.easeInOut,
              width: 220,
              height: 220,
              decoration: BoxDecoration(
                shape: BoxShape.circle,
                color: _isDark ? Colors.orange : Colors.blue,
              ),
            ),
          ),
        ),
      ),
    );
  }
}
```

This example shows color animation and an entire screen background change.

# Complete Example 3

```dart
import 'package:flutter/material.dart';

void main() => runApp(const AnimatedContainerLayoutDemo());

class AnimatedContainerLayoutDemo extends StatefulWidget {
  const AnimatedContainerLayoutDemo({super.key});

  @override
  State<AnimatedContainerLayoutDemo> createState() => _AnimatedContainerLayoutDemoState();
}

class _AnimatedContainerLayoutDemoState extends State<AnimatedContainerLayoutDemo> {
  bool _isLarge = false;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        body: Center(
          child: AnimatedContainer(
            duration: const Duration(milliseconds: 500),
            curve: Curves.easeInOut,
            width: _isLarge ? 280 : 150,
            height: _isLarge ? 120 : 80,
            alignment: _isLarge ? Alignment.centerLeft : Alignment.center,
            padding: EdgeInsets.symmetric(
              horizontal: _isLarge ? 18 : 8,
              vertical: _isLarge ? 12 : 6,
            ),
            decoration: BoxDecoration(
              color: Colors.teal,
              borderRadius: BorderRadius.circular(_isLarge ? 20 : 8),
            ),
            child: Text(
              _isLarge ? 'Expanded message' : 'Short',
              style: const TextStyle(color: Colors.white, fontSize: 18),
            ),
          ),
        ),
      ),
    );
  }
}
```

This uses the same widget with width, padding, and alignment changes to create a richer layout effect.

# Advanced Example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const AnimatedContainerAdvancedDemo());

class AnimatedContainerAdvancedDemo extends StatefulWidget {
  const AnimatedContainerAdvancedDemo({super.key});

  @override
  State<AnimatedContainerAdvancedDemo> createState() => _AnimatedContainerAdvancedDemoState();
}

class _AnimatedContainerAdvancedDemoState extends State<AnimatedContainerAdvancedDemo> {
  bool _active = false;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        body: Center(
          child: GestureDetector(
            onTap: () => setState(() => _active = !_active),
            child: AnimatedContainer(
              duration: const Duration(milliseconds: 800),
              curve: Curves.fastOutSlowIn,
              width: _active ? 260 : 120,
              height: _active ? 120 : 80,
              margin: EdgeInsets.all(_active ? 16 : 10),
              padding: EdgeInsets.all(_active ? 20 : 8),
              decoration: BoxDecoration(
                borderRadius: BorderRadius.circular(_active ? 30 : 12),
                gradient: LinearGradient(
                  colors: _active
                      ? [Colors.indigo, Colors.purple]
                      : [Colors.blue, Colors.cyan],
                ),
                boxShadow: [
                  BoxShadow(
                    color: Colors.black.withOpacity(0.2),
                    blurRadius: 18,
                    spreadRadius: 1,
                  ),
                ],
              ),
              child: Center(
                child: Text(
                  _active ? 'Expanded' : 'Small',
                  style: const TextStyle(
                    color: Colors.white,
                    fontSize: 20,
                    fontWeight: FontWeight.bold,
                  ),
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

This shows how a single container can animate a whole visual hierarchy.

# Under the Hood

AnimatedContainer does not animate by magic. It uses Flutter's animation system under the hood and schedules rebuilds when a property changes.

Because the widget is a convenience layer, it is very efficient for common changes. It keeps the code easy while still leveraging the framework's advanced interpolation logic.

# Performance Notes

- use `AnimatedContainer` whenever a property changes and there is a simple visual transition
- prefer it over custom animation controllers for small UI changes
- avoid deep widget trees inside heavily animating containers
- keep your decoration simple when possible

# Best Practices

- choose a clear `duration` and `curve`
- animate only the values that matter
- keep animation consistent across the app theme
- use `const` widgets inside when they are static

# Common Mistakes

- changing values without `setState`
- using extremely long durations for quick toggles
- expecting properties like margin or padding to animate without being changed from the old value

# Debugging

If the container does not animate:

1. ensure the state is updated with `setState`
2. verify the target values differ from the previous values
3. check that `duration` is not zero
4. confirm your widget is being rebuilt

# Interview Questions

1. What is the main purpose of `AnimatedContainer`?
2. What kinds of properties does it animate?
3. What is the difference between `AnimatedContainer` and a normal `Container`?
4. Why is `setState` important here?
5. What does `curve` control?
6. Why is it an implicit animation widget?
7. What happens if the property value stays the same?
8. What is a common practical use case for `AnimatedContainer`?
9. Name one benefit of using it over a custom controller.
10. Why is it useful for beginner Flutter development?

# Try It Yourself

Create a card that changes:

- size
- color
- radius
- padding

and animate all changes together with one `AnimatedContainer`.

# Challenge Project

Build a profile card that expands when tapped and shows:

- bigger image
- more spacing
- color change
- a subtle shadow

Use a single `AnimatedContainer` and no custom controller.

# Quick Revision Sheet

- `AnimatedContainer` is a convenience implicit animation widget
- it animates property changes between old and new values
- duration and curve control the motion
- it is perfect for simple, polished UI transitions
- the animation is triggered by a state change
- it saves time while keeping the interface feeling alive
