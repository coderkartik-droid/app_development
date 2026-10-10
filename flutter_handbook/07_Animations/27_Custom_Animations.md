# Custom Animations

## Introduction

A custom animation is motion whose visual result or timing is designed specifically for your interface. Flutter's built-in implicit widgets cover common changes such as color, size, and alignment. When an effect needs several coordinated values, an unusual shape, or a custom drawing sequence, you can combine an `AnimationController`, one or more `Tween`s, and a builder or `CustomPainter`.

The animation value is usually just a number that changes over time. Your code maps that number to properties such as angle, color, position, or path geometry.

## Why this concept exists

Prebuilt animation widgets cannot represent every design. A loading indicator may draw a custom arc; a chart may animate each bar; a decorative shape may morph as a screen opens.

Custom animation gives control over the timeline and output while still using Flutter's frame scheduling, ticker lifecycle, and rendering pipeline.

## Real-world analogy

An implicit animation is like telling a stage crew, “Move this chair over there in three seconds.” A custom animation is like directing the complete scene: when the light changes, how the chair travels, whether it pauses, and what the audience sees at each moment.

The controller is the scene clock. Tweens translate time progress into property values. Builders and painters turn those values into pixels.

## Internal Working

`AnimationController` produces a changing value over a timeline. `Tween<T>` maps controller progress into a domain range, and a `Curve` can reshape the timing. A `CurvedAnimation` or `Curves` argument to `drive` can make motion ease in, bounce, or follow another timing profile.

A listener such as `AnimatedBuilder` rebuilds a small widget subtree when the animation changes. A `CustomPainter` can instead receive the animation as a repaint `Listenable`; this schedules paint without requiring the surrounding widget subtree to rebuild on every tick.

Frames still pass through Flutter's pipeline. Painting a custom path is often appropriate for decorative effects, but complex paths, shaders, clips, and large layers can be expensive. Use vsync, dispose the controller, and separate static UI from animated output.

## Syntax

```dart
late final AnimationController controller;
late final Animation<double> progress;

@override
void initState() {
  super.initState();
  controller = AnimationController(
    vsync: this,
    duration: const Duration(milliseconds: 900),
  )..repeat();
  progress = CurvedAnimation(
    parent: controller,
    curve: Curves.easeInOut,
  );
}
```

Use `progress.value` to calculate output values. Stop infinite animations when no longer needed and dispose their controller. For a finite animation, call `forward`, `reverse`, or another controller method from an event handler rather than restarting it from `build`.

## Complete Example 1

This complete app draws a breathing, pulsing circle using `CustomPainter`. The controller's repeating value drives the circle's radius and color. The static app bar and text do not rebuild on every animation frame.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const CustomAnimationApp());

class CustomAnimationApp extends StatelessWidget {
  const CustomAnimationApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      theme: ThemeData(useMaterial3: true, colorSchemeSeed: Colors.deepPurple),
      home: const PulsePage(),
    );
  }
}

class PulsePage extends StatefulWidget {
  const PulsePage({super.key});

  @override
  State<PulsePage> createState() => _PulsePageState();
}

class _PulsePageState extends State<PulsePage>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;
  late final Animation<double> _pulse;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 1200),
    )..repeat(reverse: true);
    _pulse = CurvedAnimation(parent: _controller, curve: Curves.easeInOut);
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Custom Pulse')),
      body: Center(
        child: SizedBox(
          width: 280,
          height: 280,
          child: CustomPaint(
            painter: PulsePainter(animation: _pulse),
            child: const Center(
              child: Text(
                'Breathe',
                style: TextStyle(color: Colors.white, fontSize: 22),
              ),
            ),
          ),
        ),
      ),
    );
  }
}

class PulsePainter extends CustomPainter {
  PulsePainter({required this.animation}) : super(repaint: animation);

  final Animation<double> animation;

  @override
  void paint(Canvas canvas, Size size) {
    final center = Offset(size.width / 2, size.height / 2);
    final radius = 70 + 18 * animation.value;
    final color = Color.lerp(
      Colors.deepPurple,
      Colors.pinkAccent,
      animation.value,
    )!;
    final paint = Paint()..color = color;
    canvas.drawCircle(center, radius, paint);
  }

