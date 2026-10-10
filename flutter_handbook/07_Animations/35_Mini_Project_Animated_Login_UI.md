# Mini Project: Animated Login UI

## Introduction

This project creates a small login screen with an animated header and a form that fades and moves into view. The animation is decorative: it does not replace validation, form submission, or accessible controls.

We use one controller to coordinate opacity and vertical position, then keep the login fields inside a scrollable layout so smaller screens and the keyboard remain usable.

## Why this concept exists

A login screen is often the first interaction with an app. A short entrance transition can establish visual hierarchy and make the screen feel responsive. Motion should be brief and should never prevent users from entering credentials or submitting the form.

## Real-world analogy

When a presenter begins, the title appears first and the main content follows. An animated login screen uses a similar visual order: welcome heading first, then fields and action.

## Internal Working

The page creates an `AnimationController` in `initState`, then derives curved opacity and slide animations. A one-frame callback starts the controller after the first layout so the initial state can be rendered before the entrance transition begins.

The controller ticks with vsync. `FadeTransition` and `SlideTransition` listen directly to derived animations; they update their render output without manually calling `setState` per frame. The form uses `SingleChildScrollView` to adapt when the keyboard reduces available height. The controller is disposed with the page.

## Syntax

```dart
FadeTransition(
  opacity: opacityAnimation,
  child: SlideTransition(
    position: slideAnimation,
    child: child,
  ),
)
```

`SlideTransition.position` is an `Animation<Offset>` where offsets are fractions of the child's size, not logical pixels. Positive vertical offset begins below its normal position; the tween end can be zero to settle at the layout position.

## Complete Example 1

This runnable screen includes name and password fields, basic validation, and an entrance animation. It deliberately does not perform real authentication.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const AnimatedLoginApp());

class AnimatedLoginApp extends StatelessWidget {
  const AnimatedLoginApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      theme: ThemeData(useMaterial3: true, colorSchemeSeed: Colors.indigo),
      home: const AnimatedLoginPage(),
    );
  }
}

class AnimatedLoginPage extends StatefulWidget {
  const AnimatedLoginPage({super.key});

  @override
  State<AnimatedLoginPage> createState() => _AnimatedLoginPageState();
}

class _AnimatedLoginPageState extends State<AnimatedLoginPage>
    with SingleTickerProviderStateMixin {
  final _formKey = GlobalKey<FormState>();
  late final AnimationController _controller;
  late final Animation<double> _opacity;
  late final Animation<Offset> _slide;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 700),
    );
    final entrance = CurvedAnimation(
      parent: _controller,
      curve: Curves.easeOutCubic,
    );
    _opacity = entrance;
    _slide = Tween<Offset>(
      begin: const Offset(0, 0.12),
      end: Offset.zero,
    ).animate(entrance);

    WidgetsBinding.instance.addPostFrameCallback((_) {
      if (mounted) _controller.forward();
    });
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  void _submit() {
    if (_formKey.currentState!.validate()) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text('Demo form is valid')),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Welcome back')),
      body: SafeArea(
        child: SingleChildScrollView(
          padding: const EdgeInsets.all(24),
          child: Center(
            child: ConstrainedBox(
              constraints: const BoxConstraints(maxWidth: 420),
              child: FadeTransition(
                opacity: _opacity,
                child: SlideTransition(
                  position: _slide,
                  child: Form(
                    key: _formKey,
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.stretch,
                      children: [
                        const SizedBox(height: 24),
                        const Icon(
                          Icons.lock_outline,
                          size: 64,
                          color: Colors.indigo,
                        ),
                        const SizedBox(height: 20),
                        Text(
                          'Sign in',
                          textAlign: TextAlign.center,
                          style: Theme.of(context).textTheme.headlineMedium,
                        ),
                        const SizedBox(height: 28),
                        TextFormField(
                          decoration: const InputDecoration(
                            labelText: 'Email',
                            prefixIcon: Icon(Icons.email_outlined),
                            border: OutlineInputBorder(),
                          ),
                          keyboardType: TextInputType.emailAddress,
                          validator: (value) {
                            if (value == null || !value.contains('@')) {
                              return 'Enter a valid email address';
                            }
                            return null;
                          },
                        ),
                        const SizedBox(height: 16),
                        TextFormField(
                          obscureText: true,
                          decoration: const InputDecoration(
                            labelText: 'Password',
                            prefixIcon: Icon(Icons.key),
                            border: OutlineInputBorder(),
                          ),
                          validator: (value) {
                            if (value == null || value.length < 8) {
                              return 'Use at least 8 characters';
                            }
                            return null;
                          },
                        ),
                        const SizedBox(height: 24),
                        FilledButton(
                          onPressed: _submit,
                          child: const Text('Continue'),
                        ),
                      ],
                    ),
                  ),
                ),
              ),
            ),
          ),
        ),
      ),
    );
  }
}
```

### Line-by-line explanation

- `main` starts the app. `MaterialApp` configures the theme and login page.
- `_formKey` identifies the `Form` state for validation.
- `_controller` owns the entrance timeline; `_opacity` and `_slide` are its visual outputs.
- `initState` creates a 700 ms controller and a curved animation shared by opacity and slide.
- The slide tween starts at a small fractional vertical offset and ends at the widget's normal position.
- `addPostFrameCallback` starts the animation after the first frame. The `mounted` guard ensures the page still exists.
- `dispose` releases the controller.
- `_submit` validates the form and shows a demo confirmation only when input passes validation.
- `Scaffold` and `SafeArea` provide the page structure and avoid system UI intrusions.
- `SingleChildScrollView` lets the form remain reachable when the keyboard reduces the viewport.
- `ConstrainedBox` caps the form width on larger displays while allowing it to shrink on phones.
- `FadeTransition` and `SlideTransition` consume animation values without per-frame `setState`.
- `Form` groups fields under the form key.
- `Column(crossAxisAlignment: stretch)` makes the fields and button fill the available form width.
- `TextFormField` provides labeled, keyboard-aware input. Validators return user-facing errors.
- `obscureText` hides password characters on screen.
- `FilledButton` submits the form and remains a real control independent of animation.

### Widget tree

```text
MaterialApp
└── AnimatedLoginPage
    └── Scaffold
        └── SafeArea
            └── SingleChildScrollView
                └── Center
                    └── ConstrainedBox
                        └── FadeTransition
                            └── SlideTransition
                                └── Form
                                    └── Column
                                        ├── Icon
                                        ├── Text
                                        ├── TextFormField (email)
                                        ├── TextFormField (password)
                                        └── FilledButton
