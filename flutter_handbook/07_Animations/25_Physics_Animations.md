# Physics Animations

## Introduction

A physics animation moves a widget according to a simple model of real-world motion. Instead of saying “move from here to there in exactly 300 milliseconds,” you describe physical properties such as a spring’s stiffness and damping, or an object’s initial speed and friction.

This is useful when motion should feel responsive and natural. A card released after a drag can spring back. A panel can coast after a swipe and gradually stop. A scrollable list can slow down after a fast flick.

Flutter provides these building blocks in `package:flutter/physics.dart`. A physics simulation calculates a value as time passes. An `AnimationController` asks that simulation for a new value on each animation frame and notifies widgets that listen to it.

Physics does not mean Flutter is simulating every real-world force. These are practical mathematical models designed to create convincing interface motion.

## Why this animation exists

Fixed-duration animations are predictable, but they do not always respond naturally to user input. If a user drags an object and lets go, its release speed matters. A slow release should feel different from a fast flick.

Physics-based animations let motion respond to:

- The current position of an object.
- Its velocity when the gesture ends.
- The spring or friction properties chosen by the developer.
- The destination or boundary the object should settle at.

This makes gestures feel connected to the interface instead of starting an unrelated, pre-timed animation after every interaction.

## Real-world analogy

Imagine pulling a small toy attached to a spring. The farther you pull it, the more the spring pulls it back. If you release it gently, it returns with a small bounce. If you pull and release it quickly, it may overshoot and wobble before settling.

A spring simulation models that behavior with:

- **Mass**: how heavy the moving object feels.
- **Stiffness**: how strongly the spring pulls toward its destination.
- **Damping**: how quickly the bouncing or wobbling loses energy.

Friction is another familiar example. A puck sliding across a table keeps moving after you stop pushing it, but gradually slows down.

## Internal Working

Flutter displays an app through a frame pipeline. In simplified order, Flutter handles scheduled work, builds or updates widgets, lays out render objects, paints changed content, and submits a frame for display. Animation is coordinated with that frame schedule; it does not normally run a tight loop that tries to draw as fast as possible.

1. An input event, such as a drag release, starts a simulation.
2. An `AnimationController` uses a **ticker** to request frame callbacks.
3. The ticker measures elapsed time. On each frame, the controller asks the simulation for its value at that elapsed time.
4. The controller's value changes and listeners are notified.
5. An `AnimatedBuilder` or another listener marks the relevant widget subtree for rebuilding.
6. Flutter lays out or paints what changed and displays the new frame.
7. The simulation reports that it is done when it has settled. The controller stops ticking and its status changes.

**Vsync** means synchronizing animation updates with the display's frame schedule. A `vsync` provider, commonly `this` from a `State` using `SingleTickerProviderStateMixin`, lets the controller avoid requesting frames when its owning screen is not active for rendering. This helps avoid wasted work.

A rebuild is not the same as repainting the entire screen. Flutter can rebuild a small widget subtree, and rendering optimizations can limit which render objects need layout or paint. For example, changing a `Transform.translate` generally changes painting without changing the child's layout position. An animation that changes width or height may require layout work and can be more expensive.

Animation lifecycle matters:

- Create the controller in `initState`, not in `build`.
- Start or retarget the simulation in response to an event.
- Dispose of the controller in `dispose`.
- Avoid starting a new simulation on every build.
- Stop or replace an in-progress motion when new user input takes control.

## Syntax

Import Flutter's physics classes:

```dart
import 'package:flutter/physics.dart';
```

A spring simulation is commonly started like this:

```dart
final spring = SpringDescription(
  mass: 1,
  stiffness: 500,
  damping: 25,
);

final simulation = SpringSimulation(
  spring,
  currentPosition,
  targetPosition,
  initialVelocity,
);

controller.animateWith(simulation);
```