  @override
  bool shouldRepaint(covariant PulsePainter oldDelegate) {
    return oldDelegate.animation != animation;
  }
}
```

### Line-by-line explanation

- `package:flutter/material.dart` provides the widget, animation, and Canvas APIs used by this app.
- `main` starts `CustomAnimationApp`; `MaterialApp` sets theme and the pulse page.
- `PulsePage` owns the animation, so it is a `StatefulWidget`.
- `SingleTickerProviderStateMixin` supplies vsync for the page's controller.
- `_controller` progresses from zero to one; `_pulse` applies an ease-in/ease-out timing profile.
- In `initState`, the controller is created once and repeats in reverse, producing an ongoing expansion and contraction.
- `dispose` stops the ticker and releases controller resources.
- The page's `Scaffold` contains an app bar and a fixed `SizedBox` that gives the painter a known drawing area.
- `CustomPaint` delegates the changing circle to `PulsePainter`. Its child label is centered over the painted background.
- `PulsePainter` passes the animation as `repaint`. Flutter can call `paint` as the listenable changes without rebuilding the entire page.
- `paint` calculates the center and radius, interpolates a color, configures `Paint`, and draws the circle.
- `shouldRepaint` returns true if the painter receives a different animation object. Because this painter listens to its animation with `repaint`, value changes already trigger painting.

### Widget tree

```text
MaterialApp
└── PulsePage
    └── Scaffold
        ├── AppBar
        └── Center
            └── SizedBox
                └── CustomPaint
                    ├── PulsePainter (circle)
                    └── Center
                        └── Text
```

### Execution flow

```text
Ticker -> controller value -> CurvedAnimation
                                |
                                v
                         CustomPainter repaint
                                |
                                v
                 calculate radius and interpolated color
                                |
                                v
                         Canvas draws circle
```

### ASCII phone preview

```text
+---------------------------+
|       Custom Pulse        |
+---------------------------+
|                           |
|          .---.            |
|       .-'     '-.         |
|      (  Breathe  )        |
|       '-.     .-'         |
|          '---'            |
|                           |
+---------------------------+
```

The circle continuously grows and shrinks while its color blends between purple and pink.

### Example explanation

`CustomPainter` is suitable because the changing output is a simple drawing. Passing the animation to `super(repaint: animation)` connects its notifications directly to painting. The fixed `SizedBox` ensures the painter has a useful size; without constraints, `CustomPaint` might not receive the drawing area expected.

If the controller's `repeat` call is removed, the painter draws only the initial value and appears static. If the controller is not disposed, the ticker can outlive the page. If the `repaint` listenable is removed, the painter will not automatically repaint as the animation changes.

**Common mistakes:** Performing state updates from `paint` is incorrect; painting should be deterministic for its inputs. Forgetting to constrain `CustomPaint` can produce zero or unexpected size. A painter that changes its visual output based on external data should implement `shouldRepaint` correctly.

**Professional use case:** Custom-painted chart transitions, progress rings, audio visualizers, and branded loading indicators can avoid building large widget trees for simple graphics.

## Complete Example 2

`TweenSequence` can divide one controller timeline into stages: grow, hold, then shrink.

```dart
final size = TweenSequence<double>([
  TweenSequenceItem(
    tween: Tween(begin: 0.8, end: 1.15)
        .chain(CurveTween(curve: Curves.easeOut)),
    weight: 45,
  ),
  TweenSequenceItem(
    tween: ConstantTween<double>(1.15),
    weight: 10,
  ),
  TweenSequenceItem(
    tween: Tween(begin: 1.15, end: 1.0)
        .chain(CurveTween(curve: Curves.easeIn)),
    weight: 45,
  ),
]).animate(controller);
```

The weights allocate portions of the controller's normalized timeline. The first tween grows the element, the constant tween holds it briefly, and the final tween returns it to normal size. Use `ScaleTransition(scale: size, child: ...)` or `Transform.scale(scale: size.value, child: ...)` to display the result.

**Widget tree:** `AnimatedBuilder → ScaleTransition → target widget`.  
**Execution flow:** controller progress → sequence selects stage → curve maps stage progress → scale transition paints the result.

If the sequence weights do not total 100, the values are still treated proportionally, but the intended timing can be harder to reason about. If the controller never runs, the sequence stays at its initial value.

## Complete Example 3

Custom color changes can be driven by an animation and used to paint a container.

```dart
final color = ColorTween(
  begin: Colors.blue,
  end: Colors.orange,
).animate(controller);

