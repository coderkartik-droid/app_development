# Gesture-Driven Animations

## Introduction

A gesture-driven animation connects motion directly to a user's touch. When the user drags, pinches, or swipes, the interface responds to that input instead of playing a motion that was decided in advance.

There are two related parts:

- **Gesture tracking** reads pointer events such as movement and release velocity.
- **Animation after the gesture** continues or settles the motion, often with a spring or a duration-based animation.

Flutter's `GestureDetector` recognizes common gestures. Its callbacks provide details such as drag distance and velocity. A widget can use these values to update an `AnimationController` while the finger is down, then animate to a resting position after release.

## Why this concept exists

Touch interfaces feel natural when content follows the user's finger. A draggable sheet, swipeable card, or interactive dial should respond immediately, and its release behavior should depend on how it was moved.

Gesture-driven animation combines direct manipulation with animated settling. The finger controls the object during contact; after release, an animation can complete the interaction without an abrupt jump.

## Real-world analogy

Think of pushing a shopping cart. While your hands are on it, you control its position. When you let go, it does not instantly teleport to a parking spot; it rolls or stops according to its momentum and the floor's friction.

A gesture-driven widget behaves similarly: the pointer supplies movement, and a simulation or animation takes over when the pointer is released.

## Internal Working

Flutter receives pointer events from the platform and routes them through the gesture system. A recognizer decides whether the pointer sequence represents a pan, tap, scale, or another gesture. `GestureDetector` exposes callbacks after the relevant recognizer accepts the gesture.

For a drag interaction:

1. `onPanStart` captures the starting state if needed.
2. `onPanUpdate` provides a delta since the previous update. Applying each delta makes the object follow the finger.
3. `onPanEnd` provides velocity at release.
4. The app selects a destination or starts a physics simulation.
5. A ticker-backed animation requests frame callbacks. The controller updates its value, notifies listeners, and a builder updates the visual position.

The gesture callback itself does not draw. It changes state or a controller value; Flutter then rebuilds or repaints the affected portion during frame processing. Vsync keeps controller ticks aligned with display frames. Dispose controllers with their owning `State`.

## Syntax

```dart
GestureDetector(
  onPanUpdate: (details) {
    position.value += details.delta.dx;
  },
  onPanEnd: (details) {
    controller.animateWith(
      SpringSimulation(
        spring,
        position.value,
        destination,
        details.velocity.pixelsPerSecond.dx,
      ),
    );
  },
  child: child,
)
```

`details.delta` is movement since the previous update, whereas `details.localPosition` is the pointer's position in the detector's local coordinate system. Use the delta for incremental movement. Gesture velocity is measured in logical pixels per second; keep the animated position in logical pixels when passing it to a positional simulation.

## Complete Example 1

This app lets a user drag a tile horizontally. On release, it springs to the left or right resting point according to how far it traveled. A fresh drag immediately takes control again.

