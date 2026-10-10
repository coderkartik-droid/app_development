# Common Animation Mistakes

## Introduction

Animation bugs often come from lifecycle, ownership, bounds, or timing—not from a complicated formula. A controller may restart every build, a tween may use the wrong range, or an animation may continue after its page disappears.

This chapter groups common mistakes and shows a safe pattern for an explicit animation. The same checks apply to implicit animations, transitions, gestures, and custom drawing.

## Why this concept exists

Animations run repeatedly over time and interact with the widget lifecycle. A mistake can appear as a flicker, jump, ticker warning, stale visual, excessive work, or an animation that never starts.

Knowing the usual failure patterns makes it faster to debug and helps prevent problems before they reach users.

## Real-world analogy

An animation controller is like a timer connected to a motor. If you create a new timer every time you check the motor, it never follows one reliable schedule. If you forget to turn it off when leaving the room, it keeps using power when nobody is there.

Create the controller once, start it when the interaction calls for it, and stop/dispose it with the owner.

## Internal Working

Widgets are immutable descriptions; their `State` objects live across rebuilds. `build` can run many times, so it is not a one-time setup method. A ticker-backed controller should normally be initialized in `initState` and disposed in `dispose`.

Animation listeners schedule dependent work. If a listener calls `setState`, the state rebuilds; if the object is removed, its listener must no longer target disposed state. Flutter detects some lifecycle errors, such as updating disposed state or leaving a ticker active.

The rendering pipeline does not animate a property merely because a number changes. A widget or render object must listen to that value and use it to produce updated output.

## Syntax

```dart
late final AnimationController controller;
late final Animation<double> opacity;

@override
void initState() {
  super.initState();
  controller = AnimationController(
    vsync: this,
    duration: const Duration(milliseconds: 300),
  );
  opacity = CurvedAnimation(
    parent: controller,
    curve: Curves.easeInOut,
  );
}

@override
void dispose() {
  controller.dispose();
  super.dispose();
}
```

Keep controller construction and cleanup paired. Start it in response to a user action or a deliberate lifecycle transition.

## Complete Example 1

This app demonstrates a common safe pattern: one controller is created per `State`, an animation maps progress to opacity, a button starts or reverses it, and the controller is disposed correctly.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MistakeGuideApp());

class MistakeGuideApp extends StatelessWidget {
  const MistakeGuideApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      theme: ThemeData(useMaterial3: true, colorSchemeSeed: Colors.orange),
      home: const SafeAnimationPage(),
    );
  }
}

class SafeAnimationPage extends StatefulWidget {
  const SafeAnimationPage({super.key});

  @override
  State<SafeAnimationPage> createState() => _SafeAnimationPageState();
}

class _SafeAnimationPageState extends State<SafeAnimationPage>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;
  late final Animation<double> _opacity;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 500),
    );
    _opacity = CurvedAnimation(
      parent: _controller,
      curve: Curves.easeInOut,
    );
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  void _toggle() {
    if (_controller.status == AnimationStatus.completed) {
      _controller.reverse();
    } else {
      _controller.forward();
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Safe animation lifecycle')),
      body: Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            FadeTransition(
              opacity: _opacity,
              child: Container(
                width: 150,
                height: 100,
                alignment: Alignment.center,
                color: Colors.deepOrange,
                child: const Text(
                  'Notice me',
                  style: TextStyle(color: Colors.white),
                ),
              ),
            ),
            const SizedBox(height: 24),
            FilledButton(
              onPressed: _toggle,
              child: const Text('Toggle visibility'),
            ),
          ],
        ),
      ),
    );
  }
}
```

### Line-by-line explanation

- `main` runs the root `MaterialApp`, which provides app-level theme and page configuration.
- `SafeAnimationPage` is stateful because the controller's progress must survive rebuilds.
- `SingleTickerProviderStateMixin` provides vsync for the one controller.
- `_controller` owns the timeline. `_opacity` adapts its progress into a curved animation.
- `initState` creates both objects once. The 500 ms duration describes travel from one controller endpoint to the other.
- `CurvedAnimation` applies ease-in/ease-out timing; the value remains suitable for opacity.
- `dispose` releases the controller before the state is removed.
- `_toggle` reverses from completed or runs forward otherwise. Pressing while it is in flight continues toward the forward endpoint.
- `Scaffold`, `Center`, and `Column` arrange the demo.
- `FadeTransition` listens to `_opacity` and updates the child's alpha without manually calling `setState` each frame.
- The `Container` and `Text` are the visible content.
- `FilledButton` provides an explicit interaction to start or reverse the animation.

### Widget tree

```text
MaterialApp
└── SafeAnimationPage
    └── Scaffold
        ├── AppBar
        └── Center
            └── Column
                ├── FadeTransition
                │   └── Container
                │       └── Text
                ├── SizedBox
                └── FilledButton
