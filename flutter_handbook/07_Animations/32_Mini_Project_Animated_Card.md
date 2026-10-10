# Mini Project: Animated Card

## Introduction

In this project, we build a card that responds to taps with a coordinated scale, color, and elevation change. The interaction is intentionally small, but it demonstrates a pattern used for selectable tiles, product previews, and dashboard cards.

The card uses an explicit controller because its properties should move together on one timeline and the motion should reverse smoothly if tapped again before it finishes.

## Why this concept exists

Static cards can feel disconnected from interaction. A restrained animation confirms that a tap was received and makes selection easier to notice. Coordinating properties also gives the interface a more polished, intentional response than changing a single property abruptly.

## Real-world analogy

Pressing a physical button makes it move slightly and gives feedback that the action registered. The animated card uses a visual “press” and “release” to communicate the same acknowledgement without pretending to be a real mechanical button.

## Internal Working

The state owns one `AnimationController` and creates scale, color, and elevation animations from it. A button tap changes the controller's direction. The controller's ticker updates one timeline, and each tween maps the same progress into a different property.

`AnimatedBuilder` listens to the controller and rebuilds the card's visual wrapper. The static icon and text are supplied as the builder child, so the repeated updates do not reconstruct that content. Flutter then paints the changed transform, decoration, and shadow. The controller is disposed when the page is removed.

## Syntax

```dart
final scale = Tween<double>(begin: 1, end: 0.94).animate(controller);
final color = ColorTween(
  begin: Colors.white,
  end: Colors.lightBlue.shade50,
).animate(controller);
```

Multiple tweens can share one controller. The endpoints are design choices; keep the scale subtle and verify that the card's text remains legible against both colors.

## Complete Example 1

Run this app to tap the card and toggle its selected animation. Tapping during motion reverses or continues toward the other endpoint based on the current controller status.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const AnimatedCardApp());

class AnimatedCardApp extends StatelessWidget {
  const AnimatedCardApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      theme: ThemeData(useMaterial3: true, colorSchemeSeed: Colors.blue),
      home: const AnimatedCardPage(),
    );
  }
}

class AnimatedCardPage extends StatefulWidget {
  const AnimatedCardPage({super.key});

  @override
  State<AnimatedCardPage> createState() => _AnimatedCardPageState();
}

class _AnimatedCardPageState extends State<AnimatedCardPage>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;
  late final Animation<double> _scale;
  late final Animation<Color?> _color;
  late final Animation<double> _elevation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 220),
    );
    final curved = CurvedAnimation(
      parent: _controller,
      curve: Curves.easeOutCubic,
      reverseCurve: Curves.easeInCubic,
    );
    _scale = Tween<double>(begin: 1, end: 0.96).animate(curved);
    _color = ColorTween(
      begin: Colors.white,
      end: Colors.lightBlue.shade50,
    ).animate(curved);
    _elevation = Tween<double>(begin: 3, end: 10).animate(curved);
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
      appBar: AppBar(title: const Text('Animated card')),
      body: Center(
        child: Padding(
          padding: const EdgeInsets.all(24),
          child: AnimatedBuilder(
            animation: _controller,
            child: const _CardContents(),
            builder: (context, child) {
              return Transform.scale(
                scale: _scale.value,
                child: Card(
                  color: _color.value,
                  elevation: _elevation.value,
                  clipBehavior: Clip.antiAlias,
                  child: InkWell(
                    onTap: _toggle,
                    child: SizedBox(
                      width: 300,
                      height: 190,
                      child: child,
                    ),
                  ),
                ),
              );
            },
          ),
        ),
      ),
    );
  }
}

class _CardContents extends StatelessWidget {
  const _CardContents();

