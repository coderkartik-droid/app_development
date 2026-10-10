# Animations Cheat Sheet

## Most Important Concepts

- Implicit animations animate property changes automatically.
- Explicit animations use an `AnimationController` and `Animation<T>`.
- A ticker produces frame updates; `vsync` coordinates with screen refresh.
- Dispose controllers and use curves to shape motion.

## Quick Comparison

| Need | Tool |
|---|---|
| Animate one property change | `AnimatedContainer`, `AnimatedOpacity` |
| Control timing/repeat/reverse | `AnimationController` |
| Transform an animation | `Tween`, `CurvedAnimation` |
| Rebuild only animated child | `AnimatedBuilder` |
| Route transition | `Hero` |

## Syntax Reference

```dart
AnimatedContainer(
  duration: const Duration(milliseconds: 300),
  curve: Curves.easeInOut,
  width: expanded ? 220 : 120,
  color: expanded ? Colors.teal : Colors.grey,
)
```

```dart
late final AnimationController controller = AnimationController(
  vsync: this,
  duration: const Duration(milliseconds: 500),
);
```

## Common Classes / Methods

- `AnimationController`: `forward`, `reverse`, `repeat`, `stop`, `dispose`
- `Tween<T>.animate(parent)`: map controller range to a value
- `CurvedAnimation`: transform timing curve
- `FadeTransition`, `ScaleTransition`, `SlideTransition`, `RotationTransition`
- `AnimatedBuilder`, `TickerProviderStateMixin`

## Parameters & Properties

- `duration`, `reverseDuration`, `curve`
- `begin`, `end`, `lowerBound`, `upperBound`
- `vsync`: ticker provider
- `alignment`, `opacity`, `position`: transition-specific animated values

## Short Example

```dart
late final Animation<double> fade =
    CurvedAnimation(parent: controller, curve: Curves.easeIn);
FadeTransition(opacity: fade, child: const Text('Hello'));
```

## Common Mistakes

- Forgetting controller disposal.
- Creating a new controller in `build`.
- Animating a large subtree unnecessarily.
- Using zero duration where a visible transition is expected.
- Assuming opacity zero prevents hit testing; use `IgnorePointer` if needed.

## Interview Revision

- Implicit = value changes trigger built-in animation; explicit = app controls timeline.
- `Ticker` calls frame callbacks; `vsync` avoids unnecessary offscreen ticks.
- A tween maps normalized progress to a property value.
- Animation value changes do not automatically rebuild arbitrary widgets; use an animation builder/listener.

## Best Practices

- Prefer implicit animations for simple transitions.
- Respect reduced-motion/accessibility needs.
- Keep animated rebuild scopes small.
- Avoid overlapping controllers for the same visual property.

## Memory Tricks

- Controller = clock; Tween = range; Curve = feel; Transition = apply.
- `0.0 → 1.0` is progress, not necessarily the final property value.

## One-Page Summary

Use implicit widgets for straightforward changes and explicit controllers for choreography, gestures, or repeat/reverse control. Tie controllers to `vsync`, dispose them, transform progress with tweens/curves, and rebuild only the animated region.
