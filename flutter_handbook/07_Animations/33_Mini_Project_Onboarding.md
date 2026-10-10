# Mini Project: Animated Onboarding

## Introduction

Onboarding introduces a new user to the main value of an app. This project creates a small three-page onboarding flow with a page indicator and a button that advances to the next page.

The `PageView` already provides swipe navigation. A page-change callback updates the selected page index, and `AnimatedContainer` animates the indicator. This keeps the implementation simple while still giving clear motion feedback.

## Why this concept exists

Onboarding can feel like a static slideshow if the interface does not communicate progress. A moving indicator, clear next action, and natural page transition help users understand how many steps remain and what to do next.

Animation should support comprehension, not delay it. Keep transitions short and provide a way to skip or finish.

## Real-world analogy

An onboarding flow is like a short guided tour through an unfamiliar building. Each page reveals one landmark, while the progress markers tell you where you are and how much of the tour remains.

## Internal Working

`PageView` uses a scrollable viewport and page physics to move between pages. User swipes and `PageController.animateToPage` both result in scroll position changes. `onPageChanged` reports the settled page index.

The page index is ordinary widget state. Updating it rebuilds the indicator row, where each `AnimatedContainer` interpolates its width and color. The page movement itself is handled by the scrollable. The page controller is disposed with the state object.

## Syntax

```dart
PageView(
  controller: pageController,
  onPageChanged: (index) => setState(() => currentPage = index),
  children: pages,
)
```

`animateToPage` accepts the destination index, duration, and curve. Clamp or validate the index if it comes from external data; the button logic should never request a page outside the page count.

## Complete Example 1

This app provides three onboarding pages, swipe navigation, a changing dot indicator, and a Next/Start action.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const OnboardingApp());

class OnboardingApp extends StatelessWidget {
  const OnboardingApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      theme: ThemeData(useMaterial3: true, colorSchemeSeed: Colors.deepPurple),
      home: const OnboardingPage(),
    );
  }
}

class OnboardingPage extends StatefulWidget {
  const OnboardingPage({super.key});

  @override
  State<OnboardingPage> createState() => _OnboardingPageState();
}

class _OnboardingPageState extends State<OnboardingPage> {
  final PageController _pageController = PageController();
  int _currentPage = 0;

  final List<_OnboardingItem> _items = const [
    _OnboardingItem(
      icon: Icons.explore,
      title: 'Discover',
      description: 'Find ideas and tools that fit the way you work.',
      color: Colors.deepPurple,
    ),
    _OnboardingItem(
      icon: Icons.tune,
      title: 'Make it yours',
      description: 'Choose the settings that help you stay focused.',
      color: Colors.teal,
    ),
    _OnboardingItem(
      icon: Icons.rocket_launch,
      title: 'Get started',
      description: 'You are ready to begin your first project.',
      color: Colors.orange,
    ),
  ];

  @override
  void dispose() {
    _pageController.dispose();
    super.dispose();
  }

  Future<void> _advance() async {
    if (_currentPage < _items.length - 1) {
      await _pageController.animateToPage(
        _currentPage + 1,
        duration: const Duration(milliseconds: 350),
        curve: Curves.easeInOut,
      );
    } else {
      if (!mounted) return;
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text('Onboarding complete')),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    final isLastPage = _currentPage == _items.length - 1;

    return Scaffold(
      appBar: AppBar(
        actions: [
          TextButton(
            onPressed: () => _pageController.animateToPage(
              _items.length - 1,
              duration: const Duration(milliseconds: 300),
              curve: Curves.easeOut,
            ),
            child: const Text('Skip'),
          ),
        ],
      ),
      body: Column(
        children: [
          Expanded(
            child: PageView.builder(
              controller: _pageController,
              itemCount: _items.length,
              onPageChanged: (index) => setState(() => _currentPage = index),
              itemBuilder: (context, index) {
                final item = _items[index];
                return _OnboardingSlide(item: item);
              },
            ),
          ),
          Row(
            mainAxisAlignment: MainAxisAlignment.center,
            children: List.generate(_items.length, (index) {
              final selected = index == _currentPage;
              return AnimatedContainer(
                duration: const Duration(milliseconds: 250),
                curve: Curves.easeOut,
                width: selected ? 26 : 8,
                height: 8,
                margin: const EdgeInsets.symmetric(horizontal: 4),
                decoration: BoxDecoration(
                  color: selected ? Colors.deepPurple : Colors.black26,
                  borderRadius: BorderRadius.circular(8),
                ),
              );
            }),
          ),
          Padding(
            padding: const EdgeInsets.all(24),
            child: SizedBox(
              width: double.infinity,
              child: FilledButton(
                onPressed: _advance,
                child: Text(isLastPage ? 'Start' : 'Next'),
              ),
            ),
          ),
        ],
      ),
    );
  }
}

