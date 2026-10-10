# Selector in Flutter

## 📖 Introduction

In the previous chapter, we mastered `Consumer<T>`, which confines rebuilds to a localized subtree whenever the provided object notifies changes.

However, consider this common scenario:
You have a `UserProfile` model with 15 different properties: `username`, `email`, `avatarUrl`, `isOnline`, `unreadMessagesCount`, `followerCount`, `biography`, and more.

You build a tiny badge in the corner of the screen that displays **only the `unreadMessagesCount`**.

If you use `Consumer<UserProfile>`:
Every time the user updates their biography, changes their avatar, or toggles their online status, `notifyListeners()` is called.
And what happens? **Your unread messages badge rebuilds!**
Even though the number of unread messages did not change at all!

How can you tell Flutter:

> *"Only rebuild this widget if the UNREAD MESSAGES COUNT changes! If any other property in the model changes, completely IGNORE the notification and do NOT rebuild!"*

The answer is **`Selector<A, S>`**.

`Selector<A, S>` is an advanced filtering widget from the `provider` package. It extracts a specific selected value (`S`) from a larger model (`A`). It compares the previously selected value with the new value using deep equality. If and only if the selected value changed, it executes its `builder` callback.

In this chapter, we will master the mechanics of `Selector`, analyze how its equality comparison works under the hood, compare it with `Consumer` and `context.select`, and build an ultra-high-performance social media feed dashboard.

---

## 🤔 Why Do We Need It?

Let's look at the performance cost of a large model:

```dart
class UserModel extends ChangeNotifier {
  String name = 'Alice';
  int likes = 42;
  double scrollOffset = 120.0; // Updates at 60-120 FPS during scroll!
}
```

Now, imagine a simple widget displaying only the user's name:

### With `Consumer<UserModel>`:
Whenever `scrollOffset` updates during a smooth swipe, `notifyListeners()` fires 120 times per second.
The widget displaying `Text(user.name)` will **rebuild 120 times per second**!
This wastes massive CPU cycles, causes heating, and drops animation frames.

### With `Selector<UserModel, String>`:
```dart
Selector<UserModel, String>(
  // 1. SELECT: Extract ONLY the property we care about
  selector: (context, user) => user.name,

  // 2. BUILD: Executes ONLY when name actually changes!
  builder: (context, name, child) {
    return Text('User: $name');
  },
)
```

Now, `scrollOffset` and `likes` can change 1,000 times. Flutter checks `oldName == newName` $\rightarrow$ `true`!
**The `builder` does NOT run! Zero rebuilds occur!**

---

## 🌍 Real-Life Analogy

```
┌────────────────────────────────────────────────────────┐
│                   THE AIRPORT FLIGHT MONITOR           │
│                                                        │
│  [Consumer - Answering Every Page on the Intercom]:    │
│  The airport speaker announces: "Flight 101 delayed,   │
│  Flight 205 boarding, Flight 308 at Gate 4..."         │
│  You run to the gate desk on EVERY single announcement │
│  even though your flight is Flight 999!                │
│                                                        │
│  [Selector - Noise-Cancelling Headphones with Alert]:  │
│  You put on smart headphones programmed with one rule: │
│  "Ignore all announcements UNLESS they say Flight 999".│
│  You read your book in complete silence until your     │
│  flight is called.                                     │
└────────────────────────────────────────────────────────┘
```

A `Selector` is a customized noise filter. It filters out all the chatter and notifications from a model, only tapping your widget on the shoulder when the specific piece of data you care about changes.

---

## 🧠 Internal Working: The Selection & Equality Engine

How does `Selector` filter updates?

Let’s inspect Flutter's internal implementation of `Selector` in the `provider` package:

```dart
class _SelectorState<A, S> extends State<Selector<A, S>> {
  late S _selected;

  @override
  Widget build(BuildContext context) {
    // 1. Get current model
    final A value = context.watch<A>();

    // 2. Run the selector function to extract subset
    final S selected = widget.selector(context, value);

    // 3. Compare old selected vs new selected!
    final bool shouldInvalidate = widget.shouldRebuild != null
        ? widget.shouldRebuild!(_selected, selected)
        : !const DeepCollectionEquality().equals(_selected, selected);

    if (shouldInvalidate) {
      _selected = selected;
    }

    // 4. Return widget (rebuilt only if invalidated!)
    return widget.builder(context, _selected, widget.child);
  }
}
```

