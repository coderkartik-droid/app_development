# Performance Optimization

## Introduction

Animation performance is the ability to update motion smoothly without making frames late. Flutter normally aims to prepare each frame within the display's frame budget. At 60 Hz, that budget is about 16.7 milliseconds; at higher refresh rates it is shorter.

Smoothness is not achieved by choosing a clever animation class alone. The widget work, layout, painting, image decoding, effects, and device workload all contribute. The goal is to animate only what needs to change and measure real performance rather than guess.

## Why this concept exists

An animation that looks smooth on a developer's fast device may stutter on an older phone or when the screen contains more content. Rebuilding large widget trees, repeatedly laying out expensive content, or painting large effects can consume the frame budget.

Optimization protects responsiveness, battery life, and accessibility. It is especially important for animations that run continuously or update large portions of the screen.

## Real-world analogy

Imagine a kitchen preparing one meal every display frame. Every extra task—chopping, reheating, washing dishes—uses time before the meal can be served. If the kitchen misses the service deadline, the customer sees a delay.

Flutter frames work similarly. Reduce unnecessary work, reuse prepared ingredients (static widgets), and profile to find the slow station rather than optimizing a random step.

## Internal Working

Flutter's frame pipeline includes widget build, render-object layout, paint recording, and raster/compositing work. A frame can be late because one of these stages takes too long. The UI thread and raster thread have separate responsibilities; a graph that shows one stage is not the whole story.

Animation controllers tick with vsync. Their listeners can trigger widget rebuilds or repaint notifications. A rebuild does not necessarily mean the whole app repaints, and a repaint does not necessarily require full layout. Choose APIs that keep work in the cheapest applicable stage:

- A `Transform` often changes paint without changing layout geometry.
- `AnimatedBuilder.child` avoids rebuilding static descendants.
- `CustomPainter(repaint: animation)` can repaint a drawing without rebuilding its widget ancestors.
- `RepaintBoundary` can isolate repaint work when profiling shows that isolation helps.

No optimization is automatically faster in every case. A boundary can add a composited layer and memory cost. Confirm the bottleneck with DevTools' Performance view and profile in profile mode on representative hardware.

## Syntax

```dart
AnimatedBuilder(
  animation: controller,
  child: const ExpensiveStaticWidget(),
  builder: (context, child) {
    return Transform.translate(
      offset: Offset(0, controller.value),
      child: child,
    );
  },
)
```

The builder runs when the animation notifies its listeners. The `child` parameter is constructed by the parent and reused across those updates.

A boundary is added around content like this:

```dart
RepaintBoundary(
  child: AnimatedWidget(),
)
```

Use it to isolate paint invalidation when measurements support it, not as a blanket wrapper around every widget.

## Complete Example 1

This runnable app animates a small indicator while reusing the static card with `AnimatedBuilder.child`. A `RepaintBoundary` separates the moving indicator from the surrounding card, providing a testable example of repaint isolation; profile it to determine whether the boundary is beneficial for your actual content.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const PerformanceApp());

class PerformanceApp extends StatelessWidget {
  const PerformanceApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      theme: ThemeData(useMaterial3: true, colorSchemeSeed: Colors.blue),
      home: const PerformancePage(),
    );
  }
}

class PerformancePage extends StatefulWidget {
  const PerformancePage({super.key});

  @override
  State<PerformancePage> createState() => _PerformancePageState();
}