```dart
import 'package:flutter/material.dart';
import 'package:flutter/physics.dart';

void main() => runApp(const GestureAnimationApp());

class GestureAnimationApp extends StatelessWidget {
  const GestureAnimationApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      theme: ThemeData(useMaterial3: true, colorSchemeSeed: Colors.teal),
      home: const GestureAnimationPage(),
    );
  }
}

class GestureAnimationPage extends StatefulWidget {
  const GestureAnimationPage({super.key});

  @override
  State<GestureAnimationPage> createState() => _GestureAnimationPageState();
}

class _GestureAnimationPageState extends State<GestureAnimationPage>
    with SingleTickerProviderStateMixin {
  late final AnimationController _position;
  static const double _limit = 110;

  @override
  void initState() {
    super.initState();
    _position = AnimationController.unbounded(vsync: this);
  }

  void _settle(double velocity) {
    final target = _position.value >= 0 ? _limit : -_limit;
    const spring = SpringDescription(mass: 1, stiffness: 420, damping: 34);
    _position.animateWith(
      SpringSimulation(spring, _position.value, target, velocity),
    );
  }

  @override
  void dispose() {
    _position.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Drag to choose a side')),
      body: Center(
        child: GestureDetector(
          onPanUpdate: (details) {
            _position.value =
                (_position.value + details.delta.dx)
                    .clamp(-_limit, _limit)
                    .toDouble();
          },
          onPanEnd: (details) => _settle(details.velocity.pixelsPerSecond.dx),
          child: AnimatedBuilder(
            animation: _position,
            child: const _ChoiceTile(),
            builder: (context, child) => Transform.translate(
              offset: Offset(_position.value, 0),
              child: child,
            ),
          ),
        ),
      ),
    );
  }
}

class _ChoiceTile extends StatelessWidget {
  const _ChoiceTile();

  @override
  Widget build(BuildContext context) {
    return Container(
      width: 190,
      height: 120,
      alignment: Alignment.center,
      decoration: BoxDecoration(
        color: Colors.teal,
        borderRadius: BorderRadius.circular(20),
        boxShadow: const [
          BoxShadow(color: Colors.black26, blurRadius: 12, offset: Offset(0, 6)),
        ],
      ),
      child: const Text(
        'Drag me',
        style: TextStyle(color: Colors.white, fontSize: 20),
      ),
    );
  }
}
```

### Line-by-line explanation

- The Material import supplies the app and widgets; the physics import supplies `SpringDescription` and `SpringSimulation`.
- `main` installs the root app. `MaterialApp` provides theme and navigation infrastructure.
- `GestureAnimationPage` is stateful because position persists and changes during a gesture and a spring.
- `SingleTickerProviderStateMixin` provides vsync for the page's one animation controller.
- `_position` is unbounded because its numeric value represents horizontal logical pixels and can be negative or positive.
- `_limit` declares the maximum drag offset in either direction.
- `initState` creates the controller exactly once for this state object.
- `_settle` selects a resting position based on the sign of the current offset and creates a spring using the current position and release velocity.
- `dispose` releases the controller and its ticker.
- `GestureDetector.onPanUpdate` adds each incremental horizontal movement and clamps it to the allowed range. Clamping keeps the finger-controlled position bounded.
- `onPanEnd` passes the horizontal release velocity into `_settle`.
- `AnimatedBuilder` listens to the controller and reuses `_ChoiceTile` as its static child.
- `Transform.translate` moves the tile visually without changing its layout slot.
- `_ChoiceTile` defines the visible rounded tile, shadow, and label.

### Widget tree

```text
MaterialApp
└── GestureAnimationPage
    └── Scaffold
        ├── AppBar
        └── Center
            └── GestureDetector
                └── AnimatedBuilder
                    └── Transform.translate
                        └── _ChoiceTile
```

### Execution flow

```text
Pointer drag -> pan recognizer -> onPanUpdate -> controller value changes
                                                   |
                                                   v
                                          AnimatedBuilder rebuilds
                                                   |
                                                   v
                                            Transform moves tile

Pointer released -> velocity + current position -> choose target
                                                   |
                                                   v
                                           SpringSimulation runs
                                                   |
                                                   v
                                             tile settles
```

### ASCII phone preview

```text
+---------------------------+
|      Drag to choose       |
+---------------------------+
|                           |
|       [ Drag me ]         |
|       <------->           |
|                           |
+---------------------------+
```

The tile follows the finger within its horizontal range, then springs to a resting offset.

### Example explanation

The controller stores a pixel offset, not a progress fraction. This makes `Offset(_position.value, 0)` direct and keeps the drag delta and velocity in compatible units. The animation and gesture both write to the same controller: a drag update becomes the owner of the value during contact, while the spring updates it after release.

If the clamp is removed, the tile can be dragged arbitrarily far away. If `AnimatedBuilder` is removed, changing the controller does not update this part of the interface. If the controller is recreated in `build`, the drag position resets and the ticker lifecycle becomes invalid. If `dispose` is removed, Flutter can report a ticker leak.

The destination rule is intentionally simple. A production swipe action may choose a destination based on distance and velocity, support accessibility alternatives, and animate completely off-screen before removing its item from state.

