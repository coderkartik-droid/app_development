# Introduction

`AnimatedCrossFade` is used to animate between two child widgets smoothly.

It is helpful when you want to switch between two visual states, such as:

- login form vs success view
- active vs inactive design
- on/off states
- empty state vs filled state

The widget fades between two children while crossing them in a controlled way.

# Why this animation exists

Switching between two complete layouts is a very common app requirement. A sudden replacement can feel abrupt and confuse users.

`AnimatedCrossFade` makes this transition smooth and understandable. Instead of instantly replacing the content, it blends between the old and new views.

# Real-world analogy

Imagine a slide show transition between two images.

A harsh jump feels abrupt. A crossfade feels like a clean editorial transition.

`AnimatedCrossFade` works the same way for UI screens and sections.

# Internal Working

The widget has two children:

- `firstChild`
- `secondChild`

It also has a `crossFadeState` value that tells Flutter which child should be shown as the primary state.

When the state changes, Flutter animates the opacity and layout of the two children so they crossfade over a duration.

This is an implicit animation widget built for state transitions between two complete widgets.

# Syntax

```dart
AnimatedCrossFade(
  duration: const Duration(milliseconds: 500),
  firstChild: const Text('Login'),
  secondChild: const Text('Success'),
  crossFadeState: CrossFadeState.showFirst,
  sizeCurve: Curves.easeInOut,
)
```

Important fields:

- `firstChild` and `secondChild`: the two states
- `crossFadeState`: chooses which child is active
- `duration`: how long the transition lasts
- `sizeCurve`: animation curve for layout dimensions

# Complete Example 1

```dart
import 'package:flutter/material.dart';

void main() => runApp(const AnimatedCrossFadeDemo());

class AnimatedCrossFadeDemo extends StatefulWidget {
  const AnimatedCrossFadeDemo({super.key});

  @override
  State<AnimatedCrossFadeDemo> createState() => _AnimatedCrossFadeDemoState();
}

class _AnimatedCrossFadeDemoState extends State<AnimatedCrossFadeDemo> {
  bool _isSuccess = false;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('AnimatedCrossFade')),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              AnimatedCrossFade(
                duration: const Duration(milliseconds: 700),
                crossFadeState:
                    _isSuccess ? CrossFadeState.showSecond : CrossFadeState.showFirst,
                firstChild: const SizedBox(
                  width: 220,
                  child: Center(
                    child: Text(
                      'Loading...',
                      style: TextStyle(fontSize: 28),
                    ),
                  ),
                ),
                secondChild: const SizedBox(
                  width: 220,
                  child: Center(
                    child: Text(
                      'Success!',
                      style: TextStyle(fontSize: 28, color: Colors.green),
                    ),
                  ),
                ),
                sizeCurve: Curves.easeInOut,
              ),
              const SizedBox(height: 24),
              ElevatedButton(
                onPressed: () => setState(() => _isSuccess = !_isSuccess),
                child: const Text('Toggle state'),
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

- `_isSuccess` is the state flag controlling the current child.
- `AnimatedCrossFade` holds two views: loading state and success state.
- `crossFadeState` chooses which child is visible.
- `firstChild` and `secondChild` switch with a smooth crossfade.
- `sizeCurve` controls how the size changes during the transition.

## Widget tree

```text
MaterialApp
└── Scaffold
    ├── AppBar
    └── Center
        └── Column
            ├── AnimatedCrossFade
            └── ElevatedButton
```

## Execution flow

```text
Button tapped
   |
   v
_isSuccess toggles
   |
   v
crossFadeState changes
   |
   v
AnimatedCrossFade animates the old and new children
   |
   v
UI shows smooth crossfade between states
```

## ASCII phone preview

```text
┌────────────────────────────┐
authored by : crossfade    │
│                            │
│        Success!            │
│                            │
│                            │
│      [Toggle state]        │
│                            │
│                            │
│                            │
└────────────────────────────┘
```

## Example explanation

This is a standard success-state animation: instead of instantly replacing one message with another, Flutter crossfades between them. That makes the change feel more intentional and less abrupt.

## Common mistakes

- not updating `crossFadeState`
- using too large a child tree for crossfade
- not giving the animation enough time to be noticeable

## Professional use case

This is common in forms, success/error transitions, onboarding state changes, and animated empty states.

# Complete Example 2

```dart
import 'package:flutter/material.dart';

void main() => runApp(const CrossFadeAuthApp());

class CrossFadeAuthApp extends StatefulWidget {
  const CrossFadeAuthApp({super.key});

  @override
  State<CrossFadeAuthApp> createState() => _CrossFadeAuthAppState();
}