class _PerformancePageState extends State<PerformancePage>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 900),
    )..repeat(reverse: true);
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Efficient animation')),
      body: Center(
        child: Card(
          child: SizedBox(
            width: 280,
            height: 180,
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                const Text('Static card content'),
                const SizedBox(height: 24),
                RepaintBoundary(
                  child: AnimatedBuilder(
                    animation: _controller,
                    child: const Icon(
                      Icons.circle,
                      color: Colors.blue,
                      size: 28,
                    ),
                    builder: (context, child) {
                      return Transform.translate(
                        offset: Offset(
                          80 * (_controller.value - 0.5),
                          0,
                        ),
                        child: child,
                      );
                    },
                  ),
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }
}
```

### Line-by-line explanation

- The Material import provides the application and UI widgets.
- `main` runs the root `PerformanceApp`; `MaterialApp` installs a theme and page.
- `PerformancePage` owns the controller and is therefore stateful.
- `SingleTickerProviderStateMixin` provides the `TickerProvider` required for vsync.
- `initState` creates one controller with a 900 ms cycle and repeats it in reverse.
- `dispose` releases the controller and ticker.
- `Scaffold`, `AppBar`, `Center`, `Card`, and `SizedBox` establish the static page structure and a fixed visual sample.
- `Column` lays out the title and moving indicator.
- `RepaintBoundary` creates a potential repaint isolation boundary around the animated region.
- `AnimatedBuilder` listens to the controller and rebuilds only its builder output.
- The static `Icon` is passed as `child`, so it is not reconstructed for every animation tick.
- `Transform.translate` moves the icon horizontally while the surrounding layout remains fixed.
- The offset maps controller progress from zero-to-one into an approximate -40-to-40 logical pixel range.

### Widget tree

```text
MaterialApp
└── PerformancePage
    └── Scaffold
        ├── AppBar
        └── Center
            └── Card
                └── SizedBox
                    └── Column
                        ├── Text
                        ├── SizedBox
                        └── RepaintBoundary
                            └── AnimatedBuilder
                                └── Transform.translate
                                    └── Icon
```

### Execution flow

```text
Ticker -> controller value -> AnimatedBuilder updates transform
                                    |
                                    v
                             isolated indicator paints

Static title/card content is not rebuilt by this listener.
```

### ASCII phone preview

```text
+---------------------------+
|   Efficient animation     |
+---------------------------+
|                           |
|   +-------------------+   |
|   | Static card       |   |
|   | content           |   |
|   |      o <->        |   |
|   +-------------------+   |
|                           |
+---------------------------+
```

### Example explanation

The main optimization demonstrated is scope: only the small animated subtree listens to the controller. Passing the icon as the builder's `child` prevents its widget configuration from being recreated on each tick. The transform changes its visual position without requiring the column to recalculate a different layout slot each frame.

The `RepaintBoundary` is included to show where isolation can be tested, not to guarantee a speedup. Remove it and compare raster and UI frame timings in DevTools. If it adds a layer without preventing meaningful repaint work, leaving it out may be better.

If `AnimatedBuilder` is replaced by a page-level `setState` call on every tick, the page's build method runs for each tick. If the controller's repeat is removed, the indicator remains static. If `dispose` is removed, an active ticker may be reported after leaving the page.

**Common mistakes:** Applying `RepaintBoundary` everywhere can increase memory and compositing cost. Assuming `Transform` makes a costly child cheap is also incorrect—the child still needs to be painted initially and may require compositing.

**Professional use case:** Keep app bars, large forms, and static imagery outside the listening subtree while animating a small status badge, progress affordance, or icon.

## Complete Example 2

A paint-only animation can use a repaint listenable so its painter updates directly.

```dart
class MeterPainter extends CustomPainter {
  MeterPainter(this.progress) : super(repaint: progress);

  final Animation<double> progress;

  @override
  void paint(Canvas canvas, Size size) {
    final paint = Paint()..color = Colors.green;
    canvas.drawRect(
      Rect.fromLTWH(0, 0, size.width * progress.value, 12),
      paint,
    );
  }

  @override
  bool shouldRepaint(covariant MeterPainter oldDelegate) =>
      oldDelegate.progress != progress;
}
```

Place it in a constrained `CustomPaint` and drive `progress` from a controller. `super(repaint: progress)` requests repaint when the animation changes; it avoids rebuilding the parent widget tree for the changing bar width.

**Widget tree:** `SizedBox → CustomPaint → MeterPainter`.  
**Execution flow:** ticker → progress animation → painter repaint → bar width recalculated and drawn.

If the animation is not supplied as a repaint listenable, a painter may remain visually stale unless some other rebuild or repaint occurs. If the size is unconstrained, the painter may not receive the expected drawing area.

## Complete Example 3

Avoid recalculating expensive static configuration on every frame by preparing it once and reusing it. For example, create a `TextPainter` only when its text, style, or layout constraints change, not inside a paint method that runs for every animation tick.

```dart
final staticChild = const ComplexDecoration();

AnimatedBuilder(
  animation: controller,
  child: staticChild,
  builder: (context, child) => Transform.scale(
    scale: 0.9 + controller.value * 0.1,
    child: child,
  ),
)
```

This fragment reuses the static decoration while changing only its scale. In a real widget, the `const` child is safe only when its inputs truly are constant.

**Widget tree:** `AnimatedBuilder → Transform.scale → static decoration`.  
**Flow:** controller tick → scale value changes → transform rebuilds with the same child object.

If the child depends on changing data, caching it as if static shows stale content. Optimization must preserve correctness; separate static and dynamic parts rather than freezing values that should update.

## Advanced Example

Use Flutter DevTools' Performance view to identify whether the UI thread or raster thread is missing frame deadlines. Record a representative interaction in profile mode, inspect slow frames, and correlate spikes with build, layout, paint, image decoding, or shader work.

Then make one targeted change and profile again under the same conditions. Examples include shrinking the listener scope, using a transform instead of animated layout when appropriate, precaching images, reducing unnecessary clips or shadows, and splitting repaint regions. Compare both average and worst-frame behavior; one good run is not proof that all devices will perform well.

## Under the Hood

`AnimationController` drives listener notifications from ticker callbacks. A `Listenable` consumer decides whether to schedule build or paint work. `AnimatedBuilder` is a widget-level listener; `CustomPainter` can listen at the paint stage. `RepaintBoundary` creates a separate layer boundary that may allow repaint reuse, but has compositing and memory implications.

Flutter DevTools presents frame timing so developers can distinguish expensive UI work from rasterization. A frame can be late even if the Dart callback seems small, for example when a complex scene takes too long to rasterize.

## Performance Notes

- Profile in profile mode, not only debug mode.
- Test on lower-end and high-refresh-rate devices.
- Minimize work inside high-frequency listeners and gesture callbacks.
- Reuse static subtrees through `AnimatedBuilder.child`.
- Use paint-only updates when layout does not need to change.
- Add repaint boundaries only after identifying useful isolation.
- Avoid large opacity layers, clips, shadows, and offscreen layers unless they serve the design.
- Stop repeated animations when hidden or no longer relevant.

## Best Practices

- Measure before optimizing and remeasure after each change.
- Optimize the actual bottleneck: build, layout, paint, or raster.
- Keep animated state localized.
- Preserve correctness and accessibility while simplifying work.
- Prefer built-in transitions where they express the design clearly.
- Check image and font loading during animation, not just steady state.
- Treat smoothness as a device and workload property, not an API property.

## Common Mistakes

- Assuming every rebuild repaints the entire screen.
- Assuming every repaint requires a rebuild.
- Wrapping every widget with `RepaintBoundary`.
- Using debug-mode timing as a release performance prediction.
- Optimizing code that is not on the slow-frame path.
- Rebuilding complex static descendants on each tick.
- Changing layout geometry when a visual transform would suffice.
- Keeping off-screen infinite animations active.

## Debugging

- Record the animation in DevTools Performance view and locate missed frame deadlines.
- Check the UI and raster graphs independently.
- Enable repaint visualization to find unexpectedly broad invalidation.
- Temporarily remove effects such as blur, clipping, and shadows to isolate cost.
- Compare with and without a repaint boundary.
- Verify that image loading or shader compilation is not causing first-run jank.
- Reproduce on a physical device and the same build mode used for performance decisions.

## Interview Questions

1. **What is the frame budget at 60 Hz?** Approximately 16.7 ms for each frame.
2. **Does a widget rebuild always cause a repaint?** No; build, layout, and paint are related but distinct phases.
3. **What does `AnimatedBuilder.child` optimize?** It reuses a static child rather than rebuilding it for every animation notification.
4. **What is a `RepaintBoundary`?** A compositing boundary that can isolate paint invalidation, with possible layer and memory costs.
5. **When can `CustomPainter(repaint: animation)` help?** When the changing output is drawing and parent widget rebuilds are unnecessary.
6. **Why profile in profile mode?** Debug instrumentation and assertions distort performance compared with optimized execution.
7. **What are UI and raster work?** UI work prepares framework/render commands; raster work turns the scene into pixels for display.

## Try It Yourself

1. Remove `AnimatedBuilder.child` and compare build counts.
2. Profile the sample with and without `RepaintBoundary`.
3. Animate width instead of translation and inspect layout timings.
4. Add a large shadow and compare raster cost.
5. Stop the controller when the page is not visible.

## Challenge Project

Take an existing animation-heavy screen and use DevTools to identify one reproducible slow-frame cause. Make one focused improvement, document the before/after measurement in your own notes, and confirm that the visual and accessible behavior remains correct.

## Quick Revision Sheet

- Smooth frames require build, layout, paint, and raster work to fit their budgets.
- Profile before optimizing.
- Keep listeners small and static children reusable.
- Use transforms for visual-only movement where appropriate.
- Use `CustomPainter(repaint:)` for drawing-only updates.
- A repaint boundary can help or hurt; measure it.
- Test representative devices and build modes.
- Stop unneeded animations and preserve correctness.