**Common mistakes:** Using `details.localPosition.dx` as though it were a per-frame delta causes jumps; starting a simulation on every update fights the user's finger; forgetting to clamp can allow accidental off-screen movement; confusing pixels per second with normalized progress makes release motion unpredictable.

**Professional use case:** Swipe-to-dismiss notifications, draggable bottom sheets, card stacks, and scrubbers all combine live gesture tracking with a deliberate settle or dismissal animation.

## Complete Example 2

A gesture can drive a normalized value, such as a knob's progress from zero to one. Map horizontal drag position into that range and render the result with a progress indicator.

```dart
double progressFromDrag(double localX, double trackWidth) {
  return (localX / trackWidth).clamp(0.0, 1.0);
}
```

Attach this helper to `onHorizontalDragUpdate`, using `details.localPosition.dx` and the track width from `LayoutBuilder` constraints. Store the returned fraction in state and use it as `LinearProgressIndicator.value`. The local coordinate is correct here because the goal is to map a pointer location onto the track, rather than accumulate movement.

**Widget tree:** `LayoutBuilder → GestureDetector → Stack(track, thumb)`.  
**Flow:** pointer position in local coordinates → divide by available width → clamp to `[0, 1]` → rebuild track and thumb.

```text
+---------------------------+
| Seek                      |
| [========o-----------]    |
|       42%                 |
+---------------------------+
```

If the clamp is removed, the value can leave the progress indicator's valid range. If the track width is zero, division is invalid for the intended mapping, so only perform the mapping when width is positive. This compact snippet is a helper rather than a standalone app; connect it to the state and widget tree described above.

## Complete Example 3

A pinch gesture can control scale. `onScaleUpdate` reports the gesture scale relative to its start, which is useful for zoomable images.

```dart
GestureDetector(
  onScaleStart: (_) => _scaleAtGestureStart = _scale,
  onScaleUpdate: (details) {
    setState(() {
      _scale = (_scaleAtGestureStart * details.scale)
          .clamp(0.8, 2.5)
          .toDouble();
    });
  },
  child: Transform.scale(
    scale: _scale,
    child: image,
  ),
)
```

`_scaleAtGestureStart` preserves the scale already accumulated before this pinch. `details.scale` is relative to the current gesture's start, so multiplying it by the saved starting scale prevents the image from snapping back when a new pinch begins. The clamp keeps zoom within product-defined limits.

**Widget tree:** `GestureDetector → Transform.scale → image`.  
**Flow:** pinch begins and stores the current scale → pointer separation changes → new scale is computed and clamped → transformed image rebuilds.

```text
+---------------------------+
|       Photo viewer       |
|                           |
|       .----------.        |
|       |  IMAGE   |        |
|       '----------'        |
|        pinch to zoom      |
+---------------------------+
```

If `onScaleStart` is removed, the scale may be based on stale gesture state. If the clamp is removed, users can zoom to unusable values. In a complete page, reset this scale when changing images if that matches the expected product behavior.

## Advanced Example

For a robust draggable surface, separate the **input policy** from the **rendering**:

1. Convert pointer positions or deltas into a clearly defined coordinate system.
2. Clamp direct manipulation to allowed limits.
3. On release, choose a target using both displacement and velocity.
4. Start one simulation toward that target.
5. Let new input interrupt the old simulation without a jump.

For nested scrolling, coordinate gesture ownership with Flutter's scrollable widgets rather than wrapping everything in competing pan detectors. Gesture recognizers participate in a gesture arena; only recognizers that win receive their accepted gesture callbacks. When an interaction should cooperate with scrolling, use the appropriate scroll or drag APIs.

## Under the Hood

Pointer events are distinct from recognized gestures. Raw events describe pointer down, move, and up; recognizers interpret sequences. `GestureDetector` wires recognizers and calls user callbacks once a gesture is accepted.

