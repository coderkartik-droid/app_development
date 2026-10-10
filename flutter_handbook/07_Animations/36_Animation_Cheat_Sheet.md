# Animation Cheat Sheet

## Introduction

This chapter is a compact reference for choosing Flutter animation APIs and remembering their responsibilities. It is not a replacement for the preceding chapters; use it to recall the right starting point and then check the full API details for the feature you are building.

The central pattern is: define a changing value, decide what drives it, map it to a visual property, and make sure the widget tree listens to it.

## Why this concept exists

Flutter has implicit animations, explicit controllers, transitions, simulations, gesture callbacks, and custom painting. A quick decision guide reduces the temptation to use a controller for every small property change—or to force an implicit widget to handle sequencing it was not designed for.

## Real-world analogy

A toolbox contains many tools, but a successful repair begins by identifying the job. Choose a hammer for a nail and a screwdriver for a screw; likewise choose an implicit widget for a simple target change and a controller for coordinated timeline control.

## Internal Working

Animations are updated with the frame pipeline. Ticker callbacks provide elapsed time; controllers and simulations calculate values; curves and tweens shape and map those values; widgets or painters listen and update; Flutter performs the required build, layout, paint, and compositing work.

Lifecycle rule: create an owned controller in `initState`, use a `TickerProvider` for vsync, and dispose it in `dispose`. An animation does not update a UI unless something listens to its changing value.

## Syntax

### Choose the API

| Requirement | Starting point |
|---|---|
| Change a few properties between target values | `AnimatedContainer`, `AnimatedPadding`, `AnimatedAlign` |
| Fade or switch widgets | `AnimatedOpacity`, `AnimatedSwitcher`, `AnimatedCrossFade` |
| Move between routes with a shared element | `Hero` |
| Control progress, direction, or sequence | `AnimationController` |
| Convert progress into property values | `Tween`, `ColorTween`, `TweenSequence` |
| Shape timing | `Curves`, `CurvedAnimation` |
| Build from an animation | `AnimatedBuilder` |
| Apply a common visual transition | `FadeTransition`, `ScaleTransition`, `SlideTransition`, `RotationTransition`, `SizeTransition` |
| Continue with velocity-sensitive motion | `SpringSimulation`, `FrictionSimulation` |
| Draw a custom animated graphic | `CustomPainter` with a repaint `Listenable` |

### Explicit lifecycle skeleton

```dart
late final AnimationController controller;

@override
void initState() {
  super.initState();
  controller = AnimationController(
    vsync: this,
    duration: const Duration(milliseconds: 300),
  );
}

@override
void dispose() {
  controller.dispose();
  super.dispose();
}
```

## Complete Example 1

This runnable reference app uses a button to animate a box's alignment and color implicitly. It demonstrates the simplest common case: state changes the targets and `AnimatedContainer` handles interpolation.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const CheatSheetApp());

class CheatSheetApp extends StatelessWidget {
  const CheatSheetApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      theme: ThemeData(useMaterial3: true, colorSchemeSeed: Colors.indigo),
      home: const CheatSheetPage(),
    );
  }
}

class CheatSheetPage extends StatefulWidget {
  const CheatSheetPage({super.key});

  @override
  State<CheatSheetPage> createState() => _CheatSheetPageState();
}