### The 4-Step Decision Pipeline:
1. **Extraction**: Whenever the parent model calls `notifyListeners()`, `Selector` catches the update and runs `widget.selector(context, model)` to get the new value `S`.
2. **Comparison**: It compares `oldSelected` with `newSelected`. By default, it uses **`DeepCollectionEquality`** (which even checks elements inside lists or maps!).
3. **Filtering**:
   - If `oldSelected == newSelected` $\rightarrow$ It **skips** the `builder` callback and reuses the previous widget element!
   - If `oldSelected != newSelected` $\rightarrow$ It updates its cache and executes `builder(context, selected, child)`.

---

## ⚙ Syntax: Anatomy of `Selector`

```dart
Selector<ModelType, SelectedType>(
  // 1. SELECTOR: Pure function extracting the target property
  selector: (BuildContext context, ModelType model) => model.targetProperty,

  // 2. BUILDER: Builds UI using the extracted value
  builder: (BuildContext context, SelectedType selectedValue, Widget? child) {
    return Text('$selectedValue');
  },

  // 3. OPTIONAL CUSTOM COMPARISON
  shouldRebuild: (previous, next) => previous != next,

  // 4. OPTIONAL STATIC CHILD
  child: const ExpensiveStaticIcon(),
)
```

### Multi-Selector Variants:
Just like `Consumer`, if you need to select from multiple models or multiple fields:
- **`Selector<A, S>`**: Selects from 1 model.
- **`Selector2<A, B, S>`**: Selects from 2 models.

---

## 📋 Comparison: `Consumer` vs. `Selector` vs. `context.select`

| Feature | `Consumer<T>` | `Selector<A, S>` | `context.select<A, S>()` |
| :--- | :--- | :--- | :--- |
| **Rebuild Trigger** | ANY change in `T` | ONLY when selected `S` changes | ONLY when selected `S` changes |
| **Rebuild Scope** | Localized to its builder | Localized to its builder | Entire enclosing `build()` method |
| **Has `child` Optimization?** | ✔ Yes | ✔ Yes | ❌ No |
| **Boilerplate** | Low | Moderate | Lowest |
| **Best For** | Widgets reading multiple fields of a model | High-frequency updates or large complex models | Quick inline selections in small leaf widgets |

---

## 💻 Complete Runnable Example

The following standalone example demonstrates **`Selector` vs. `Consumer` performance**:
- Implements `CustomSelector<A, S>` built on pure framework primitives.
- Models an **eCommerce Dashboard & Live Stock Ticker**:
  - `StoreModel`: Manages `visitorCount` (changes every second!), `cartCount`, and `storeName`.
- Displays three cards:
  1. **Card 1 (`Consumer`)**: Rebuilds on EVERY tick (wasteful!).
  2. **Card 2 (`Selector`)**: Selects `cartCount`. It **completely ignores** the live visitor count and rebuilds *only* when an item is added to the cart!
  3. **Card 3 (`Selector`)**: Selects `storeName`. Never rebuilds!