`SpringSimulation` needs the spring description, starting value, destination, and starting velocity. `AnimationController.animateWith` advances the controller according to the simulation. Unlike `animateTo`, which uses a duration and curve, `animateWith` lets the physical model determine when it has settled.

A friction simulation can be started in the same way:

```dart
controller.animateWith(
  FrictionSimulation(
    0.08, // drag coefficient
    controller.value, // starting position
    velocity, // starting velocity
  ),
);
```

The friction simulation models motion that slows down over time. If the moving object must stop at a boundary, check its predicted destination or use an appropriate bounded interaction design; a friction simulation by itself does not define a wall that prevents it from moving past that boundary.

## Complete Example 1

This runnable app lets you drag a card horizontally and release it. The card springs back to the center. The release velocity is passed to the spring, so a quick flick produces different motion from a gentle release.

```dart
import 'package:flutter/material.dart';
import 'package:flutter/physics.dart';

void main() {
  runApp(const PhysicsDemoApp());
}

class PhysicsDemoApp extends StatelessWidget {
  const PhysicsDemoApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Physics Animation',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
        useMaterial3: true,
      ),
      home: const PhysicsCardPage(),
    );
  }
}

class PhysicsCardPage extends StatefulWidget {
  const PhysicsCardPage({super.key});

  @override
  State<PhysicsCardPage> createState() => _PhysicsCardPageState();
}

class _PhysicsCardPageState extends State<PhysicsCardPage>
    with SingleTickerProviderStateMixin {
  late final AnimationController _position;

  @override
  void initState() {
    super.initState();
    _position = AnimationController.unbounded(vsync: this);
  }

  void _returnToCenter(double velocity) {
    const spring = SpringDescription(
      mass: 1,
      stiffness: 500,
      damping: 25,
    );

    _position.animateWith(
      SpringSimulation(
        spring,
        _position.value,
        0,
        velocity,
      ),
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
      appBar: AppBar(title: const Text('Physics Card')),
      body: Center(
        child: GestureDetector(
          onPanUpdate: (details) {
            _position.value += details.delta.dx;
          },
          onPanEnd: (details) {
            _returnToCenter(details.velocity.pixelsPerSecond.dx);
          },
          child: AnimatedBuilder(
            animation: _position,
            child: const _PhysicsCard(),
            builder: (context, child) {
              return Transform.translate(
                offset: Offset(_position.value, 0),
                child: child,
              );
            },
          ),
        ),
      ),
    );
  }
}

class _PhysicsCard extends StatelessWidget {
  const _PhysicsCard();

  @override
  Widget build(BuildContext context) {
    return Card(
      elevation: 8,
      color: Colors.indigo,
      child: SizedBox(
        width: 220,
        height: 140,
        child: Center(
          child: Text(
            'Drag and release',
            style: Theme.of(context).textTheme.titleMedium?.copyWith(
                  color: Colors.white,
                  fontWeight: FontWeight.bold,
                ),
          ),
        ),
      ),
    );
  }
}
```

### Line-by-line explanation

- The Material and physics imports provide the UI widgets and the simulation classes.
- `main` starts the app. `PhysicsDemoApp` provides the `MaterialApp`, theme, and page.
- `PhysicsCardPage` is stateful because the card's position changes over time.
- `SingleTickerProviderStateMixin` supplies vsync for this page's one controller.
- `_position` is an unbounded controller. Its value is a horizontal distance in logical pixels, so it can be negative or positive and does not get clamped to the usual `0.0`–`1.0` range.
- `initState` creates the controller once. Creating it in `build` would create a new controller repeatedly and break the animation lifecycle.
- `_returnToCenter` describes a spring and starts a simulation from the current position to `0`. The supplied velocity is in logical pixels per second, matching the position's units.
- `dispose` releases the ticker-backed controller when the page is removed.
- `GestureDetector` receives pointer movement. `onPanUpdate` adds the horizontal drag delta to the current position, so the card follows the finger.
- `onPanEnd` reads the horizontal release velocity and passes it to the spring. A near-zero velocity produces a calmer return; a stronger flick can produce a more energetic return.
- `AnimatedBuilder` listens to the controller and rebuilds only the transform portion. Its `child` is built once and reused on animation frames.
- `Transform.translate` paints the card at the controller's current horizontal offset without changing its layout slot.
- `_PhysicsCard` contains the card's fixed size, color, elevation, and label.
- The null-aware text-style update (`?.copyWith`) keeps the example safe if the theme does not provide that text style.

