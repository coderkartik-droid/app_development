# Animation Practice Exercises

## Introduction

Practice turns animation concepts into engineering intuition. Start with a visible goal, decide which values change, choose the simplest suitable Flutter animation tool, and test the result under interruption and different device sizes.

The exercises in this chapter move from basic property changes to gesture and lifecycle challenges. Try each prompt before reading the guidance.

## Why this concept exists

Reading API explanations does not reveal every design decision. Developers need practice selecting implicit versus explicit control, mapping values, coordinating gestures, and spotting expensive rendering work.

Small exercises provide fast feedback before those patterns are used in a larger feature.

## Real-world analogy

Learning animation resembles practicing a musical scale. The basic motion is simple, but repetition teaches timing, control, and how to recover when a note is missed. Each exercise isolates one skill before combining them.

## Internal Working

Every exercise can be analyzed using the same pipeline:

1. A user action or lifecycle event changes an animation target or controller.
2. A controller/ticker or implicit animation schedules updates.
3. A curve and tween map elapsed progress into the desired property.
4. A transition, builder, or painter consumes the value.
5. Flutter performs the required build, layout, paint, and compositing work.

When debugging, identify which step is absent rather than randomly changing durations.

## Syntax

The smallest implicit animation pattern is:

```dart
AnimatedContainer(
  duration: const Duration(milliseconds: 250),
  width: expanded ? 220 : 120,
  color: selected ? Colors.green : Colors.grey,
)
```

The smallest explicit pattern creates a controller in `State`, connects a transition to it, starts it from an event, and disposes it with the state.

## Complete Example 1

This runnable practice app lets the learner tap a button to animate a square's color, size, and corner radius. It uses `AnimatedContainer`, so the exercise can focus on target-state changes rather than controller setup.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const PracticeApp());

class PracticeApp extends StatelessWidget {
  const PracticeApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      theme: ThemeData(useMaterial3: true, colorSchemeSeed: Colors.cyan),
      home: const ShapePracticePage(),
    );
  }
}

class ShapePracticePage extends StatefulWidget {
  const ShapePracticePage({super.key});

  @override
  State<ShapePracticePage> createState() => _ShapePracticePageState();
}

class _ShapePracticePageState extends State<ShapePracticePage> {
  bool _expanded = false;

  @override
  Widget build(BuildContext context) {
    final size = _expanded ? 210.0 : 130.0;

    return Scaffold(
      appBar: AppBar(title: const Text('Shape practice')),
      body: Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            AnimatedContainer(
              duration: const Duration(milliseconds: 450),
              curve: Curves.easeInOutCubic,
              width: size,
              height: size,
              alignment: Alignment.center,
              decoration: BoxDecoration(
                color: _expanded ? Colors.cyan : Colors.deepPurple,
                borderRadius: BorderRadius.circular(_expanded ? 48 : 12),
              ),
              child: Text(
                _expanded ? 'Expanded' : 'Tap below',
                style: const TextStyle(color: Colors.white, fontSize: 18),
              ),
            ),
            const SizedBox(height: 24),
            FilledButton(
              onPressed: () => setState(() => _expanded = !_expanded),
              child: const Text('Toggle shape'),
            ),
          ],
        ),
      ),
    );
  }
}
```

### Line-by-line explanation

- `main` runs the app and its shape-practice page.
- `MaterialApp` supplies theme and app-level structure.
- `ShapePracticePage` is stateful because `_expanded` records the target state.
- `_expanded` determines the target size, color, corner radius, and label.
- `build` calculates `size` from state. The `210.0` and `130.0` values are doubles suitable for dimensions.
- `Scaffold`, `AppBar`, `Center`, and `Column` position the sample and its control.
- `AnimatedContainer.duration` sets the time over which supported property changes animate.
- `curve` controls the pacing of interpolation.
- `width` and `height` use the same target, keeping the shape square.
- `alignment` centers the label inside the box.
- `BoxDecoration` supplies the animated color and border radius.
- The label changes with the target state; text itself is not automatically cross-faded.
- `FilledButton` toggles the state with `setState`, causing a rebuild with new targets.

### Widget tree

```text
MaterialApp
└── ShapePracticePage
    └── Scaffold
        ├── AppBar
        └── Center
            └── Column
                ├── AnimatedContainer
                │   └── Text
                ├── SizedBox
                └── FilledButton
```

### Execution flow

```text
Button tap -> toggle _expanded -> build with new targets
                                      |
                                      v
                       AnimatedContainer compares old/new
                                      |
                                      v
                     interpolates size, color, and radius
                                      |
                                      v
                                updated frames