class _CheatSheetPageState extends State<CheatSheetPage> {
  bool _moved = false;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Animation quick reference')),
      body: Column(
        children: [
          Expanded(
            child: AnimatedContainer(
              duration: const Duration(milliseconds: 350),
              curve: Curves.easeInOut,
              alignment: _moved ? Alignment.bottomRight : Alignment.topLeft,
              color: _moved ? Colors.teal : Colors.indigo,
              child: const Padding(
                padding: EdgeInsets.all(24),
                child: Icon(Icons.star, color: Colors.white, size: 42),
              ),
            ),
          ),
          Padding(
            padding: const EdgeInsets.all(20),
            child: FilledButton(
              onPressed: () => setState(() => _moved = !_moved),
              child: const Text('Animate'),
            ),
          ),
        ],
      ),
    );
  }
}
```

### Line-by-line explanation

- `main` starts the app and its reference page.
- `MaterialApp` sets theme and removes the debug banner for a clean demo.
- `CheatSheetPage` owns `_moved`, a boolean target state.
- `Scaffold` supplies the app bar and body.
- `Column` divides the screen into an animated region and a button region.
- `Expanded` gives the `AnimatedContainer` all remaining vertical space.
- `duration` and `curve` configure interpolation and pacing.
- `alignment` moves the child between opposite corners.
- `color` changes at the same time as alignment.
- `Padding` keeps the star away from the edge.
- `FilledButton` toggles the target and triggers a rebuild.
- No controller is needed: the implicit widget manages its own transition.

### Widget tree

```text
MaterialApp
└── CheatSheetPage
    └── Scaffold
        ├── AppBar
        └── Column
            ├── Expanded
            │   └── AnimatedContainer
            │       └── Padding
            │           └── Icon
            └── FilledButton
```

### Execution flow

```text
Button -> _moved toggles -> widget rebuilds with new targets
                                  |
                                  v
                AnimatedContainer interpolates color/alignment
                                  |
                                  v
                             frames appear