```

### Execution flow

```text
Button pressed -> _toggle -> controller.forward/reverse
                                     |
                                     v
                              CurvedAnimation updates
                                     |
                                     v
                              FadeTransition listens
                                     |
                                     v
                                frame is painted
```

### ASCII phone preview

```text
+---------------------------+
| Safe animation lifecycle  |
+---------------------------+
|                           |
|      +-------------+      |
|      | Notice me   |      |
|      +-------------+      |
|                           |
|   [ Toggle visibility ]  |
+---------------------------+
```

The orange panel fades between transparent and fully visible when the button is pressed.

### Example explanation

The animation is declared in state because its lifetime must match the page, not an individual call to `build`. `FadeTransition` consumes an `Animation<double>` directly, so the page does not need `setState` for each animation tick.

If the controller is created in `build`, every rebuild can replace the animation and restart it. If the transition is removed and no listener remains, the controller may still animate but the visual will not change. If `dispose` is removed, the ticker can leak when navigating away.

The same lifecycle applies to `AnimationController`, `AnimationController.unbounded`, and controllers used with builders or custom painters.

**Common mistakes:** Do not call `forward()` in `build`; do not create a controller as a local variable there; do not forget `dispose`; do not attach multiple listeners without removing them; do not use opacity values outside their meaningful range.

**Professional use case:** This pattern is suitable for expandable hints, loading states, dialog transitions, and any reversible visibility affordance.

## Complete Example 2

A bounded controller normally expects values in its configured range. If the visual property needs a different range, use a tween rather than sending the property value directly to the controller.

```dart
final width = Tween<double>(begin: 100, end: 240).animate(controller);
```

Use `width.value` in a `SizedBox(width: width.value)`. The controller stays normalized from zero to one; the tween translates that progress into logical pixels.

**Widget tree:** `AnimatedBuilder → SizedBox(width: width.value) → child`.  
**Flow:** controller progress → tween interpolation → width changes → layout updates.

If the tween is removed and controller value is assigned directly as width, the result is only a one-pixel range. If the controller is unbounded, explicitly constrain or validate values for properties that require bounds.

## Complete Example 3

An animation listener should be detached if it is manually registered and its lifetime does not automatically match the animation.

```dart
void _onTick() {
  if (!mounted) return;
  setState(() {});
}

@override
void initState() {
  super.initState();
  animation.addListener(_onTick);
}