```

### ASCII phone preview

```text
+---------------------------+
|      Shape practice       |
+---------------------------+
|                           |
|      +----------+         |
|      | Tap below|         |
|      +----------+         |
|                           |
|      [Toggle shape]       |
+---------------------------+
```

### Example explanation

The parent does not manually calculate intermediate values. It changes the target by toggling `_expanded`; `AnimatedContainer` compares its previous and new supported properties and animates between them.

If the `duration` is set to zero, the shape changes immediately. If the button does not call `setState`, the new target is not rebuilt. If `AnimatedContainer` is replaced by `Container`, the values still change but there is no implicit interpolation. The changing label may switch abruptly because a container does not animate text content.

**Common mistakes:** State should represent the target, not an arbitrary intermediate value. Keep dimensions within available layout constraints. Do not expect unsupported properties or child content to animate automatically.

**Professional use case:** A compact selection tile can subtly expand and change color to communicate selection without an explicit controller.

## Complete Example 2

**Exercise:** Fade a message in when a switch is enabled.  
**Hint:** Use `AnimatedOpacity` with a boolean target and a nonzero duration.

```dart
AnimatedOpacity(
  opacity: visible ? 1 : 0,
  duration: const Duration(milliseconds: 250),
  child: const Text('Saved successfully'),
)
```

The parent must rebuild when `visible` changes. `AnimatedOpacity` interpolates its opacity. At zero opacity the child may still occupy layout space and may still need hit-testing or semantics handling if it is interactive.

**Widget tree:** `StatefulWidget → AnimatedOpacity → Text`.  
**Flow:** toggle boolean → rebuild target opacity → fade transition → text becomes visible.

If the duration is omitted or zero, the change is immediate. If the child can receive input while invisible, wrap it in an appropriate pointer/semantics policy.

## Complete Example 3

**Exercise:** Animate a panel from closed to open while coordinating its arrow rotation. Use one explicit controller with two tweens, then start it forward or reverse from a button.

```dart
final height = Tween<double>(begin: 0, end: 160).animate(controller);
final turns = Tween<double>(begin: 0, end: 0.5).animate(controller);
```

Use `SizeTransition(sizeFactor: height, child: panel)` and `RotationTransition(turns: turns, child: icon)`. Both animations share one timeline, so they remain synchronized.

**Widget tree:** button and `Column` containing `RotationTransition` plus `SizeTransition`.  
**Flow:** tap -> controller direction changes -> both tweens update -> icon rotates and panel reveals.

If the controller is not driven, neither animation changes. If separate controllers are used without synchronization, their timing can drift or require extra lifecycle code.

## Advanced Example

For each exercise, write tests for the target and the temporal behavior. Widget tests can use `tester.pump` and `pumpAndSettle` for finite animations; avoid waiting for an infinite repeating animation to settle. Test interruption by changing the target before the first transition completes.

An exercise can be extended with responsiveness by using `LayoutBuilder` constraints, accessibility by adding semantic labels and alternative controls, and reduced motion by choosing a shorter or immediate transition according to the app's policy.

## Under the Hood

Implicit widgets keep internal animation state and retarget supported properties when their configuration changes. Explicit animation uses a controller created by the developer. In both cases, frame updates eventually affect render objects and visible output through Flutter's pipeline.

Widget tests do not display frames on a physical screen, but they can advance the test clock to verify expected intermediate and final states.

## Performance Notes

- Prefer a small example that isolates one animation concept.
- Do not use infinite controllers in tests unless the test explicitly controls and stops them.
- Avoid rebuilding unrelated UI on every tick.
- Use constrained dimensions and test small screens.
- Exercise overlapping or interrupted state changes.

## Best Practices

- Make each exercise observable and measurable.
- State the intended final value before choosing an API.
- Start with implicit animations for target changes.
- Use explicit controllers for sequencing or shared progress.
- Add gesture and physics only when the interaction needs them.
- Test edge cases, interruption, and cleanup.
- Keep sample code runnable and minimal while explaining the production considerations.

## Common Mistakes

- Animating several properties with unrelated controllers when one timeline suffices.
- Calling `pumpAndSettle` on a repeating animation that never settles.
- Using opacity as if it removed a widget from layout.
- Forgetting the parent must rebuild with new implicit targets.
- Ignoring responsive constraints.
- Testing only the final frame, not the intermediate state.

## Debugging

- Confirm state changed and the parent rebuilt.
- Check that the animation duration is nonzero.
- Use intermediate `pump` durations to inspect in-flight output.
- For explicit animation, inspect controller status and value.
- Ensure the test stops repeat animations.
- Check layout constraints if the animation overflows or appears clipped.

## Interview Questions

1. **What should you decide before selecting an animation API?** Which values change, how they are controlled, and whether sequencing or gestures are required.
2. **Why are implicit animations useful in a simple toggle?** They manage interpolation between rebuilt target values.
3. **When is one controller with multiple tweens helpful?** When several properties must share a synchronized timeline.
4. **Why can `pumpAndSettle` hang?** A repeating or otherwise never-ending animation prevents the tree from settling.
5. **Does opacity zero remove layout space?** No; the widget can still occupy its layout slot.
6. **How can you test animation progress?** Advance the widget test clock using timed `pump` calls and inspect intermediate states.
7. **What should animation tests cover besides the final frame?** Target changes, interruption, lifecycle, bounds, and relevant accessibility behavior.

## Try It Yourself

1. Change the shape exercise to use an `AnimatedAlign`.
2. Make the label cross-fade instead of switching abruptly.
3. Add a reverse action while the animation is in progress.
4. Write a widget test for the initial and final size.
5. Test on a narrow screen and prevent overflow.

## Challenge Project

Create a small animation playground with three cards: an implicit property animation, a controller-driven sequence, and a gesture-driven spring. Each card must have a reset action, a short explanation of the API used, and a layout that remains usable on a narrow phone.

## Quick Revision Sheet

- Change a target for implicit animations.
- Use controllers for explicit timing and coordination.
- Tweens map progress; curves shape timing.
- Test intermediate frames as well as final states.
- Avoid waiting for infinite animations to settle.
- Opacity affects visibility, not necessarily layout or interaction.
- Use one controller for synchronized properties where appropriate.
- Always test interruption, boundaries, and small screens.
