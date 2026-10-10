# Mini Project: Instagram-Like Button

## Introduction

This project builds a familiar animated like button. Tapping toggles the liked state, changes the heart color, briefly enlarges the icon, and announces the current state to assistive technology.

It is inspired by a common social interaction pattern, not a copy of any particular service's branding or interface. The same approach works for favorites, bookmarks, and reactions.

## Why this concept exists

A toggle needs to communicate both the new state and that the user's tap was received. Color change communicates state; a brief scale pulse adds tactile visual feedback. The animation should be quick and should not make the button harder to tap.

## Real-world analogy

Pressing a physical switch gives both a new position and a little movement under your finger. A like button uses a small visual pulse to signal the action while its filled/unfilled heart shows whether the state is on or off.

## Internal Working

The page stores `isLiked` as ordinary application state. The tap updates that state and drives a short controller animation. A `ColorTween` changes the icon color along the same timeline, while a `TweenSequence` creates a grow-and-return scale pulse.

The controller ticker updates values, and `AnimatedBuilder` rebuilds only the icon's visual transform. A `Semantics` widget exposes the button label and selected state. The controller is disposed with the page.

## Syntax

```dart
GestureDetector(
  onTap: onToggle,
  child: ScaleTransition(
    scale: scaleAnimation,
    child: Icon(Icons.favorite, color: color),
  ),
)
```

For production buttons, prefer a Material button or `IconButton` for focus, semantics, and interaction defaults; layer the animation into its child.

## Complete Example 1

This app has a reusable heart button with an animated color transition, short pulse, and accessible selected state.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const LikeButtonApp());

class LikeButtonApp extends StatelessWidget {
  const LikeButtonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      theme: ThemeData(useMaterial3: true, colorSchemeSeed: Colors.pink),
      home: const LikeButtonPage(),
    );
  }
}

class LikeButtonPage extends StatefulWidget {
  const LikeButtonPage({super.key});

  @override
  State<LikeButtonPage> createState() => _LikeButtonPageState();
}

class _LikeButtonPageState extends State<LikeButtonPage>
    with SingleTickerProviderStateMixin {
  late final AnimationController _pulseController;
  late final Animation<double> _scale;
  late final Animation<Color?> _color;
  bool _isLiked = false;

  @override
  void initState() {
    super.initState();
    _pulseController = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 420),
    );
    _scale = TweenSequence<double>([
      TweenSequenceItem(
        tween: Tween<double>(begin: 1, end: 1.35)
            .chain(CurveTween(curve: Curves.easeOut)),
        weight: 55,
      ),
      TweenSequenceItem(
        tween: Tween<double>(begin: 1.35, end: 1)
            .chain(CurveTween(curve: Curves.easeIn)),
        weight: 45,
      ),
    ]).animate(_pulseController);
    _color = ColorTween(
      begin: Colors.blueGrey,
      end: Colors.pink,
    ).animate(_pulseController);
  }

  @override
  void dispose() {
    _pulseController.dispose();
    super.dispose();
  }

  void _toggleLike() {
    setState(() => _isLiked = !_isLiked);
    if (_isLiked) {
      _pulseController.forward();
    } else {
      _pulseController.reverse();
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Animated like button')),
      body: Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            Semantics(
              button: true,
              label: _isLiked ? 'Unlike this item' : 'Like this item',
              selected: _isLiked,
              child: IconButton(
                tooltip: _isLiked ? 'Unlike' : 'Like',
                onPressed: _toggleLike,
                icon: AnimatedBuilder(
                  animation: _pulseController,
                  builder: (context, child) {
                    return Transform.scale(
                      scale: _scale.value,
                      child: Icon(
                        _isLiked ? Icons.favorite : Icons.favorite_border,
                        color: _color.value,
                        size: 52,
                      ),
                    );
                  },
                ),
              ),
            ),
            const SizedBox(height: 12),
            Text(_isLiked ? 'Liked' : 'Tap the heart'),
          ],
        ),
      ),
    );
  }
}
```

### Line-by-line explanation

- `main` starts the app. `MaterialApp` provides the theme and page.
- `LikeButtonPage` owns the selected state and animation controller.
- `SingleTickerProviderStateMixin` provides vsync for the pulse controller.
- `_pulseController` defines a 420 ms one-shot timeline.
- `_scale` uses two weighted tween stages: grow quickly, then return to normal size. `_color` interpolates from grey-blue to pink on the same timeline.
- `_isLiked` is the durable toggle state. It is separate from temporary animation progress.
- `initState` creates the controller and sequence once.
- `dispose` releases the controller.
- `_toggleLike` changes the selected state and moves the controller forward for like or backward for unlike.
- `Semantics` announces the action and selected state for assistive technology.
- `IconButton` provides a Material button hit target, focus behavior, and tooltip.
- The nested `AnimatedBuilder` listens to the pulse controller and transforms the icon.
- The icon glyph changes between outlined and filled hearts based on state.
- The text below mirrors the toggle state for this demonstration.

### Widget tree

```text
MaterialApp
└── LikeButtonPage
    └── Scaffold
        ├── AppBar
        └── Center
            └── Column
                ├── Semantics
                │   └── IconButton
                │       └── AnimatedBuilder
                │           └── Transform.scale
                │               └── Icon
                ├── SizedBox
                └── Text