```dart
import 'dart:async';
import 'package:flutter/material.dart';

void main() {
  runApp(const SelectorMasteryApp());
}

// ============================================================================
// 1. REACTIVE STATE MODEL (ChangeNotifier with high-frequency updates)
// ============================================================================
class StoreModel extends ChangeNotifier {
  String _storeName = 'Apex Tech Hub';
  int _visitorCount = 1420;
  int _cartCount = 0;

  String get storeName => _storeName;
  int get visitorCount => _visitorCount;
  int get cartCount => _cartCount;

  void incrementVisitors() {
    _visitorCount++;
    notifyListeners(); // High-frequency notification!
  }

  void addToCart() {
    _cartCount++;
    notifyListeners(); // Business event
  }

  void updateStoreName(String newName) {
    _storeName = newName;
    notifyListeners();
  }
}

// ============================================================================
// 2. PURE DART IMPLEMENTATION OF Selector<A, S>
// ============================================================================
class CustomProvider<T extends ChangeNotifier> extends InheritedNotifier<T> {
  const CustomProvider({
    super.key,
    required T super.notifier,
    required super.child,
  });

  static T of<T extends ChangeNotifier>(BuildContext context) {
    return context.dependOnInheritedWidgetOfExactType<CustomProvider<T>>()!.notifier!;
  }
}

/// Standalone pure implementation of Selector<A, S>
class CustomSelector<A extends ChangeNotifier, S> extends StatefulWidget {
  final S Function(BuildContext context, A model) selector;
  final Widget Function(BuildContext context, S selected, Widget? child) builder;
  final Widget? child;

  const CustomSelector({
    super.key,
    required this.selector,
    required this.builder,
    this.child,
  });

  @override
  State<CustomSelector<A, S>> createState() => _CustomSelectorState<A, S>();
}

class _CustomSelectorState<A extends ChangeNotifier, S>
    extends State<CustomSelector<A, S>> {
  S? _previousSelectedValue;
  Widget? _cachedBuiltWidget;

  @override
  Widget build(BuildContext context) {
    // 1. Access the model
    final A model = CustomProvider.of<A>(context);

    // 2. Extract selected property
    final S newSelectedValue = widget.selector(context, model);

    // 3. Check equality
    final bool hasChanged = _previousSelectedValue == null ||
        _previousSelectedValue != newSelectedValue;

    if (hasChanged) {
      _previousSelectedValue = newSelectedValue;
      // Re-run builder only when value changed!
      _cachedBuiltWidget = widget.builder(context, newSelectedValue, widget.child);
    }

    return _cachedBuiltWidget!;
  }
}

// ============================================================================
// 3. ROOT APPLICATION
// ============================================================================
class SelectorMasteryApp extends StatefulWidget {
  const SelectorMasteryApp({super.key});

  @override
  State<SelectorMasteryApp> createState() => _SelectorMasteryAppState();
}

class _SelectorMasteryAppState extends State<SelectorMasteryApp> {
  final StoreModel _store = StoreModel();
  Timer? _liveVisitorTimer;

  @override
  void initState() {
    super.initState();
    // Simulate live visitor counter ticking every 800ms
    _liveVisitorTimer = Timer.periodic(const Duration(milliseconds: 800), (_) {
      _store.incrementVisitors();
    });
  }

  @override
  void dispose() {
    _liveVisitorTimer?.cancel();
    _store.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return CustomProvider<StoreModel>(
      notifier: _store,
      child: MaterialApp(
        title: 'Selector Mastery',
        debugShowCheckedModeBanner: false,
        theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
        home: const PerformanceComparisonScreen(),
      ),
    );
  }
}

// ============================================================================
// 4. PERFORMANCE COMPARISON SCREEN
// ============================================================================
class PerformanceComparisonScreen extends StatelessWidget {
  const PerformanceComparisonScreen({super.key});

  @override
  Widget build(BuildContext context) {
    final store = CustomProvider.of<StoreModel>(context);

    return Scaffold(
      appBar: AppBar(
        title: const Text('Selector vs Consumer Demo'),
        centerTitle: true,
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(24.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: [
            // CARD 1: LIVE VISITOR STREAM (High frequency)
            CustomSelector<StoreModel, int>(
              selector: (context, model) => model.visitorCount,
              builder: (context, visitors, _) {
                print('👥 [VisitorSelector] Rebuilt (Count: $visitors)');
                return Card(
                  color: Colors.blue.shade50,
                  child: Padding(
                    padding: const EdgeInsets.all(16.0),
                    child: Row(
                      mainAxisAlignment: MainAxisAlignment.spaceBetween,
                      children: [
                        const Text('Live Active Visitors (800ms tick):', style: TextStyle(fontWeight: FontWeight.bold)),
                        Text('$visitors', style: const TextStyle(fontSize: 22, fontWeight: FontWeight.bold, color: Colors.blue)),
                      ],
                    ),
                  ),
                );
              },
            ),
            const SizedBox(height: 20),

            // CARD 2: SHOPPING CART WITH SELECTOR (FILTERS OUT VISITOR TICKS!)
            CustomSelector<StoreModel, int>(
              // CRITICAL: We select ONLY cartCount!
              selector: (context, model) => model.cartCount,
              child: const Icon(Icons.shopping_bag, size: 36, color: Colors.teal),
              builder: (context, cartItems, staticIcon) {
                // Notice this print statement!
                print('🛒 [CartSelector] Rebuilt! (Cart Items: $cartItems)');

                return Card(
                  elevation: 4,
                  shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(16)),
                  child: Padding(
                    padding: const EdgeInsets.all(20.0),
                    child: Column(
                      children: [
                        Row(
                          children: [
                            if (staticIcon != null) staticIcon,
                            const SizedBox(width: 16),
                            Expanded(
                              child: Column(
                                crossAxisAlignment: CrossAxisAlignment.start,
                                children: [
                                  const Text('Shopping Cart Widget', style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold)),
                                  const Text('Filtered with Selector: Ignores visitor ticks!', style: TextStyle(color: Colors.grey, fontSize: 12)),
                                ],
                              ),
                            ),
                            Text('$cartItems', style: const TextStyle(fontSize: 32, fontWeight: FontWeight.bold, color: Colors.teal)),
                          ],
                        ),
                      ],
                    ),
                  ),
                );
              },
            ),
            const SizedBox(height: 32),

            // ACTION BUTTON
            FilledButton.icon(
              onPressed: store.addToCart,
              icon: const Icon(Icons.add_shopping_cart),
              label: const Text('Add Product to Cart'),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## 🔍 Line-by-Line Example Explanation

- **Lines 10–35 (`StoreModel`)**:
  - `_visitorCount`: Incremented every 800 milliseconds by a timer.
  - `_cartCount`: Incremented only when the user taps the button.
  - Every timer tick calls `notifyListeners()`.
- **Lines 49–87 (`CustomSelector<A, S>`)**:
  - Subclasses `StatefulWidget`.
  - In its `build()` method, calls `widget.selector(context, model)`.
  - Compares `_previousSelectedValue != newSelectedValue`.
  - If they are equal, it **returns `_cachedBuiltWidget`**, skipping the builder callback entirely!
- **Lines 147–167 (`VisitorSelector`)**:
  - `selector: (context, model) => model.visitorCount`
  - Rebuilds every 800ms because the visitor count changes.
- **Lines 170–211 (`CartSelector`)**:
  - `selector: (context, model) => model.cartCount`
  - **Observe the console logs**:
    - `👥 [VisitorSelector]` prints continuously every 800ms.
    - `🛒 [CartSelector]` **DOES NOT PRINT**!
    - Only when you tap **"Add Product to Cart"** does `🛒 [CartSelector]` execute once!
  - `Selector` completely filtered out hundreds of visitor updates from reaching the cart widget!

---

## 📱 ASCII Phone Screen Preview

```
┌──────────────────────────────────────┐
│       Selector vs Consumer Demo      │
├──────────────────────────────────────┤
│ ┌──────────────────────────────────┐ │
│ │ Live Active Visitors:       1428 │ │  <-- Rebuilds every 800ms
│ └──────────────────────────────────┘ │
│                                      │
│ ┌──────────────────────────────────┐ │
│ │ [🛍️] Shopping Cart Widget        │ │  <-- Ignores visitor ticks!
│ │     Filtered with Selector       │ │      Cart Count: 0
│ └──────────────────────────────────┘ │      Zero redundant rebuilds!
│                                      │
│ [ + Add Product to Cart ]            │
└──────────────────────────────────────┘
                   │ User taps "Add Product"
                   ▼