class _OnboardingItem {
  const _OnboardingItem({
    required this.icon,
    required this.title,
    required this.description,
    required this.color,
  });

  final IconData icon;
  final String title;
  final String description;
  final Color color;
}

class _OnboardingSlide extends StatelessWidget {
  const _OnboardingSlide({required this.item});

  final _OnboardingItem item;

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.all(32),
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          CircleAvatar(
            radius: 72,
            backgroundColor: item.color.withValues(alpha: 0.14),
            child: Icon(item.icon, size: 64, color: item.color),
          ),
          const SizedBox(height: 32),
          Text(
            item.title,
            textAlign: TextAlign.center,
            style: Theme.of(context).textTheme.headlineMedium,
          ),
          const SizedBox(height: 12),
          Text(
            item.description,
            textAlign: TextAlign.center,
            style: Theme.of(context).textTheme.bodyLarge,
          ),
        ],
      ),
    );
  }
}

```

### Line-by-line explanation

- `main` runs the onboarding app and applies a Material theme.
- `OnboardingPage` is stateful because it owns a page controller and current-page index.
- `_pageController` controls page navigation and is disposed with the state.
- `_currentPage` determines which indicator is active and whether the button reads Next or Start.
- `_items` stores immutable page data: icon, title, description, and color.
- `dispose` releases the page controller.
- `_advance` animates to the next page when available. On the last page, it checks `mounted` before using the context after the asynchronous step and shows a completion message.
- The `AppBar` exposes Skip, which animates directly to the final page.
- `Expanded` gives the `PageView` the available vertical space above the indicators and button.
- `PageView.builder` lazily creates visible slides, reports the selected index, and supports swipe gestures.
- The indicator `Row` generates one animated pill per page. Each pill changes width and color based on `_currentPage`.
- `Padding`, `SizedBox`, and `FilledButton` create a wide, touch-friendly action.
- `_OnboardingItem` provides typed data instead of parallel lists.
- `_OnboardingSlide` lays out the icon, heading, and body copy with consistent spacing.
- `withValues(alpha: 0.14)` creates a translucent background from the page color on current Flutter versions.

### Widget tree

```text
MaterialApp
└── OnboardingPage
    └── Scaffold
        ├── AppBar(Skip)
        └── Column
            ├── Expanded
            │   └── PageView
            │       └── _OnboardingSlide
            ├── Row
            │   └── AnimatedContainer indicators
            └── FilledButton
```

### Execution flow

```text
Swipe or Next tap -> PageController scrolls to page
                           |
                           v
                    onPageChanged(index)
                           |
                           v
                  currentPage state updates
                     /             \
                    v               v
           indicator animates   button label updates