AnimatedBuilder(
  animation: color,
  builder: (context, child) => Container(
    width: 120,
    height: 120,
    color: color.value,
    child: child,
  ),
  child: const Icon(Icons.star, color: Colors.white),
)
```

`ColorTween` interpolates between two colors as the controller changes. `AnimatedBuilder` rebuilds the container's color while reusing the star icon as its static child.

**Widget tree:** `AnimatedBuilder → Container → Icon`.  
**Flow:** controller changes → color tween calculates an intermediate color → builder updates the container.

If the tween is removed, the container receives no changing color. If the builder's `child` is removed, the icon is reconstructed on each update, even though it is static.

## Advanced Example

For a custom chart animation, compute each bar's height from a shared controller value and its own target data. Keep data normalization outside `paint` if it is expensive. The painter should receive immutable inputs and use the animation value only to interpolate between old and new values.

Use `Canvas.save` and `Canvas.restore` when applying temporary transforms or clips. Consider clipping, device pixel ratio, semantics, and hit testing: custom-painted pixels do not automatically become accessible controls. If users must interact with individual painted marks, provide a gesture layer and semantic labels.

## Under the Hood

An `Animation<T>` is a listenable value over time. The controller produces a numeric timeline; a tween maps that timeline to another type; a curve changes how progress is distributed over time. `AnimatedBuilder` registers a listener and schedules a build when notified.

`CustomPainter(repaint: listenable)` uses a repaint listenable to mark the render object for painting. This can avoid rebuilding ancestors for every animation tick, though painting itself still costs time. Canvas operations are recorded by Flutter's rendering system and later composited for display.

## Performance Notes

- Reuse static widgets and painter objects where practical.
- Keep paint methods free of I/O, expensive allocations, and application state changes.
- Use `repaint` listenables for paint-only animation.
- Avoid unnecessary `saveLayer`; it can allocate offscreen buffers.
- Bound custom drawing with a sensible size and clip only when required.
- Profile complex paths and effects on real devices.
- Stop infinite controllers when the animation is no longer visible or relevant.

## Best Practices

- Begin with an implicit animation or transition widget; use custom machinery only when it adds needed control.
- Separate timeline, interpolation, and drawing responsibilities.
- Use named constants for timing and geometry.
- Implement `shouldRepaint` based on painter input changes.
- Provide accessibility semantics for information drawn only on a canvas.
- Test at different sizes and text scales.
- Dispose all controllers and remove listeners.

## Common Mistakes

- Starting or repeating a controller inside `build`.
- Rebuilding an entire page for a tiny animated property.
- Updating application state inside `paint`.
- Forgetting the `repaint` listenable and expecting a painter to update itself.
- Incorrectly implementing `shouldRepaint`.
- Ignoring canvas size, clipping, or device-independent logical coordinates.
- Assuming painted shapes are automatically accessible or hit-testable.

## Debugging

- Confirm the controller is active and the animation value changes.
- Log or inspect tween output before blaming Canvas drawing.
- Check `CustomPaint` constraints and painter `Size`.
- If painter changes are not reflected, inspect `repaint` and `shouldRepaint`.
- If a shape is clipped, inspect the canvas bounds and ancestor clipping.
- Use performance tooling to distinguish build cost from paint and raster cost.

## Interview Questions

1. **What does a `Tween` do?** It maps an animation's input range into a typed output range.
2. **What is the role of a curve?** It transforms the progress timing without changing the underlying timeline.
3. **Why use `CustomPainter(repaint: animation)`?** Animation notifications can schedule paint directly rather than rebuilding a larger widget subtree.
4. **What should `shouldRepaint` determine?** Whether the new painter's inputs differ in a way that changes the drawing.
5. **What is a `TweenSequence`?** A set of weighted tween stages driven by one animation timeline.
6. **Why is `paint` expected to be deterministic?** Flutter may invoke it whenever rendering needs updating; it should draw from supplied state, not mutate app state.
7. **Do Canvas-drawn controls automatically have semantics?** No; accessibility semantics and interaction need to be supplied.

## Try It Yourself

1. Change the pulse curve to `Curves.elasticInOut`.
2. Add a glowing outer circle with lower opacity.
3. Animate a rounded rectangle's width and corner radius.
4. Add a pause/resume button and stop the controller when paused.
5. Convert the pulse to a painter-driven progress ring.

## Challenge Project

Create a custom-painted progress indicator with an animated arc, percentage label, and pause/resume action. Support arbitrary progress values, expose the progress in semantic labels, and keep the text readable at large accessibility text scales.

## Quick Revision Sheet

- Controller = timeline; tween = value mapping; curve = timing shape.
- `AnimatedBuilder` is convenient for widget animation.
- `CustomPainter(repaint: animation)` is useful for paint-only updates.
- `paint` draws; it should not mutate application state.
- `shouldRepaint` compares drawing inputs.
- Constrain custom paint, provide semantics, and profile complex drawing.
- Dispose controllers and stop infinite animations when unnecessary.