Controller listeners do not paint directly. They schedule dependent widget updates; Flutter then performs required build, layout, paint, and compositing work for the frame. A `Transform` changes visual placement during painting, whereas changing constraints or dimensions may trigger layout.

Gesture updates may arrive at a different cadence than display frames. Flutter processes input and frame work through its event loop, and the ticker provides time-based animation updates. Do not assume a fixed number of pointer events per frame.

## Performance Notes

- Keep gesture-driven rebuilds localized; use `AnimatedBuilder.child` for static content.
- Prefer transforms for visual movement when layout need not change.
- Avoid expensive work, synchronous decoding, or large computations inside high-frequency gesture callbacks.
- Use `LayoutBuilder` or measured constraints for responsive boundaries rather than hard-coded screen widths.
- Profile dropped frames on representative devices, especially for zooming or large images.
- Dispose controllers and remove any manually registered listeners.

## Best Practices

- Decide whether the gesture needs absolute local positions or incremental deltas.
- Keep coordinate and velocity units consistent.
- Handle boundaries and cancellation deliberately.
- Ensure fresh input can interrupt animation immediately.
- Make the final state obvious and accessible; do not rely only on a swipe.
- Use semantic controls or buttons as alternatives to gestures.
- Use framework scroll and drag widgets when they already model the interaction.

## Common Mistakes

- Applying local pointer coordinates as repeated deltas, causing movement to grow unexpectedly.
- Applying deltas when an absolute track location is required.
- Mixing global and local coordinates.
- Forgetting to clamp position, scale, or progress.
- Starting animations on every pointer update.
- Letting animations and active pointer input fight over the same value.
- Wrapping scrollables in competing gesture detectors without considering the gesture arena.
- Making a gesture the only way to activate an important action.

## Debugging

- Log `details.delta`, `details.localPosition`, and `details.velocity` to confirm the expected coordinate space and sign.
- Check whether the intended recognizer wins when nested scrollables are involved.
- If movement jumps at gesture start, inspect the initial coordinate and saved starting value.
- If the UI updates only after release, verify that the controller or state is updated in the move callback.
- If animation fights the finger, ensure the drag callback assigns the value directly and does not restart a settling animation.
- If scale jumps between pinches, capture the starting scale in `onScaleStart`.

## Interview Questions

1. **What is the difference between a pointer event and a gesture?** A pointer event is a raw contact change; a gesture is a recognized pattern such as a drag or scale.
2. **When should `details.delta` be used?** For incremental movement since the last update.
3. **When should `details.localPosition` be used?** When interpreting the pointer location within a widget, such as mapping it to a slider track.
4. **What units does drag velocity use?** Logical pixels per second.
5. **How can a drag continue naturally after release?** Pass its velocity and current position to a physics simulation or use them to select a destination.
6. **What is the gesture arena?** Flutter's mechanism for resolving competition among recognizers for a pointer sequence.
7. **Why clamp a gesture-driven value?** To keep it within meaningful layout or domain boundaries.
8. **Why use `AnimatedBuilder.child`?** To reuse static widgets rather than rebuilding them on each controller tick.

## Try It Yourself

1. Add a vertical drag and spring the tile back to its original position.
2. Require a minimum swipe velocity before selecting the opposite snap point.
3. Replace the pan recognizer with a scale recognizer for a photo.
4. Display the current offset in text while dragging.
5. Add a reset button for keyboard and accessibility users.

## Challenge Project

Create a swipeable card stack. The top card follows the finger, rotates slightly based on horizontal offset, and either springs back or flies away based on drag distance and velocity. Remove a dismissed card only after its exit animation completes, and provide visible buttons for the same actions.

## Quick Revision Sheet

- `GestureDetector` recognizes gestures; raw pointer events are lower-level.
- Use deltas for incremental movement and local positions for location mapping.
- Gesture release velocity is in logical pixels per second.
- Keep controller value, position, and velocity units compatible.
- Clamp values and plan for interruption.
- Gestures update values; listeners/builders render their changes.
- Use the gesture arena-aware framework APIs for nested scrolling.
- Provide accessible alternatives for gesture-only actions.