### Widget tree

```text
MaterialApp
└── PhysicsCardPage
    └── Scaffold
        ├── AppBar
        └── Center
            └── GestureDetector
                └── AnimatedBuilder
                    └── Transform.translate
                        └── _PhysicsCard
                            └── Card
                                └── SizedBox
                                    └── Center
                                        └── Text
```

### Execution flow

```text
Finger moves ──> onPanUpdate ──> controller value changes
                                      │
                                      v
                               AnimatedBuilder
                                      │
                                      v
                               card is translated

Finger releases ──> onPanEnd ──> SpringSimulation
                                      │
                           ticker requests frames
                                      │
                         simulation updates position
                                      │
                         card springs toward center
                                      │
                            simulation settles
```

### ASCII phone preview

```text
+---------------------------+
|       Physics Card        |
+---------------------------+
|                           |
|       +-------------+     |
|       | Drag and    |     |
|       | release     |     |
|       +-------------+     |
|                           |
+---------------------------+
```

During a drag, the card moves horizontally with the finger. After release, it springs back to the center.

### Example explanation

The `AnimationController` is being used as a numeric position, not as a normalized progress from zero to one. That is why the example uses `AnimationController.unbounded`. Both the value and velocity are in logical pixels and logical pixels per second respectively, so they can be used directly with `Transform.translate` and the spring simulation.

The important interaction detail is that the gesture owns the position while the finger is moving, and the simulation owns it after the finger is released. If another drag begins during the spring, assigning a new controller value stops the old trajectory from controlling the card and lets the user's finger take over.

If the controller's `vsync` is removed, the controller cannot be constructed with the normal ticker-aware API. If `dispose` is removed, the ticker can remain active after the page is gone and Flutter may report a ticker leak. If the `AnimatedBuilder` is removed and nothing listens to the controller, the value can change without this UI updating.

**Common mistakes in this example**

- Using `AnimationController` with fixed `0.0` and `1.0` bounds for a pixel position can clamp movement unexpectedly. Use an unbounded controller or deliberately convert pixels to a normalized range.
- Passing gesture velocity in pixels per second while using a normalized position creates mismatched units. Convert velocity to normalized units if the controller represents normalized progress.
- Starting the spring inside `build` restarts it whenever the widget rebuilds. Start it from the gesture event instead.
- Omitting the controller's `dispose` can leave a ticker active.
- Rebuilding the card on every animation tick wastes work. Passing it as `AnimatedBuilder.child` avoids rebuilding the static card subtree.

**Professional use case:** A swipeable product card can follow the user's finger and return with a spring when the gesture is not strong enough to dismiss it. For a dismissible card, use the velocity and distance to choose between returning to the start and animating off-screen.

## Complete Example 2

This example models a tile that coasts after a flick. The controller represents horizontal distance in logical pixels. A friction simulation gradually reduces the velocity.

```dart
void coastAfterFlick(double velocity) {
  _motion.animateWith(
    FrictionSimulation(
      0.12,
      _motion.value,
      velocity,
    ),
  );
}
```

Use it in a stateful widget with an unbounded `AnimationController` and an `AnimatedBuilder`, just as in Example 1. Call `coastAfterFlick(details.velocity.pixelsPerSecond.dx)` from `onPanEnd`.

The first argument, `0.12`, is the drag coefficient: larger values slow the object more quickly, while smaller values allow it to coast farther. The second argument is the starting position. The third is the initial velocity. The simulation produces successive positions; the controller and ticker deliver those values to listeners.

