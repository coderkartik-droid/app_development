# Introduction

`FadeTransition` is a simple but powerful animation widget that changes the opacity of a child over time.

It is often used for:

- image reveal
- content entry
- dialog appearance
- screen transitions
- state changes

Opacity is one of the easiest ways to create smooth transitions because the user naturally understands a fade-in or fade-out.

# Why this animation exists

Sometimes the important thing is not movement but the appearance or disappearance of content. A fade communicates that change without forcing a jump or abrupt visual interruption.

`FadeTransition` is especially effective for transitions that should feel gentle and controlled.

# Real-world analogy

Think of a light bulb turning on in a room. It does not appear instantly at full brightness; it warms up gradually.

A fade transition is exactly this idea in UI: the content becomes visible gently instead of suddenly.

# Internal Working

`FadeTransition` listens to an animation value and applies it as an opacity value to the child.

The child remains in the widget tree while its alpha changes, which means the operation is a visual transition rather than an actual removal of the widget.

Flow:

```
Animation value -> opacity -> child fades -> painted frame updates
```

This is commonly paired with an `AnimationController` and a `CurvedAnimation`.

# Syntax

```dart
FadeTransition(
  opacity: _animation,
  child: const Text('Visible now'),
)
```

Important parts:

- `opacity`: animation that drives transparency
- `child`: content that fades
- `alwaysIncludeSemantics`: optional accessibility behavior

# Complete Example 1

```dart
import 'package:flutter/material.dart';

void main() => runApp(const FadeTransitionDemo());

class FadeTransitionDemo extends StatefulWidget {
  const FadeTransitionDemo({super.key});

  @override
  State<FadeTransitionDemo> createState() => _FadeTransitionDemoState();
}

class _FadeTransitionDemoState extends State<FadeTransitionDemo>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;
  late final Animation<double> _opacity;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 1),
    );
    _opacity = CurvedAnimation(
      parent: _controller,
      curve: Curves.easeIn,
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
              FadeTransition(
                opacity: _opacity,
                child: Container(
                  width: 220,
                  height: 120,
                  color: Colors.blue,
                  child: const Center(
                    child: Text(
                      'Fade in',
                      style: TextStyle(color: Colors.white, fontSize: 32),
                    ),
                  ),
                ),
              ),
              const SizedBox(height: 20),
              ElevatedButton(
                onPressed: () => _controller.forward(from: 0),
                child: const Text('Show'),
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

- `AnimationController` drives the temporal change.
- `Curves.easeIn` makes the fade smoother at the start.
- `FadeTransition` takes the opacity animation and applies it to the child.
- the button restarts the fade when tapped.

## Widget tree

```text
MaterialApp
└── Scaffold
    └── Center
        └── Column
            ├── FadeTransition
            │   └── Container
            └── ElevatedButton
```

## Execution flow

```text
Button pressed
   |
   v
Controller starts at 0 and moves forward
   |
   v
Opacity animation value changes over time
   |
   v
FadeTransition updates alpha and redraws current frame
```

## ASCII phone preview

```text
┌────────────────────────────┐
authored by : fade        │
│                            │
│       Fade in              │
│                            │
│        [Show]              │
│                            │
│                            │
│                            │
└────────────────────────────┘
```

## Example explanation

The content does not jump into view; it appears gradually, which helps users understand a transition and gives the UI a more premium feel.

## Common mistakes

- using a fade where a slide or scale is more appropriate
- not starting the controller
- using opacity changes without meaningful state transitions

## Professional use case

Used in splash screens, content entry, dialogs, success states, and route transitions.

# Complete Example 2

```dart
import 'package:flutter/material.dart';

void main() => runApp(const FadeStatusApp());

class FadeStatusApp extends StatefulWidget {
  const FadeStatusApp({super.key});

  @override
  State<FadeStatusApp> createState() => _FadeStatusAppState();
}