```

### Execution flow

```text
Tap -> setState toggles liked state -> icon glyph/color and label update
  |
  +-> controller.forward(from: 0) -> scale sequence -> AnimatedBuilder
                                                         |
                                                         v
                                                   pulse renders
```

### ASCII phone preview

```text
+---------------------------+
|   Animated like button    |
+---------------------------+
|                           |
|            ♥              |
|          Liked            |
|                           |
+---------------------------+
```

### Example explanation

There are two kinds of state: `_isLiked` persists as the meaningful selection, while `_pulseController` represents temporary visual feedback and color interpolation. Separating them avoids treating an animation's current value as the source of truth for the user's action. Reversing the controller on unlike keeps the color transition and pulse connected to one timeline.

`IconButton` already includes semantics and interaction behavior. The explicit `Semantics` supplies a clear action label and selected flag. If it produces duplicate announcements in a particular screen-reader configuration, inspect the semantics tree and adjust the wrapper rather than removing accessible labeling without replacement.

If `setState` is removed, the liked icon and label do not update. If `forward(from: 0)` is removed, the state toggles without a pulse. If the controller is not disposed, a ticker leak may occur. If the tooltip/semantics are removed, the control becomes less understandable to non-visual users.

**Common mistakes:** Avoid a persistent oversized heart after liking; the pulse should return to its resting scale. Keep the hit target larger than the icon itself. Do not rely on pink alone to communicate selection.

**Professional use case:** Favorite, bookmark, and reaction controls can use the same toggle-plus-feedback separation.

## Complete Example 2

An implicit implementation can animate icon color when a boolean changes.

```dart
AnimatedContainer(
  duration: const Duration(milliseconds: 180),
  curve: Curves.easeOut,
  padding: const EdgeInsets.all(8),
  decoration: BoxDecoration(
    color: liked ? Colors.pink.shade50 : Colors.transparent,
    shape: BoxShape.circle,
  ),
  child: Icon(
    liked ? Icons.favorite : Icons.favorite_border,
    color: liked ? Colors.pink : Colors.blueGrey,
  ),
)
```

Place the container inside an `IconButton` and toggle `liked` in the parent. The icon glyph switches immediately; the surrounding supported decoration fades between targets.

**Widget tree:** `IconButton → AnimatedContainer → Icon`.  
**Flow:** boolean toggles → container receives new decoration → implicit transition runs.

This is enough when a pulse is not required. If the whole effect must sequence or replay on repeated taps, use an explicit pulse animation as in Example 1.

## Complete Example 3

A pressed/long-press reaction can use `GestureDetector` to start a second animation or display a reaction picker.

```dart
IconButton(
  onPressed: toggleLiked,
  onLongPress: showReactionPicker,
  icon: const Icon(Icons.favorite_border),
)
```

`onPressed` performs the primary toggle; `onLongPress` opens an expanded reaction choice. Flutter's Material control supplies familiar focus and tap behavior.

**Widget tree:** `IconButton → icon`.  
**Flow:** short activation toggles; long press invokes the picker callback.

If the long-press action is not exposed in visible instructions or semantics, users may not discover it. Keep a visible alternative when the picker contains important actions.

## Advanced Example

In a real social feed, the like action may update local state optimistically and synchronize with a server. Keep network state separate from the pulse animation. Handle rapid repeated taps and failed requests with a deliberate policy, and do not let an animation completion callback overwrite a newer user choice.

For a burst effect, animate decorative particles only if it supports the product's tone and does not obscure surrounding content. Respect reduced-motion settings and keep the action available without the flourish.

## Under the Hood

The controller sends tick notifications. The `TweenSequence` maps normalized time through growth and return stages. `AnimatedBuilder` listens and rebuilds the icon's transform. State changes rebuild the widget configuration for color, glyph, label, and selected semantics.

The transform affects the painted scale while the button's layout and tap target stay stable. This is useful: visual feedback can grow without shrinking the hit target.

## Performance Notes

- Keep the animation subtree limited to the icon.
- Avoid starting many particle effects for a long feed at once.
- Reuse static controls and profile dense scrolling lists.
- Keep animations short and stop repeating effects.
- Verify color interpolation and semantics independently from animation.

## Best Practices

- Keep semantic state separate from animation progress.
- Use `IconButton` or another accessible control as the interaction base.
- Add tooltip, label, and selected state.
- Keep the hit target generous and the pulse subtle.
- Ensure repeated taps have predictable behavior.
- Offer reduced-motion behavior when the app supports it.

## Common Mistakes

- Treating controller value as the durable liked state.
- Letting the pulse remain enlarged.
- Making an icon-only tiny hit target.
- Communicating state only through color.
- Omitting labels or selected semantics.
- Starting overlapping effects that obscure feedback.
- Coupling server response timing to animation completion.

## Debugging

- Confirm the semantic boolean and visible glyph change on each tap.
- Inspect controller status/value if the pulse does not run.
- Check the `IconButton` hit area and tooltip.
- Use the semantics debugger to find duplicate or missing labels.
- Test rapid taps and reduced-motion behavior.
- Verify an in-flight pulse does not overwrite a newer selected state.

## Interview Questions

1. **Why separate liked state from pulse progress?** The user selection persists; the pulse is temporary presentation.
2. **Why use `IconButton` instead of a bare icon gesture detector?** It provides button semantics, focus behavior, and a suitable interaction pattern.
3. **What does `TweenSequence` do here?** It maps one timeline into multiple pulse stages.
4. **How does the scale avoid changing the hit target?** The transform changes painted content while the button's layout bounds stay stable.
5. **What should an accessible toggle expose?** A meaningful label/action and its selected or toggled state.
6. **How should rapid repeated taps be handled?** Keep the semantic state authoritative and define predictable animation restart/reversal behavior.

## Try It Yourself

1. Add a numeric like count and animate its change.
2. Add a bookmark icon using a separate semantic state.
3. Make the pulse replay smoothly on repeated taps.
4. Test the control with keyboard focus and a screen reader.
5. Add a no-motion mode for the pulse.

## Challenge Project

Build a reusable reaction button that supports like/unlike, count changes, an accessible label, and a reaction picker. Simulate a failed update and ensure the semantic state and animation recover consistently.

## Quick Revision Sheet

- Keep persistent toggle state separate from temporary animation progress.
- Use a controller and sequence for a growth-and-return pulse.
- Use `IconButton` for built-in button behavior.
- Communicate selection through semantics and more than color.
- Keep hit targets stable and generous.
- Handle repeated taps and asynchronous updates deliberately.