**Widget tree:** `GestureDetector → AnimatedBuilder → Transform.translate → tile`.

**Execution flow:** a flick ends → the gesture supplies velocity → `FrictionSimulation` calculates position over time → the controller notifies the builder → the tile moves while slowing.

```text
+---------------------------+
|       Flick the tile      |
|                           |
|             [ TILE ] ---> |
|                 slows    |
+---------------------------+
```

This snippet focuses on the simulation call, so it relies on the controller and UI listener being set up as in Example 1. If you omit the initial velocity, the tile has no flick momentum. If you expect friction alone to stop the tile at an edge, it may travel beyond that edge; friction slows motion but is not a boundary constraint.

## Complete Example 3

This example shows how to use a spring's predicted resting point when deciding whether to settle a swipeable item at one of two positions.

```dart
void settleToNearestSnap({
  required AnimationController controller,
  required double velocity,
  required double midpoint,
  required double rightSnap,
}) {
  final projectedPosition = controller.value + velocity * 0.15;
  final target = projectedPosition >= midpoint ? rightSnap : 0.0;

  const spring = SpringDescription(
    mass: 1,
    stiffness: 420,
    damping: 32,
  );

  controller.animateWith(
    SpringSimulation(spring, controller.value, target, velocity),
  );
}
```

The helper estimates where a moving item is heading by projecting its current position forward by a short interval. It chooses the left or right snap point using the midpoint, then starts a spring from the controller's actual position with the actual release velocity. The `0.15` value is a product-tuning choice in seconds; adjust it based on interaction testing. The controller should represent position in logical pixels, and `rightSnap` and `midpoint` must use those same units.

In production code, make the snap positions and selection rule explicit for the widget's actual size and direction. For example, a horizontal pager may use its page width rather than a hard-coded `300`, and right-to-left layouts may need a direction-aware comparison.

**Widget tree:** `GestureDetector → stateful swipe item → AnimatedBuilder → Transform.translate → content`.

**Execution flow:** release gesture → estimate the direction of travel from position and velocity → select a snap destination → animate the controller toward that destination.

```text
Start position             Midpoint             End positions
      0 ----------------------|-------------------- 300
                              ^
              projected position selects a side
```

The projected position makes the decision consider both position and velocity. A fast swipe can select the farther side even when the current position has not crossed the midpoint. The projection is a simple heuristic, not a prediction of the spring's mathematical resting point. Do not use it as a replacement for a complete gesture policy without testing slow drags, reverse flicks, and both layout directions.

## Advanced Example

For a production swipe interaction, separate gesture handling, target selection, and rendering. Use layout constraints to determine the available travel distance, use the release velocity to choose a destination, and animate the controller with a spring. If a widget must remain within a region, clamp the gesture's displayed position or select a valid destination; an unbounded simulation does not enforce application-specific limits for you.

One useful spring quality check is its damping ratio:

```text
damping ratio = damping / (2 × sqrt(stiffness × mass))
```

With the Flutter spring parameters:

- A ratio below `1` is underdamped and can oscillate.
- A ratio around `1` is critically damped and returns quickly without oscillation.
- A ratio above `1` is overdamped and returns without oscillation, usually more slowly.

This is a guide, not a universal design rule. The best parameters depend on the scale of the motion and the feeling the interface needs. Tune using real device interaction, not only by watching a static preview.

If the interaction can be interrupted, let a new gesture take control immediately. Avoid queuing multiple old simulations that continue to update the same controller. When a simulation should hand off to another phase, listen for status changes or use the `TickerFuture` returned by the controller where appropriate.

## Under the Hood

`SpringSimulation` and `FrictionSimulation` are examples of `Simulation` implementations. A simulation is asked for a position at an elapsed time, and can also report velocity and whether it is done. The animation controller does not need to know the spring's mathematical details; it repeatedly samples the simulation and publishes the resulting numeric value.