  @override
  Widget build(BuildContext context) {
    return const Padding(
      padding: EdgeInsets.all(20),
      child: Row(
        children: [
          CircleAvatar(
            radius: 28,
            child: Icon(Icons.auto_awesome),
          ),
          SizedBox(width: 16),
          Expanded(
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(
                  'Design ideas',
                  style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
                ),
                SizedBox(height: 8),
                Text('Tap this card to select it.'),
              ],
            ),
          ),
        ],
      ),
    );
  }
}
```

### Line-by-line explanation

- `main` starts a `MaterialApp` with a blue-seeded Material 3 theme.
- `AnimatedCardPage` is stateful because it owns the animation controller.
- `SingleTickerProviderStateMixin` supplies vsync for the one controller.
- `_controller` owns the shared 220 ms timeline.
- `_scale`, `_color`, and `_elevation` are typed animations derived from that timeline.
- `initState` constructs the controller once and creates a curved view with separate forward and reverse curves.
- Each `Tween` converts progress into a property-specific range.
- `dispose` releases the controller and ticker.
- `_toggle` reverses a completed selection animation or forwards otherwise.
- `Scaffold`, `AppBar`, `Center`, and `Padding` create the screen and safe card spacing.
- `AnimatedBuilder` listens to the controller and receives the static card contents as `child`.
- `Transform.scale` provides subtle press feedback without changing the card's layout slot.
- `Card` consumes the changing color and elevation. `clipBehavior` clips the ink ripple to the card shape.
- `InkWell` makes the whole card tappable and displays Material ink feedback.
- `SizedBox` defines the card's content dimensions.
- `_CardContents` is a separate stateless widget containing the avatar, title, and supporting text.
- Its `Row`, `Expanded`, and `Column` keep text inside the card's horizontal space.

### Widget tree

```text
MaterialApp
└── AnimatedCardPage
    └── Scaffold
        ├── AppBar
        └── Center
            └── Padding
                └── AnimatedBuilder
                    └── Transform.scale
                        └── Card
                            └── InkWell
                                └── SizedBox
                                    └── _CardContents
```

### Execution flow

```text
Tap card -> _toggle -> controller moves forward or reverse
                             |
                             +--> scale tween
                             +--> color tween
                             +--> elevation tween
                                      |
                                      v
                               AnimatedBuilder
                                      |
                                      v
                                  card paints