┌──────────────────────────────────────┐
│ ┌──────────────────────────────────┐ │
│ │ [🛍️] Shopping Cart Widget        │ │  <-- Cart Count: 1
│ │     Filtered with Selector       │ │      Rebuilds ONLY now!
│ └──────────────────────────────────┘ │
└──────────────────────────────────────┘
```

---

## 🌳 Widget Tree

```
CustomProvider<StoreModel>
 └── MaterialApp
      └── PerformanceComparisonScreen (StatelessWidget)
           └── Scaffold
                └── SingleChildScrollView
                     └── Column
                          ├── CustomSelector<StoreModel, int> (Visitor Count)
                          │    └── Card (Rebuilds on 800ms tick)
                          ├── CustomSelector<StoreModel, int> (Cart Count)
                          │    └── Card (Filtered: Rebuilds ONLY on add to cart!)
                          └── FilledButton (Add Product)
```

---

## ⚙ Flutter Execution Flow

```
1. Background Timer increments `visitorCount` (1420 -> 1421)
                             │
                             ▼
2. `StoreModel.notifyListeners()` runs
                             │
                             ▼
3. Provider alerts both CustomSelector elements
                             │
                             ▼
4. VisitorSelector runs selector: `1420 != 1421` -> TRUE!
   --> Rebuilds Visitor Card!
                             │
                             ▼