`animateWith` starts a controller simulation. The controller's ticker supplies elapsed time, and each tick updates the controller value. The value change notifies listeners such as `AnimatedBuilder`. When the simulation says it is done, the controller stops scheduling ticks and completes its animation future.

`vsync` connects the ticker to the lifecycle of the widget tree and display. A `TickerProvider` such as `SingleTickerProviderStateMixin` helps ensure that frames are coordinated with the screen rather than generated by an uncontrolled timer.

The widget system and rendering system have distinct jobs:

- A controller value change notifies animation listeners.
- A listening builder may rebuild a widget subtree.
- Layout runs when geometry or constraints require it.
- Paint runs when a render object's visual output changes.
- The resulting frame is composited and displayed.

Choosing a transform for movement often avoids repeated layout changes. It does not make every animation free: large, complex subtrees, expensive effects, and unnecessary rebuilds can still affect frame time.

## Performance Notes

- Keep animated rebuilds small. Use `AnimatedBuilder.child` for widgets that do not depend on the changing value.
- Prefer paint-only changes such as `Transform` when they express the desired motion. Use size or layout animation only when the surrounding layout must actually change.
- Dispose every controller and avoid leaving ticker-backed work active after its screen is removed.
- Avoid creating controllers, simulations, or large widget subtrees on every build.
- Test on representative physical devices. Smoothness depends on the cost of the whole frame, not just the simulation calculation.
- Avoid expensive clipping, shadows, opacity layers, and image effects on large moving regions unless needed.
- Use Flutter DevTools' Performance view to inspect slow frames instead of guessing which animation is expensive.
- Respect reduced-motion or accessibility requirements where the product supports them; an interaction should remain understandable if motion is reduced.

## Best Practices

- Choose a simulation that matches the behavior: spring for returning or snapping, friction for coasting.
- Keep units consistent. Position and velocity must use the same coordinate system and corresponding units.
- Select spring mass, stiffness, and damping intentionally; change one at a time while tuning.
- Use gesture velocity when it improves the interaction, but test slow releases and direction changes too.
- Define explicit boundaries and snap targets for constrained interfaces.
- Let user input interrupt an animation naturally.
- Keep animation state in a controller owned by the appropriate `State` object.
- Explain the destination in code with named constants or helper methods rather than scattering magic numbers.
- Check that the interface remains usable when animation is disabled, interrupted, or completed immediately.

## Common Mistakes

- **Expecting a fixed duration:** physics simulations end when they settle, so their duration varies with the start, velocity, and simulation parameters.
- **Mismatched units:** a controller value in normalized units cannot directly use a velocity in pixels per second.
- **Forgetting bounds:** a friction simulation can keep moving past an edge. Add boundary handling or use a suitable constrained interaction.
- **Using a bounded controller for free movement:** values outside its bounds may be clamped. Use an unbounded controller or map values deliberately.
- **Recreating controllers in `build`:** this resets animation state and can leak tickers.
- **Forgetting vsync or disposal:** use a ticker provider and dispose controllers in `dispose`.
- **Starting animations during build:** build can run many times. Start or retarget motion in event handlers or lifecycle methods.
- **Rebuilding too much:** pass static children through `AnimatedBuilder.child`.
- **Ignoring interruption:** a user's next gesture should not feel as if an old animation is fighting it.
- **Tuning by one device only:** motion scale, refresh rate, and frame workload vary. Test realistic devices and interaction speeds.

## Debugging

- If the widget does not move, confirm that a listener such as `AnimatedBuilder` is observing the same controller that the simulation updates.
- If movement stops at an unexpected value, check controller bounds and whether the controller is bounded or unbounded.
- If the release motion feels wrong, verify the velocity's sign and units. `pixelsPerSecond.dx` is horizontal velocity; `.dy` is vertical.
- If the card jumps on release, check that drag updates and the simulation use the same position coordinate.
- If the motion does not settle, inspect spring damping, the simulation's completion behavior, and whether new gestures repeatedly restart it.
- If Flutter reports an active ticker after a widget is disposed, ensure its controller is disposed and not shared with an unrelated widget.
- If frames are slow, profile build, layout, and paint in DevTools. A physics equation is usually not the only possible source of a slow frame.
- Temporarily display the controller value in a `Text` widget or log gesture position and velocity to check the data before tuning the spring.