```

### ASCII phone preview

```text
+---------------------------+
|      Animated card        |
+---------------------------+
|                           |
|  +---------------------+  |
|  | ( * ) Design ideas  |  |
|  |       Tap this card |  |
|  +---------------------+  |
|                           |
+---------------------------+
```

### Example explanation

One controller synchronizes all three properties, so the card's scale, color, and elevation share the same progress. The tap can reverse the motion in progress without waiting for the current animation to finish. `InkWell` provides an accessible Material interaction model, while the animation is additional feedback rather than the only indication of action.

If the controller is removed, the animations have no timeline. If `AnimatedBuilder.child` is removed, the static card content is rebuilt each tick. If `clipBehavior` is removed, the ink ripple may draw beyond the card's rounded shape. If `dispose` is removed, Flutter may report an active ticker after navigating away.

**Common mistakes:** Keep press scaling subtle to avoid layout jitter or a distorted appearance. Verify the color contrast in both states. Do not use elevation alone to communicate selection.

**Professional use case:** Product category cards, selectable plan cards, and dashboard shortcuts can use a small coordinated press/selection response.

## Complete Example 2

An implicit version is appropriate if the card simply changes selected styling and does not need a manually controlled timeline.

```dart
AnimatedContainer(
  duration: const Duration(milliseconds: 200),
  curve: Curves.easeOut,
  padding: const EdgeInsets.all(20),
  decoration: BoxDecoration(
    color: selected ? Colors.blue.shade50 : Colors.white,
    borderRadius: BorderRadius.circular(20),
    boxShadow: [
      BoxShadow(
        blurRadius: selected ? 14 : 5,
        color: Colors.black12,
      ),
    ],
  ),
  child: child,
)
```

When `selected` changes, the container interpolates supported decoration changes. The child is passed through, and the state owner only needs to update the boolean.

**Widget tree:** `AnimatedContainer → card contents`.  
**Flow:** tap → boolean changes → container receives new target → decoration transitions.

Use the explicit form when values must be synchronized with a controller, reversed from a particular point, or coordinated with another animation.

## Complete Example 3

Use a `Hero` when tapping the card navigates to a details page and the same visual element should appear to travel between routes.

```dart
Hero(
  tag: 'design-card-art',
  child: const Icon(Icons.auto_awesome, size: 48),
)
```

Place the same unique tag on the destination route's corresponding widget. Both routes must be in a navigator transition for the hero flight to occur.

**Widget tree:** source route `Hero → icon`; destination route `Hero → matching image or icon`.  
**Flow:** push route → navigator matches tags → overlay flight animates between route geometries.

If the tags do not match or are duplicated in one route, the expected shared-element transition cannot be formed.

## Advanced Example

To make the card part of a list, store selection state by stable item ID rather than by list position. If an item is removed during its exit motion, wait for the animation to finish before removing it from the model. Add semantics that describe the selected state, and provide keyboard or switch-access navigation through normal controls.

## Under the Hood

The controller tick notifies the animations. Their tweens calculate values, and the builder updates the `Transform` and `Card` configuration. Flutter performs the needed paint/compositing work; card elevation can involve shadow rasterization, so keep effects modest and profile dense lists.

## Performance Notes

- Reuse static card contents through the builder's `child`.
- Avoid many simultaneous animated shadows in long lists.
- Use implicit animation for simple selected-state changes.
- Keep list items keyed by stable identity if their state moves.
- Profile on lower-end devices if many cards animate together.

## Best Practices

- Use motion as feedback, not as the only communication of selection.
- Keep scale and elevation changes restrained.
- Provide a semantic label and selected state.
- Make tappable area large enough for touch.
- Ensure reverse and interruption behavior is intentional.
- Use stable keys in animated collections.

## Common Mistakes

- Animating every visible card continuously.
- Losing selection state when list ordering changes.
- Using low-contrast selected colors.
- Making only a tiny icon tappable unintentionally.
- Forgetting ripple clipping or accessibility semantics.
- Using explicit controllers for a simple property transition without a need.

## Debugging

- Confirm taps reach the `InkWell` and update animation direction.
- Check controller status and tween outputs.
- If the card jumps, inspect whether a new controller or key is created.
- If list cards exchange animation state, use stable keys.
- If shadows stutter, profile raster cost and reduce simultaneous elevation animation.

## Interview Questions

1. **Why share one controller for scale, color, and elevation?** It synchronizes properties on one timeline.
2. **Why pass card contents as `AnimatedBuilder.child`?** To reuse static descendants.
3. **When is `AnimatedContainer` sufficient?** When selection only changes target properties without needing timeline control.
4. **What does `InkWell` add?** A Material tap recognizer and ink feedback.
5. **Why use stable keys in animated lists?** To preserve item identity and associated state during reorder/removal.
6. **What is a Hero transition used for?** Animating a shared visual element between routes.

## Try It Yourself

1. Add a checkmark that fades in on selection.
2. Add an unselected/selected semantic state.
3. Change the card to use `AnimatedContainer` and compare complexity.
4. Add a second card with independent selection.
5. Test rapid repeated taps.

## Challenge Project

Build a list of selectable subscription cards. Selecting one card should animate its border, background, and checkmark while deselecting the previous choice. Preserve correct selection during list updates and expose the selection to assistive technology.

## Quick Revision Sheet

- Use one controller for synchronized card properties.
- Use `AnimatedContainer` for simple target-state changes.
- Pass static children to `AnimatedBuilder.child`.
- `InkWell` provides tap and Material ripple feedback.
- Use stable keys for animated collections.
- Keep motion subtle, legible, and accessible.