class _FadeStatusAppState extends State<FadeStatusApp>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;
  bool _visible = true;

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
              FadeTransition(
                opacity: CurvedAnimation(
                  parent: _controller,
                  curve: Curves.easeInOut,
                ),
                child: Text(
                  _visible ? 'Online' : 'Offline',
                  style: TextStyle(
                    fontSize: 28,
                    color: _visible ? Colors.green : Colors.red,
                  ),
                ),
              ),
              const SizedBox(height: 20),
              ElevatedButton(
                onPressed: () {
                  _visible = !_visible;
                  _controller.forward(from: 0);
                  setState(() {});
                },
                child: const Text('Toggle status'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

This uses a fade to signal a state change like online/offline status.

# Complete Example 3

```dart
import 'package:flutter/material.dart';

void main() => runApp(const FadeAlertApp());

class FadeAlertApp extends StatefulWidget {
  const FadeAlertApp({super.key});

  @override
  State<FadeAlertApp> createState() => _FadeAlertAppState();
}

class _FadeAlertAppState extends State<FadeAlertApp>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;
  bool _showWarning = false;

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

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              FadeTransition(
                opacity: CurvedAnimation(
                  parent: _controller,
                  curve: Curves.easeOut,
                ),
                child: _showWarning
                    ? Container(
                        width: 260,
                        height: 90,
                        color: Colors.amber,
                        child: const Center(
                          child: Text(
                            'Warning! Save before exit',
                            style: TextStyle(color: Colors.black, fontSize: 18),
                          ),
                        ),
                      )
                    : const SizedBox.shrink(),
              ),
              const SizedBox(height: 20),
              ElevatedButton(
                onPressed: () {
                  setState(() => _showWarning = !_showWarning);
                  _controller.forward(from: 0);
                },
                child: const Text('Toggle warning'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

This creates a warning banner that fades in and out.

# Advanced Example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const AdvancedFadeApp());

class AdvancedFadeApp extends StatefulWidget {
  const AdvancedFadeApp({super.key});

  @override
  State<AdvancedFadeApp> createState() => _AdvancedFadeAppState();
}

class _AdvancedFadeAppState extends State<AdvancedFadeApp>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 1200),
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
          child: FadeTransition(
            opacity: CurvedAnimation(
              parent: _controller,
              curve: Curves.easeInOutCubic,
            ),
            child: Container(
              width: 260,
              height: 180,
              decoration: BoxDecoration(
                color: Colors.teal,
                borderRadius: BorderRadius.circular(24),
              ),
              child: const Center(
                child: Text(
                  'Repeating fade',
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

This is a fade cycle used for a subtle repeated reveal.

# Under the Hood

The fade value is simply an alpha level. The animation system produces the value over time, and Flutter applies it to the child. The child remains in the layout while only its opacity changes.

This is part of why `FadeTransition` is so attractive: it creates subtle transitions without deeply altering layout or properties.

# Performance Notes

- fades are efficient and visually simple
- useful for low-cost transitions
- avoid overusing repeated fades for too many elements at once

# Best Practices

- use fade for gentle state transitions
- pair it with other animations for richer UI experience
- keep it short and purposeful

# Common Mistakes

- thinking a fade removes the widget from the layout
- overusing it for all transitions
- not pairing the fade with meaningful state changes

# Debugging

If the fade is not visible:

1. verify the animation is being driven by the controller
2. confirm the child is still in the tree
3. check whether the opacity value is starting at 0 or 1 where expected
4. inspect if the parent widget is masking the fade visually

# Interview Questions

1. What does `FadeTransition` change?
2. Why is a fade more gentle than a hard switch?
3. What property does the animation modify?
4. Why is the child not removed from the tree?
5. When would you use it in a real app?
6. What is the difference between `FadeTransition` and `AnimatedOpacity`?
7. Which curve is best for a gentle fade?
8. Why is this suitable for onboarding or loading screens?
9. What does a fade imply to users?
10. When might a fade be less appropriate than a slide or scale?

# Try It Yourself

Create a small card that fades in on startup and another that fades out when a button is pressed. Compare the visual feel with a quick scale effect.

# Challenge Project

Build a mini onboarding step screen with text and a CTA button that fades in as the user advances through steps.

# Quick Revision Sheet

- `FadeTransition` animates opacity
- useful for subtle entry and exit transitions
- keeps the child in the tree while changing alpha
- ideal for screens, overlays, and notifications
- combine with curves for gentle user experience
- simple, elegant, and widely useful