## Interview Questions

1. **What is a physics-based animation?**  
   An animation whose values are calculated from a physical model such as a spring or friction, rather than only from a fixed duration and curve.

2. **How does a spring simulation differ from `animateTo`?**  
   `animateTo` moves toward a target over a chosen duration and curve. A spring simulation uses spring parameters, initial position, and initial velocity, and settles according to the model.

3. **What do mass, stiffness, and damping describe?**  
   Mass describes perceived inertia, stiffness describes the strength of the spring, and damping describes how quickly oscillation loses energy.

4. **Why does release velocity matter?**  
   It determines the initial motion after release, allowing a fast flick and a gentle release to behave differently.

5. **What is a ticker?**  
   A ticker calls its callback in coordination with animation frames and provides elapsed time to drive updates.

6. **What does `vsync` do?**  
   It connects an animation ticker to a `TickerProvider`, coordinating updates with the widget/display lifecycle and avoiding unnecessary off-screen ticking.

7. **When would you use `AnimationController.unbounded`?**  
   When the controller represents a value that may move outside the usual `0.0`–`1.0` range, such as a position in logical pixels.

8. **Does a physics simulation automatically enforce a visual boundary?**  
   No. The app must select bounded targets, clamp input, or otherwise design the interaction to respect its limits.

9. **How can animation updates cause widget rebuilding?**  
   A controller notifies its listeners when its value changes. A listening builder marks its dependent subtree for rebuild.

10. **Why pass `child` to `AnimatedBuilder`?**  
    A static child can be reused rather than rebuilt on every animation tick.

## Try It Yourself

1. Change the spring's stiffness while keeping its mass and damping constant. Observe how quickly it returns.
2. Increase damping until the card returns with little or no visible wobble.
3. Log the release velocity and compare a slow release with a quick flick.
4. Change the card to move vertically and use `details.delta.dy` and `details.velocity.pixelsPerSecond.dy`.
5. Add a maximum drag distance and decide what should happen when the finger reaches it.
6. Replace the spring with friction and observe why coasting needs explicit boundary behavior.
7. Use an `AnimatedBuilder` child and then temporarily remove it to compare which widgets rebuild.

## Challenge Project

Build a swipeable notification card:

- The card follows a horizontal drag.
- A short drag springs back to its starting point.
- A sufficiently long drag or strong flick dismisses it.
- The dismissal animation respects the direction of the swipe.
- A dismiss button provides a non-gesture alternative.
- A callback removes the dismissed card from the list only after the exit motion completes.
- A new gesture interrupts a return animation without causing a position jump.

Before coding, write down the card's coordinate units, allowed travel range, and the rules for choosing a snap or dismissal target. Test slow drags, quick flicks, reverse flicks, and repeated gestures.

## Quick Revision Sheet

- Physics animations use `Simulation` implementations such as `SpringSimulation` and `FrictionSimulation`.
- `AnimationController.animateWith(simulation)` advances the controller according to a simulation.
- Spring inputs include mass, stiffness, damping, start, destination, and initial velocity.
- Friction models coasting that slows down; it does not automatically impose an edge boundary.
- A ticker requests animation frame callbacks; vsync synchronizes those updates with the display lifecycle.
- Controller changes notify listeners, which can rebuild dependent widgets and cause rendering work.
- Keep position and velocity units consistent.
- Use `AnimationController.unbounded` for values that are not limited to `0.0`–`1.0`.
- Create controllers once, dispose them, and let new gestures interrupt old motion.
- Profile the complete frame and keep animated widget subtrees small.