@override
void dispose() {
  animation.removeListener(_onTick);
  super.dispose();
}
```

The `mounted` guard protects against a callback that arrives after removal; removing the listener is the primary lifecycle cleanup. Prefer `AnimatedBuilder` when it fits, because it manages the listening relationship declaratively.

**Widget tree:** `StatefulWidget → widget subtree listening through State`.  
**Flow:** animation notification → listener checks lifetime → `setState` schedules a build.

If `removeListener` is omitted, the animation retains the callback and can keep referencing state that should be gone.

## Advanced Example

When an animation's input changes while it is running, define the intended retargeting behavior. `animateTo` can move from the current value to a new target. A physics simulation can start from the current value and velocity. Replacing an animation object may require updating state and disposing resources associated with the old controller.

For asynchronous completion, remember that navigation or disposal may happen before the returned `TickerFuture` completes. Use lifecycle-aware patterns and avoid assuming a future always finishes normally if the animation is canceled.

## Under the Hood

Controllers own ticker-driven timelines; transitions and builders listen to `Animation` values. The widget lifecycle determines when state is created and destroyed. A ticker can continue requesting frames until stopped or disposed, so ownership and cleanup are essential.

`FadeTransition` changes opacity through a render object rather than requiring the parent to manually rebuild each frame. However, opacity can require compositing and can have hit-testing implications; making a widget invisible does not necessarily remove it from interaction or semantics.

## Performance Notes

- Use specialized transition widgets when available.
- Keep listener scopes small.
- Avoid `setState` on a large ancestor for every tick.
- Do not run hidden infinite animations without a reason.
- Remember that opacity, clipping, blur, and shadows may involve extra compositing work.
- Profile before adding complexity.

## Best Practices

- Create controllers in `initState` and dispose them.
- Start animations from events, not from `build`.
- Use bounded controllers for normalized progress and tweens for property ranges.
- Let transitions listen to animations where possible.
- Define interruption, reverse, and retargeting behavior.
- Keep animation and widget ownership aligned.
- Provide a reduced-motion or non-animated path when appropriate.

## Common Mistakes

- Creating an `AnimationController` in `build`.
- Calling `forward`, `repeat`, or `reverse` during every build.
- Forgetting controller disposal or manual listener removal.
- Updating state after disposal.
- Using mismatched tween and controller ranges.
- Animating a widget that is no longer visible.
- Assuming opacity zero disables gestures or removes semantics.
- Rebuilding a large page for every tick.

## Debugging

- Check the Flutter console for ticker and lifecycle warnings.
- Confirm the controller value changes and a widget listens to it.
- Check bounds, begin/end values, and curve output.
- If the animation restarts unexpectedly, search for controller creation or start calls in `build`.
- If a hidden widget still responds to taps, inspect hit testing and `IgnorePointer`/`AbsorbPointer`.
- If a callback updates disposed state, remove listeners and align their lifetime with the state object.

## Interview Questions

1. **Why shouldn't controllers be created in `build`?** Build can run repeatedly; repeated creation resets state and creates ticker lifecycle problems.
2. **Where should an owned controller be disposed?** In the owning `State.dispose`.
3. **What does `FadeTransition` listen to?** An `Animation<double>` representing opacity.
4. **Why use a tween with a normalized controller?** To map progress to a domain-specific range such as width or color.
5. **Does opacity zero make a widget non-interactive?** Not necessarily; hit testing and semantics may need separate handling.
6. **What does a ticker warning usually suggest?** A ticker-backed animation was not stopped or disposed with its owner.
7. **Why prefer `AnimatedBuilder` over a manual listener sometimes?** It manages the listenable-to-subtree relationship declaratively.

## Try It Yourself

1. Intentionally put controller creation in `build`, observe the problem, then restore the lifecycle-safe version.
2. Add a second tween for scale.
3. Make the animation reverse while in flight.
4. Hide the panel and ensure it does not intercept taps.
5. Navigate away during motion and inspect for ticker warnings.

## Challenge Project

Build a reusable animated notice widget with show, hide, reverse, and dispose-safe behavior. It should support a reduced-motion preference, avoid intercepting input when hidden, and expose a clear semantic label.

## Quick Revision Sheet

- `build` can run many times; it is not initialization.
- Own a controller in `State`; create it in `initState` and dispose in `dispose`.
- Controller progress can be mapped with a tween and curve.
- A widget must listen to a changing value to update visually.
- Remove manually registered listeners.
- Define interruption, hit testing, and hidden-state behavior.
- Keep animated rebuild scope small and profile expensive effects.