```

### Execution flow

```text
First frame lays out form -> post-frame callback starts controller
                                      |
                                      v
                           curve produces progress
                              /             \
                             v               v
                          opacity         slide offset
                              \             /
                               transitions render

Submit -> validators -> demo confirmation (valid input only)
```

### ASCII phone preview

```text
+---------------------------+
|      Welcome back         |
+---------------------------+
|          [lock]           |
|          Sign in          |
|  [ Email              ]   |
|  [ Password           ]   |
|  [      Continue      ]   |
+---------------------------+
```

### Example explanation

The form fades in while sliding a small fraction of its own height. `SlideTransition` uses relative units, so the initial distance scales with the form rather than being a fixed number of pixels. `SingleChildScrollView` ensures the button remains reachable when the keyboard appears.

If the post-frame callback is removed, the controller remains at its initial value and the form is invisible. If `mounted` is removed, a delayed callback could try to use a controller after the page has been disposed. If the scroll view is removed, a short screen or keyboard can cause overflow. If validators or labels are removed, the form becomes less usable and less accessible.

**Common mistakes:** Do not animate the login button so late that users must wait to submit. Do not use animation as a substitute for validation or secure authentication. Do not remove labels when adding hint text.

**Professional use case:** Login and account setup screens can use a short entrance transition, while authentication itself remains a separate, secure application flow.

## Complete Example 2

Show a validation message with `AnimatedSize` so the form adjusts smoothly when an error appears.

```dart
AnimatedSize(
  duration: const Duration(milliseconds: 180),
  curve: Curves.easeOut,
  child: errorMessage == null
      ? const SizedBox.shrink()
      : Text(errorMessage!),
)
```

When the error widget changes height, `AnimatedSize` animates the layout size. Keep the error text concise and ensure that the field remains associated with its validation message.

**Widget tree:** `Form field → AnimatedSize → error text`.  
**Flow:** validation failure → error appears → animated layout expands → following fields move down.

If the wrapper is removed, the error appears abruptly. For dense forms or rapidly changing errors, profile and test layout stability.

## Complete Example 3

Animate submission feedback with a state-driven progress indicator.

```dart
AnimatedSwitcher(
  duration: const Duration(milliseconds: 180),
  child: isSubmitting
      ? const CircularProgressIndicator(key: ValueKey('loading'))
      : const Text('Continue', key: ValueKey('label')),
)
```

Distinct keys tell `AnimatedSwitcher` that the child changed, allowing an outgoing and incoming child to transition. Disable repeated submission while the request is active and restore the button on success or error.

**Widget tree:** button area → `AnimatedSwitcher` → progress indicator or label.  
**Flow:** request begins → submitting state becomes true → child identity changes → transition swaps views.

If the child types and keys do not distinguish states, the switcher may update the existing element rather than animate between distinct children.

## Advanced Example

For production authentication, use secure transport and a trusted authentication service; never embed secrets in the app. Keep network requests outside animation listeners. Model idle, submitting, success, and error states explicitly, prevent duplicate submissions, preserve form accessibility, and provide error recovery.

Respect the operating system's reduced-motion preference if the app offers a global motion policy. Ensure the page remains usable without entrance motion and avoid delaying focus or keyboard behavior.

## Under the Hood

The controller emits timeline values through a ticker. `CurvedAnimation` changes timing, and the tween maps progress to `Offset` values. `FadeTransition` and `SlideTransition` listen directly to those animations. The form's validation is separate state and runs only on submission.

`SingleChildScrollView` responds to viewport constraints as the keyboard opens. The animation does not make the screen intrinsically responsive; scroll and width constraints do that work.

## Performance Notes

- Animate the form as a single cohesive group rather than every field separately.
- Avoid expensive blur or oversized shadows on the full-screen form.
- Keep validation and authentication work out of per-frame callbacks.
- Use constraints and scrolling rather than fixed screen heights.
- Profile low-end devices if the screen contains image-heavy decoration.

## Best Practices

- Keep the animation brief and non-blocking.
- Keep labels visible and validation messages actionable.
- Make the page scrollable with the keyboard open.
- Separate UI animation from authentication state and networking.
- Prevent duplicate submissions and show progress accessibly.
- Make error, success, and reduced-motion states usable.

## Common Mistakes

- Making the entire form invisible if the entrance animation does not start.
- Animating layout with fixed heights that overflow small screens.
- Using unlabeled fields or hint text as the only label.
- Allowing repeated submissions while loading.
- Storing credentials unsafely or treating a UI demo as authentication.
- Delaying focus or action until animation completes.
- Forgetting controller disposal.

## Debugging

- If the form remains invisible, check that the controller starts and reaches its end.
- If a delayed-start callback throws, check the `mounted` guard and disposal.
- If the keyboard causes overflow, confirm `SafeArea` and scroll behavior.
- If validation messages overlap, inspect field constraints and `AnimatedSize`.
- If switcher does not animate, verify child identity and keys.
- Test keyboard navigation, screen-reader labels, and error announcements.

## Interview Questions

1. **Why use both `FadeTransition` and `SlideTransition`?** They compose distinct visual properties from a shared timeline.
2. **What unit does `SlideTransition` use?** Fractional offsets relative to the child's dimensions.
3. **Why start after the first frame?** The page can render its initial state before the entrance motion begins.
4. **Why guard a delayed callback with `mounted`?** To avoid acting on a state object that has been disposed.
5. **Why use `SingleChildScrollView`?** To keep content reachable under reduced height, such as when the keyboard opens.
6. **How does `AnimatedSwitcher` distinguish children?** Element identity/type and keys determine whether a child is replaced.
7. **Should authentication be tied to animation completion?** No; business logic and security must not depend on decorative timing.

## Try It Yourself

1. Add a “Forgot password?” action.
2. Add an animated loading state and disable submission while loading.
3. Move the logo and form in with staggered intervals.
4. Test landscape and large text scale.
5. Add an error state that can be dismissed or corrected.

## Challenge Project

Build an account sign-in screen with entrance motion, validation, a submitting state, and an error state. Keep it scrollable with the keyboard open, expose errors to assistive technology, prevent duplicate submissions, and include a reduced-motion path. Use a mock service only; do not embed credentials or secrets.

## Quick Revision Sheet

- Use one controller to synchronize entrance effects.
- `SlideTransition` offsets are relative to child size.
- Start an entry animation after first layout if needed.
- Guard delayed callbacks with `mounted`.
- Keep forms scrollable and labeled.
- Keep animation separate from validation and authentication.
- Use keys for distinct `AnimatedSwitcher` children.
- Dispose the controller and test keyboard/accessibility behavior.