class _CrossFadeAuthAppState extends State<CrossFadeAuthApp> {
  bool _loggedIn = false;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        body: Center(
          child: AnimatedCrossFade(
            duration: const Duration(milliseconds: 800),
            firstChild: const Text(
              'Please sign in',
              style: TextStyle(fontSize: 28),
            ),
            secondChild: const Text(
              'Welcome back!',
              style: TextStyle(fontSize: 28, color: Colors.green),
            ),
            crossFadeState:
                _loggedIn ? CrossFadeState.showSecond : CrossFadeState.showFirst,
            sizeCurve: Curves.easeInOutCubic,
          ),
        ),
        floatingActionButton: FloatingActionButton(
          onPressed: () => setState(() => _loggedIn = !_loggedIn),
          child: const Icon(Icons.login),
        ),
      ),
    );
  }
}
```

This shows a simple auth-like state transition without heavy custom code.

# Complete Example 3

```dart
import 'package:flutter/material.dart';

void main() => runApp(const EmptyStateSwapApp());

class EmptyStateSwapApp extends StatefulWidget {
  const EmptyStateSwapApp({super.key});

  @override
  State<EmptyStateSwapApp> createState() => _EmptyStateSwapAppState();
}

class _EmptyStateSwapAppState extends State<EmptyStateSwapApp> {
  bool _hasItems = false;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        body: Center(
          child: AnimatedCrossFade(
            duration: const Duration(milliseconds: 600),
            firstChild: const Text(
              'No items yet',
              style: TextStyle(fontSize: 24),
            ),
            secondChild: const Text(
              'You have 3 new items',
              style: TextStyle(fontSize: 24, color: Colors.deepPurple),
            ),
            crossFadeState:
                _hasItems ? CrossFadeState.showSecond : CrossFadeState.showFirst,
            sizeCurve: Curves.easeInOut,
          ),
        ),
        floatingActionButton: FloatingActionButton(
          onPressed: () => setState(() => _hasItems = !_hasItems),
          child: const Icon(Icons.add),
        ),
      ),
    );
  }
}
```

This is a UI pattern frequently used in app screens that present either an empty or filled state.

# Advanced Example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const AdvancedCrossFadeApp());

class AdvancedCrossFadeApp extends StatefulWidget {
  const AdvancedCrossFadeApp({super.key});

  @override
  State<AdvancedCrossFadeApp> createState() => _AdvancedCrossFadeAppState();
}

class _AdvancedCrossFadeAppState extends State<AdvancedCrossFadeApp> {
  bool _showInfo = false;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        body: Center(
          child: AnimatedCrossFade(
            duration: const Duration(milliseconds: 900),
            firstChild: Container(
              width: 250,
              height: 150,
              color: Colors.blue[100],
              child: const Center(child: Text('Offline mode')),
            ),
            secondChild: Container(
              width: 250,
              height: 150,
              color: Colors.green[100],
              child: const Center(child: Text('Online mode')),
            ),
            crossFadeState:
                _showInfo ? CrossFadeState.showSecond : CrossFadeState.showFirst,
            sizeCurve: Curves.fastOutSlowIn,
            firstCurve: Curves.easeIn,
            secondCurve: Curves.easeOut,
          ),
        ),
        floatingActionButton: FloatingActionButton(
          onPressed: () => setState(() => _showInfo = !_showInfo),
          child: const Icon(Icons.wifi),
        ),
      ),
    );
  }
}
```

This uses separate curves for the first and second child for a more refined feel.

# Under the Hood

`AnimatedCrossFade` effectively manages two children in a transition. The widget blends their opacity and resizes when needed. This makes it good for toggling between complete states while maintaining the user's sense of continuity.

It is designed to help developers avoid ugly swaps in their interface.

# Performance Notes

- avoid putting extremely large trees inside crossfade if they are not needed
- use it when a whole state transition is visually important
- keep the two versions simple and focused

# Best Practices

- use it for state replacement where continuity matters
- keep both states semantically clear
- use consistent sizing and motion to make transitions feel smooth

# Common Mistakes

- forgetting to update the state that drives the crossfade
- using it for ordinary tiny color flickers
- making the transition too slow for a simple toggle

# Debugging

If the crossfade does not happen:

1. verify `crossFadeState` value changes
2. confirm both children are valid widgets
3. inspect whether state rebuilds are happening
4. test with a very obvious difference between the two views

# Interview Questions

1. What is `AnimatedCrossFade` used for?
2. What are `firstChild` and `secondChild`?
3. How does `crossFadeState` decide what to show?
4. Why is this better than an instant switch?
5. When would you use it in a real app?
6. What does `sizeCurve` control?
7. Why might two different curves produce a better result?
8. What is the difference between a fade and a crossfade?
9. How does `AnimatedCrossFade` support state transitions?
10. What kinds of screens benefit from this pattern?

# Try It Yourself

Create a screen that switches between a loading state and a success state using `AnimatedCrossFade`. Try changing the duration and observe how it impacts perception.

# Challenge Project

Build a mini form that shows a login form first and then a success state after the user submits. Use `AnimatedCrossFade` to make the transition smooth.

# Quick Revision Sheet

- `AnimatedCrossFade` animates between two children
- `crossFadeState` decides which child is active
- it is useful for state-based transitions
- it blends the visual difference instead of teleporting between views
- the animation improves clarity and perceived quality
- use it when one screen or state replaces another