```

### ASCII phone preview

```text
+---------------------------+
|                    Skip   |
|                           |
|          (icon)           |
|                           |
|         Discover          |
| Find ideas and tools      |
| that fit your work.       |
|                           |
|          --- .. ..        |
|      [       Next       ] |
+---------------------------+
```

### Example explanation

`PageView` owns the horizontal page-scroll animation. This chapter adds a separate implicit animation to the progress indicator. The current page index is the shared source of truth for the indicator and button label. The button awaits page navigation only as a convenience; the page index itself updates from `onPageChanged`, including when the user swipes.

If `onPageChanged` is removed, swiping can move the page while the indicator and button state remain stale. If the controller is not disposed, its scroll resources may outlive the page. If `Expanded` is removed, the page view may receive unbounded height in the column and cause a layout error. **Common mistakes:** Do not keep separate page indexes for the button and indicator. Make the final action real in an app rather than leaving users at a dead-end snackbar. Ensure content fits at large text scale and in landscape.

**Professional use case:** First-run onboarding, feature tours, and short setup wizards can use this structure with a persistence flag to avoid showing onboarding on every launch.

## Complete Example 2

Fade page content as its page becomes selected by wrapping a page's content in `AnimatedOpacity` and using the current index as the target.

```dart
AnimatedOpacity(
  opacity: currentPage == index ? 1 : 0.65,
  duration: const Duration(milliseconds: 200),
  child: pageContent,
)
```

This effect can help emphasize the current page in a custom non-scrollable carousel. With a normal `PageView`, pages move in and out of the viewport; avoid layering extra fades unless the design benefits and performance has been checked.

**Widget tree:** carousel/page → `AnimatedOpacity` → content.  
**Flow:** selected index changes → opacity target changes → content fades to emphasis.

If the page remains interactive while dimmed, keep hit testing and semantics in mind. An animation is not a substitute for correct page selection.

## Complete Example 3

Animate a welcome illustration with a one-time `ScaleTransition`. Create a controller in state, call `forward` after the first frame if the animation should run on page entry, and dispose the controller.

```dart
ScaleTransition(
  scale: CurvedAnimation(parent: controller, curve: Curves.elasticOut),
  child: const Icon(Icons.rocket_launch, size: 96),
)
```

The curve creates a playful overshoot. Use sparingly; `elasticOut` may be distracting and can exceed the nominal scale range.

**Widget tree:** `ScaleTransition → Icon`.  
**Flow:** route/page appears → controller starts → curved scale changes → illustration settles.

If the controller is never started, the icon stays at the beginning value. If it is created inside `build`, rebuilds can restart it and leak tickers.

## Advanced Example

In a production onboarding flow, persist completion, make Skip and Back behavior explicit, support system back navigation, and avoid requiring users to wait through animation. Use meaningful semantic labels and ensure progress dots are not the only indication of page position.

If page content has network images, precache critical assets to reduce first-display jank. Use localized text and responsive layouts; fixed heights can fail for longer translations or large text scale. Test the final button action, re-entry behavior, and state restoration if the app supports it.

## Under the Hood

`PageView` is built on scrollable page mechanics. It changes scroll position over time either from drag physics or an animated controller request. `onPageChanged` reports which page is considered current. The indicator's `AnimatedContainer`s retarget when the state changes.

The page controller is a resource associated with the `State`; disposing it prevents stale ownership. Only visible or cached pages need to be built by the lazy builder.

## Performance Notes

- Use `PageView.builder` for larger page collections.
- Keep page content lightweight and avoid unnecessary nested scrolling.
- Precache large critical images when appropriate.
- Avoid animating all off-screen page content.
- Test first-frame loading and the slowest supported device.

## Best Practices

- Keep progress state derived from `onPageChanged`.
- Provide a clear Skip, Back, Next, and completion path as product requires.
- Use concise copy, readable contrast, and scalable layouts.
- Make animation short and purposeful.
- Persist completion appropriately.
- Test gestures and buttons, not just a single path through the pages.

## Common Mistakes

- Forgetting `Expanded` or fixed constraints around `PageView` inside a `Column`.
- Updating page index only from the button, not from swipes.
- Forgetting to dispose `PageController`.
- Leaving final action unimplemented.
- Making indicator dots too small or inaccessible.
- Using hard-coded vertical sizes that overflow with larger text.
- Overusing animated content on every page.

## Debugging

- If the page view overflows, inspect its constraints in the column.
- If the indicator is stale, check `onPageChanged` and state updates.
- If navigation jumps, verify page index bounds and controller attachment.
- If `animateToPage` runs before attachment, ensure the page is mounted and controller has clients.
- Check large-font and landscape layouts.
- Verify Skip and Next end on the expected page.

## Interview Questions

1. **What does `PageController` control?** The scroll position and page navigation of a `PageView`.
2. **Why use `onPageChanged`?** It keeps app state synchronized for swipes as well as programmatic navigation.
3. **Why does a `PageView` in a `Column` often need `Expanded`?** It needs a bounded height constraint.
4. **What does `AnimatedContainer` add to the indicator?** It interpolates target dimensions and colors.
5. **Why dispose a page controller?** To release resources tied to that scrollable.
6. **What should Skip do?** Navigate to a defined destination and honor the product's completion behavior.

## Try It Yourself

1. Add a Back button after the first page.
2. Animate illustration opacity and scale together.
3. Add a page counter such as “2 of 3”.
4. Try long localized descriptions and larger text scale.
5. Persist onboarding completion using the app's chosen storage approach.

## Challenge Project

Build a four-step onboarding flow for a budgeting app. Each page introduces one task, supports swipe and buttons, displays accessible progress, and ends with a functional start action. Ensure content adapts to small screens and large text.

## Quick Revision Sheet

- `PageView` handles swipe paging; `PageController` performs programmatic navigation.
- Derive progress from `onPageChanged`.
- Use `Expanded` to bound `PageView` in a `Column`.
- Animate the indicator as its target index changes.
- Dispose controllers.
- Implement Skip and final actions, and test text scale and small screens.