5. CartSelector runs selector: `0 == 0` -> FALSE!
   --> Re-uses cached widget without calling builder!
   --> Cart Card is 100% UNTOUCHED!
```

---

## ⚠ Common Beginner Mistakes

### 1. Returning a New Collection Instance in the Selector

❌ **Fatal Bug**:
```dart
Selector<TodoListModel, List<Todo>>(
  // BUG: Creates a brand new List on EVERY notifyListeners call!
  selector: (context, model) => model.todos.where((t) => t.isDone).toList(),
  builder: ...,
)
```

Because `.toList()` creates a new list instance at a new memory address, default reference equality checks will conclude that the selected value changed every single time, defeating the entire purpose of `Selector`!

✔ **Correct**: Return an immutable record, a scalar count (`model.completedCount`), or use a deep equality `shouldRebuild` comparator.

---

### 2. Mutating the Selected Object in Place

❌ **Wrong**: Returning an object whose internal fields mutated without changing its reference. Dart's `==` check will compare the object with itself and return `true`, causing `Selector` to falsely skip the rebuild!
✔ **Correct**: Always use immutable data models (or Dart 3 records) for selector return types.

---

### 3. Using `Selector` When the Entire Model is Consumed

If a widget genuinely displays 8 out of the 10 fields in a model, writing a complex tuple selector is unnecessary overhead. Standard `Consumer<T>` is simpler and more readable.

---

## 🚀 Professional Usage: Selecting Dart 3 Records

With Dart 3 records (tuples), you can select multiple fields simultaneously without custom classes or multi-selectors:

```dart
Selector<UserModel, (String, bool)>(
  // Selects BOTH username and isOnline in a lightweight record!
  selector: (context, user) => (user.username, user.isOnline),
  builder: (context, record, child) {
    final (name, isOnline) = record;
    return UserStatusRow(name: name, isOnline: isOnline);
  },
)
```

Dart records support structural value equality out of the box, making this the cleanest selective rebuild pattern in modern Flutter!

---

## 💡 Best Practices Checklist

- [ ] **Use `Selector` when a model updates frequently** (timers, scroll offsets, GPS coordinates, animations).
- [ ] **Return immutable primitives or Dart 3 records** from the `selector` function.
- [ ] **Avoid creating new list instances** inside `selector` unless using deep collection equality.
- [ ] **Leverage the `child` argument** for static icons and headers.
- [ ] **Verify rebuild isolation** using debug console logs or Flutter DevTools.

---

## 🧪 Try It Yourself

1. Watch the live active visitors count up automatically in Card 1.
2. Check your debug console: Verify that `🛒 [CartSelector]` does not print during visitor ticks.
3. Tap **"Add Product to Cart"** and confirm that `🛒 [CartSelector]` prints and updates the badge to 1.

---

## 🎯 Challenge

Build a **Stock Trading Ticker**:
- Create a `CryptoPortfolioModel` containing 5 coins: Bitcoin, Ethereum, Solana, Cardano, and Ripple.
- Simulate random price updates ticking every 200ms across all 5 coins.
- Build a `BitcoinTrackerCard` using `Selector` that monitors *only* Bitcoin.
- Confirm that price fluctuations in the other 4 coins never cause the Bitcoin card to re-render!

---

## ❓ Practice Questions

### Q1. What is the fundamental purpose of `Selector<A, S>` in Flutter?
**Answer**: `Selector<A, S>` is an advanced filtering widget from the `provider` package that extracts a specific sub-state `S` from a larger model `A`. It compares the previous value of `S` with the new value; it executes its `builder` callback if and only if the selected value has actually changed, ignoring updates to all other fields in the model.

---

### Q2. How does `Selector` determine if its `builder` callback should re-run?
**Answer**: It evaluates `shouldRebuild(previousSelected, newSelected)`. If no custom `shouldRebuild` function is provided, it uses `DeepCollectionEquality().equals(previous, next)` (or `operator ==`), only triggering a rebuild if the values are not equal.

---

### Q3. What is the difference between `Consumer<T>` and `Selector<A, S>`?
**Answer**: `Consumer<T>` rebuilds whenever `T` calls `notifyListeners()`, regardless of which field changed. `Selector<A, S>` only rebuilds if the specific selected property `S` changes, filtering out notifications caused by other unrelated properties.

---

### Q4. Why is returning `model.items.where(...).toList()` inside the `selector` function an anti-pattern?
**Answer**: Calling `.toList()` allocates a brand new `List` object in memory on every invocation. If shallow equality is used, the memory references will never match, causing `Selector` to rebuild on every notification and completely defeating its filtering optimization.

---

### Q5. What is the difference between `Selector<A, S>` and `context.select<A, S>()`?
**Answer**: Both perform selective rebuilds based on an extracted value. However, `context.select<A, S>()` causes the *entire enclosing widget's `build()` method* to rebuild. `Selector<A, S>` confines the rebuild strictly to its internal `builder` closure and provides a `child` parameter for static subtree optimization.

---

### Q6. How do Dart 3 records (tuples) improve the use of `Selector`?
**Answer**: Dart 3 records allow developers to select multiple fields simultaneously (e.g., `(user.name, user.avatarUrl)`) without creating custom DTO classes, and records naturally implement value equality out of the box.

---

### Q7. When is `Selector` NOT recommended?
**Answer**: When a widget consumes almost all properties of a model, or when the model updates very infrequently. In those cases, the overhead of the selection function and equality comparison outweighs the benefits, and a standard `Consumer` is preferred.

---

### Q8. What is the role of the `shouldRebuild` parameter in `Selector`?
**Answer**: `shouldRebuild: (previous, next) => bool` allows developers to provide custom comparison logic (e.g., ignoring case differences in strings or ignoring price fluctuations under 1%).

---

### Q9. Can `Selector` be used with `child` parameter optimization?
**Answer**: Yes. Just like `Consumer` and `ValueListenableBuilder`, `Selector` accepts an optional `child` widget that is created once and passed into `builder`, avoiding rebuilds of static subtrees.

---

### Q10. What performance issue does `Selector` prevent in high-frequency applications?
**Answer**: In apps with rapid state changes (e.g., scroll listeners, GPS coordinates, media playback progress, live chat tickers), `Selector` prevents low-priority UI widgets from rebuilding 60-120 times per second, eliminating CPU bottlenecks, battery drain, and frame drops (jank).

---

## 📝 Summary

- **Surgical Precision**: `Selector<A, S>` extracts property `S` from model `A` and rebuilds only when `S` changes.
- **Noise Filtering**: Eliminates redundant rebuilds caused by unrelated property updates in large models.
- **Deep Equality Guard**: Automatically compares previous vs. new values before scheduling a frame.
- **Dart 3 Records**: Combine multiple fields into clean tuple selectors `(user.id, user.name)`.
- **High-Frequency Shield**: Indispensable for smooth 60/120 FPS performance in apps with live streaming, animations, or scrolling telemetry.