```

### ASCII phone preview

```text
+---------------------------+
| Animation quick reference |
+---------------------------+
| *                         |
|                           |
|                           |
|                         * |
|                           |
|       [ Animate ]         |
+---------------------------+
```

### Example explanation

`AnimatedContainer` is appropriate because the interaction only changes target properties after a rebuild. The child moves with alignment and the color transitions concurrently. No manually controlled progress or sequence is needed.

If `Expanded` is removed, the container may not receive the remaining screen height as intended. If `setState` is removed, targets do not update. Replacing `AnimatedContainer` with `Container` makes property changes immediate. Adding an `AnimationController` here would add lifecycle responsibilities without providing useful extra control.

**Common mistakes:** An implicit widget animates only supported property changes. It does not make its child contents automatically fade or sequence. Use an explicit transition if the motion requires custom sequencing or interruption rules.

**Professional use case:** Selection highlights, expanding tiles, and alignment changes are good implicit-animation candidates.

## Complete Example 2

An explicit transition is appropriate when a controller owns a timeline.

```dart
ScaleTransition(
  scale: animation,
  child: child,
)
```

`animation` must be an `Animation<double>`. Its values represent the scale directly; a tween or curved animation can define its range and timing.

**Widget tree:** `ScaleTransition → child`.  
**Flow:** controller tick → animation value → scale transition → child is painted at the new scale.

Without a controller or another changing animation, this widget does not animate by itself.

## Complete Example 3

For a draggable object that returns naturally, use a simulation after release.

```dart
controller.animateWith(
  SpringSimulation(spring, controller.value, target, velocity),
);
```

The simulation consumes starting value, destination, and initial velocity. Units for position and velocity must be compatible.

**Widget tree:** `GestureDetector → AnimatedBuilder → Transform`.  
**Flow:** release → select destination → spring samples values per frame → transform updates.

If an interaction does not need release momentum, a fixed-duration animation may be simpler and more predictable.

## Advanced Example

### Value and lifecycle reminders

| Concept | Responsibility |
|---|---|
| `AnimationController` | Owns a time/value progression and ticker |
| `Animation<T>` | Exposes a changing typed value and notifications |
| `Tween<T>` | Maps normalized progress into a property range |
| `Curve` | Reshapes progress timing |
| `AnimatedBuilder` | Rebuilds a dependent subtree for notifications |
| `CustomPainter(repaint:)` | Repaints custom drawing for notifications |
| `TickerProvider` | Supplies vsync to ticker-backed animation |

### Performance reminders

- Use `AnimatedBuilder.child` for static descendants.
- Prefer paint-only transforms when layout should not change.
- Use `RepaintBoundary` only when profiling justifies it.
- Keep repeated animations localized and stop them when unnecessary.
- Profile UI and raster work on representative devices.

### Accessibility reminders

- Do not communicate state only through motion or color.
- Keep actions available when animations are reduced or disabled.
- Preserve semantics and hit testing when fading or transforming widgets.
- Make gesture-driven functionality available through visible controls where appropriate.

## Under the Hood

The ticker supplies elapsed time. The controller or simulation produces values, then a tween and/or curve shapes them. A widget, builder, transition, or painter listens for updates. Flutter schedules only the required frame work, which may include build, layout, paint, and compositing.

Implicit widgets manage their internal controller-like transitions as targets change. Explicit controllers give direct control but require deliberate lifecycle management.

## Performance Notes

- Measure before optimizing.
- Keep animated work small and avoid unnecessary layout.
- Avoid rebuilding unrelated ancestors for every tick.
- Stop off-screen repeating animations.
- Remember that opacity and effects can involve compositing.
- Test slow devices, large text, and high refresh rates.

## Best Practices

- Begin with the simplest API that meets the desired behavior.
- Separate target state from animation progress.
- Keep units and ranges explicit.
- Define start, finish, reverse, interruption, and cancellation behavior.
- Dispose controllers and remove manual listeners.
- Provide semantics and a reduced-motion-friendly experience.
- Use DevTools to validate performance assumptions.

## Common Mistakes

- Using explicit controllers for every simple property change.
- Expecting an animation value to update UI without a listener.
- Creating controllers or starting animations in `build`.
- Forgetting to dispose ticker-backed objects.
- Mixing normalized progress with pixel-based velocity.
- Confusing widget rebuilding with painting the full screen.
- Assuming transparent means removed from layout or interaction.
- Adding repaint boundaries without measurement.

## Debugging

- Confirm state/target changes first.
- Check controller value, status, bounds, and listener connection.
- Verify tween range and curve output.
- Inspect ticker lifecycle warnings.
- Inspect layout constraints when content jumps or overflows.
- Use DevTools to locate slow build, layout, paint, or raster frames.
- Check semantics and hit testing for visually hidden elements.

## Interview Questions

1. **When should you use implicit versus explicit animation?** Implicit for direct target changes; explicit for timeline control, sequences, and coordination.
2. **What does a tween do?** Maps one value range to another typed range.
3. **What does a curve do?** Shapes how progress changes over time.
4. **What is vsync?** Frame-synchronized ticking provided by a ticker provider.
5. **Why is controller disposal important?** It stops ticker activity and releases resources.
6. **What is the frame pipeline?** Flutter's sequence of build/update, layout, paint, and compositing work to produce a frame.
7. **Does a `Transform` change layout position?** Typically no; it applies a visual transform during painting.
8. **What does `AnimatedBuilder.child` help with?** Reusing static descendants across animation updates.
9. **Does `RepaintBoundary` always improve performance?** No; it trades repaint isolation against layer and memory cost.
10. **How do physics animations use gestures?** Gesture position and release velocity initialize a simulation or select a target.

## Try It Yourself

1. Recreate the quick-reference example with `AnimatedAlign`.
2. Add a fade transition driven by an explicit controller.
3. Explain the widget tree and controller lifecycle aloud.
4. Use DevTools to compare an alignment animation with a transform animation.
5. Make the interaction usable without motion.

## Challenge Project

Build a one-page animation reference app with examples for an implicit transition, an explicit transition, and a gesture-driven spring. Each example must have a reset control, a visible explanation of its API choice, and semantics that describe its state.

## Quick Revision Sheet

- **Simple targets:** implicit widgets.
- **Controlled timeline:** `AnimationController`.
- **Map progress:** `Tween`.
- **Shape timing:** `Curve`.
- **Widget listener:** `AnimatedBuilder`.
- **Common visual output:** transition widgets.
- **Velocity response:** physics simulations.
- **Custom drawings:** `CustomPainter(repaint:)`.
- Create/dispose controllers with state; profile; preserve accessibility.
