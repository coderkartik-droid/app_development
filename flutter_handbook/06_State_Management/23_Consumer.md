# Consumer in Flutter

## 📖 Introduction

In the previous chapters, we learned how to read data from a provider using `context.watch<T>()`. While `context.watch<T>()` is concise, it has a significant architectural side effect:

> **Calling `context.watch<T>()` subscribes the ENTIRE enclosing `build()` method to rebuild whenever the data changes.**

If you have a 300-line `build()` method with complex layouts, headers, animations, and lists, and you write `final count = context.watch<Counter>().count;` on line 2, **every single widget in that 300-line method will rebuild** on every tick!

How can you localize the rebuild so that **only the exact 10-pixel text widget updates**, while the other 290 lines of layout remain untouched?

The answer is **`Consumer<T>`**.

`Consumer<T>` is a dedicated widget provided by the `provider` package. It searches up the widget tree for an ancestor `Provider<T>`, extracts the value, and passes it into its own `builder` callback.

By wrapping only the specific widget that needs the data with `Consumer<T>`, you achieve **pinpoint rebuild isolation**. Furthermore, just like `ValueListenableBuilder`, `Consumer` includes an optional **`child` optimization parameter** that allows you to reuse static subtrees without rebuilding them.

In this chapter, we will master the architecture of `Consumer`, understand its relationship with `context.watch`, explore multi-type variants (`Consumer2`, `Consumer3`), and build a high-performance interactive news feed application.

---

## 🤔 Why Do We Need It?

Let's compare the rebuild scope of `context.watch` vs. `Consumer`:

### The Problem with `context.watch()` on a Large Screen:
```dart
class ArticleScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // ❌ Subscribes the ENTIRE ArticleScreen!
    final bookmarks = context.watch<BookmarkModel>();

    return Scaffold(
      appBar: AppBar(title: Text('Tech Daily')),        // REBUILDS!
      body: Column(
        children: [
          HugeHeroImage(),                               // REBUILDS!
          ComplexArticleBodyParagraphs(),                // REBUILDS!
          CommentsSection(),                             // REBUILDS!
          // The ONLY widget that actually needed bookmarks:
          BookmarkButton(isBookmarked: bookmarks.isSaved),
        ],
      ),
    );
  }
}
```

When the user taps the bookmark icon, the hero image, the entire article text, and the comments section are all rebuilt unnecessarily!

### The `Consumer` Solution:
```dart
class ArticleScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // Zero subscriptions on ArticleScreen! This method NEVER rebuilds!
    return Scaffold(
      appBar: AppBar(title: Text('Tech Daily')),
      body: Column(
        children: [
          HugeHeroImage(),
          ComplexArticleBodyParagraphs(),
          CommentsSection(),
          // ✔ ONLY this tiny Consumer rebuilds!
          Consumer<BookmarkModel>(
            builder: (context, bookmarks, child) {
              return BookmarkButton(isBookmarked: bookmarks.isSaved);
            },
          ),
        ],
      ),
    );
  }
}
```

Now, toggling a bookmark executes only the 3 lines inside `Consumer.builder`. The hero image, paragraphs, and comments are 100% untouched!

---

## 🌍 Real-Life Analogy

```
┌────────────────────────────────────────────────────────┐
│                   THE RESTAURANT KITCHEN BELL          │
│                                                        │
│  [context.watch() - Shouting in the Dining Room]:      │
│  The chef rings the bell and screams into the dining   │
│  room: "Table 4's soup is ready!"                      │
│  Every single guest in the restaurant stops eating,    │
│  looks up, and turns around, even though only Table 4  │
│  ordered the soup.                                     │
│                                                        │
│  [Consumer - Vibrating Table Pager]:                   │
│  Table 4 is handed a small electronic pager.          │
│  When the soup is ready, ONLY Table 4's pager vibrates.│
│  The rest of the restaurant continues dining peacefully│
│  in silence.                                           │
└────────────────────────────────────────────────────────┘
```

A `Consumer` is an electronic buzzer on a restaurant table. Only the customer holding that specific buzzer is alerted when their order is ready; everyone else in the restaurant continues eating uninterrupted.

---

## 🧠 Internal Working: Framework Implementation

What is `Consumer` actually doing behind the scenes?

Let’s inspect the implementation of `Consumer` inside the `provider` package:

```dart
class Consumer<T> extends SingleChildStatelessWidget {
  Consumer({
    super.key,
    required this.builder,
    super.child,
  });

  final Widget Function(BuildContext context, T value, Widget? child) builder;

  @override
  Widget buildWithChild(BuildContext context, Widget? child) {
    // IT SIMPLY CALLS Provider.of<T>(context) UNDER THE HOOD!
    return builder(
      context,
      Provider.of<T>(context), // listen: true (subscribes THIS micro-widget!)
      child,
    );
  }
}
```

### The Big Architectural Insight:
`Consumer` is **not magic**! It is literally a micro-widget whose sole job is to call `Provider.of<T>(context)` (which is equivalent to `context.watch<T>()`) inside its own tiny `buildWithChild()` method!

By inserting a tiny new `BuildContext` at the leaf level, Flutter's dependency tracker binds the rebuild to **that micro-element only**, sparing the large parent widget above it!

---

## ⚙ Syntax: Anatomy of `Consumer`

```dart
Consumer<CartModel>(
  // 1. The builder callback
  builder: (BuildContext context, CartModel cart, Widget? child) {
    return Row(
      children: [
        Text('Items in cart: ${cart.totalItems}'),
        // 2. Reusing the static child!
        if (child != null) child,
      ],
    );
  },
  // 3. Optional static child optimization (built ONCE)
  child: const Icon(Icons.shopping_cart),
)
```

### Understanding the 3 Parameters of `builder`:
1. **`BuildContext context`**: The localized context of the `Consumer` widget.
2. **`T value`** (`CartModel cart`): The current instance of the provided model.
3. **`Widget? child`**: The pre-built widget passed into `Consumer.child`.

---

## 📋 Multi-Consumer Variants

What if your UI needs data from **two** or **three** different providers at the same time?
Instead of nesting multiple `Consumer` widgets, the `provider` package provides multi-type variants:

| Widget | Purpose | Signature |
| :--- | :--- | :--- |
| **`Consumer<A>`** | Listens to 1 provider | `builder: (context, a, child)` |
| **`Consumer2<A, B>`** | Listens to 2 providers | `builder: (context, a, b, child)` |
| **`Consumer3<A, B, C>`** | Listens to 3 providers | `builder: (context, a, b, c, child)` |
| **`Consumer4`, `5`, `6`** | Listens up to 6 providers | Eliminates deep nesting pyramids |

---

## 💻 Complete Runnable Example

The following standalone example demonstrates **`Consumer` and `Consumer2` in action**:
- Implements `CustomConsumer<T>` and `CustomConsumer2<A, B>` built on pure framework primitives.
- Models a **High-Performance News Reader**:
  - `ThemeController`: Manages dark/light theme.
  - `ArticleFavoritesController`: Tracks favorited articles.
- The parent screen contains a large, expensive header image and paragraph text.
- When the user toggles a favorite, **ONLY the bookmark button rebuilds**. The screen header and paragraphs are never marked dirty!

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ConsumerMasteryApp());
}

// ============================================================================
// 1. DATA MODELS & NOTIFIERS
// ============================================================================
class ThemeController extends ChangeNotifier {
  bool _isDark = false;
  bool get isDark => _isDark;

  void toggleTheme() {
    _isDark = !_isDark;
    notifyListeners();
  }
}

class FavoritesController extends ChangeNotifier {
  final Set<String> _bookmarkedIds = {};

  bool isBookmarked(String id) => _bookmarkedIds.contains(id);

  void toggleBookmark(String id) {
    if (_bookmarkedIds.contains(id)) {
      _bookmarkedIds.remove(id);
    } else {
      _bookmarkedIds.add(id);
    }
    notifyListeners();
  }
}

// ============================================================================
// 2. PURE DART IMPLEMENTATION OF Consumer & Consumer2
// ============================================================================
class CustomProvider<T extends ChangeNotifier> extends InheritedNotifier<T> {
  const CustomProvider({
    super.key,
    required T super.notifier,
    required super.child,
  });

  static T of<T extends ChangeNotifier>(BuildContext context) {
    final provider = context.dependOnInheritedWidgetOfExactType<CustomProvider<T>>();
    assert(provider != null, 'No CustomProvider<$T> found in context!');
    return provider!.notifier!;
  }
}

/// Standalone implementation of Consumer<T>
class CustomConsumer<T extends ChangeNotifier> extends StatelessWidget {
  final Widget Function(BuildContext context, T value, Widget? child) builder;
  final Widget? child;

  const CustomConsumer({
    super.key,
    required this.builder,
    this.child,
  });

  @override
  Widget build(BuildContext context) {
    // Scopes the rebuild strictly to this micro-widget!
    final value = CustomProvider.of<T>(context);
    return builder(context, value, child);
  }
}

/// Standalone implementation of Consumer2<A, B>
class CustomConsumer2<A extends ChangeNotifier, B extends ChangeNotifier>
    extends StatelessWidget {
  final Widget Function(BuildContext context, A a, B b, Widget? child) builder;
  final Widget? child;

  const CustomConsumer2({
    super.key,
    required this.builder,
    this.child,
  });

  @override
  Widget build(BuildContext context) {
    final a = CustomProvider.of<A>(context);
    final b = CustomProvider.of<B>(context);
    return builder(context, a, b, child);
  }
}

// ============================================================================
// 3. ROOT APPLICATION
// ============================================================================
class ConsumerMasteryApp extends StatefulWidget {
  const ConsumerMasteryApp({super.key});

  @override
  State<ConsumerMasteryApp> createState() => _ConsumerMasteryAppState();
}

class _ConsumerMasteryAppState extends State<ConsumerMasteryApp> {
  final ThemeController _theme = ThemeController();
  final FavoritesController _favorites = FavoritesController();

  @override
  void dispose() {
    _theme.dispose();
    _favorites.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    // Nesting two providers
    return CustomProvider<ThemeController>(
      notifier: _theme,
      child: CustomProvider<FavoritesController>(
        notifier: _favorites,
        child: CustomConsumer<ThemeController>(
          builder: (context, theme, _) {
            return MaterialApp(
              title: 'Consumer Mastery',
              debugShowCheckedModeBanner: false,
              themeMode: theme.isDark ? ThemeMode.dark : ThemeMode.light,
              theme: ThemeData(colorSchemeSeed: Colors.indigo, brightness: Brightness.light, useMaterial3: true),
              darkTheme: ThemeData(colorSchemeSeed: Colors.indigo, brightness: Brightness.dark, useMaterial3: true),
              home: const ArticleReaderScreen(),
            );
          },
        ),
      ),
    );
  }
}

// ============================================================================
// 4. ARTICLE SCREEN (Stateless - Never Rebuilds When Bookmarks Toggle!)
// ============================================================================
class ArticleReaderScreen extends StatelessWidget {
  const ArticleReaderScreen({super.key});

  static const String articleId = 'flutter_3_22_deep_dive';

  @override
  Widget build(BuildContext context) {
    print('🎨 [ArticleReaderScreen] build() executed (Screen Rebuild)');

    return Scaffold(
      appBar: AppBar(
        title: const Text('Tech Dispatch Weekly'),
        actions: [
          // Theme switch consumer
          CustomConsumer<ThemeController>(
            builder: (context, theme, _) {
              return IconButton(
                icon: Icon(theme.isDark ? Icons.light_mode : Icons.dark_mode),
                onPressed: theme.toggleTheme,
              );
            },
          ),
        ],
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(20.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // EXPENSIVE HEADER CARD (Built once, never rebuilt on bookmark toggle!)
            Card(
              clipBehavior: Clip.antiAlias,
              child: Container(
                height: 140,
                width: double.infinity,
                color: Colors.indigo.shade700,
                padding: const EdgeInsets.all(20),
                alignment: Alignment.bottomLeft,
                child: const Text(
                  'Deep Dive: High Performance Flutter Architecture',
                  style: TextStyle(color: Colors.white, fontSize: 20, fontWeight: FontWeight.bold),
                ),
              ),
            ),
            const SizedBox(height: 16),

            // INTERACTIVE TOOLBAR WITH CONSUMER2
            Card(
              elevation: 2,
              child: Padding(
                padding: const EdgeInsets.symmetric(horizontal: 16.0, vertical: 8.0),
                child: Row(
                  mainAxisAlignment: MainAxisAlignment.spaceBetween,
                  children: [
                    const Text('Article Actions:', style: TextStyle(fontWeight: FontWeight.bold)),

                    // ========================================================
                    // CONSUMER2: Listens to BOTH Favorites & Theme
                    // ========================================================
                    CustomConsumer2<FavoritesController, ThemeController>(
                      // STATIC CHILD OPTIMIZATION: Tooltip is pre-built!
                      child: const Text('Save to Reading List', style: TextStyle(fontSize: 12)),
                      builder: (context, favorites, theme, staticLabel) {
                        final isSaved = favorites.isBookmarked(articleId);

                        print('⭐ [BookmarkConsumer] Localized rebuild executed!');

                        return Row(
                          children: [
                            if (staticLabel != null) staticLabel,
                            const SizedBox(width: 8),
                            IconButton.filledTonal(
                              icon: Icon(
                                isSaved ? Icons.bookmark : Icons.bookmark_border,
                                color: isSaved ? Colors.amber.shade800 : null,
                              ),
                              onPressed: () => favorites.toggleBookmark(articleId),
                            ),
                          ],
                        );
                      },
                    ),
                  ],
                ),
              ),
            ),
            const SizedBox(height: 20),

            // LONG STATIC BODY TEXT
            const Text(
              'The Consumer widget represents the pinnacle of rebuild isolation in the Provider ecosystem. '
              'By encapsulating calls to dependOnInheritedWidgetOfExactType inside a localized StatelessWidget, '
              'it constructs a micro-element boundary in the Element tree.\n\n'
              'When the underlying state notifies changes, Flutter traverses down to the micro-element and re-runs '
              'only the builder closure. Ancestor layouts, adjacent paragraphs, hero images, and complex multi-child '
              'columns are completely bypassed, preventing frame drops during high-frequency user interactions.',
              style: TextStyle(fontSize: 15, height: 1.5),
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

- **Lines 44–58 (`CustomConsumer<T>`)**:
  - Implements the exact pattern of the official `Consumer`.
  - In its `build()` method, calls `CustomProvider.of<T>(context)`.
  - This registers **only this `CustomConsumer` element** as a dependent of the provider!
- **Lines 61–79 (`CustomConsumer2<A, B>`)**:
  - Queries both `A` and `B` providers simultaneously.
  - Passes both instances into `builder(context, a, b, child)`.
- **Lines 140–146 (`ThemeController` Consumer)**:
  - Positioned in the AppBar. Rebuilds only the icon button when dark mode is toggled.
- **Lines 179–205 (`CustomConsumer2` in Article Toolbar)**:
  - Notice the **`child` parameter**: `child: const Text('Save to Reading List')`.
  - The text is passed into `builder` as `staticLabel`. It is created **once** and never rebuilt.
  - Tapping the bookmark button triggers `favorites.toggleBookmark(articleId)`.
  - Check your debug console: You will see `⭐ [BookmarkConsumer] Localized rebuild executed!`, but `🎨 [ArticleReaderScreen] build()` is **NEVER printed again**!

---

## 📱 ASCII Phone Screen Preview

```
┌──────────────────────────────────────┐
│        Tech Dispatch Weekly      [🌙]│  <-- Theme toggle
├──────────────────────────────────────┤
│ ┌──────────────────────────────────┐ │
│ │ [██████████████████████████████] │ │  <-- Huge Header Card
│ │ Deep Dive: High Performance...   │ │      (Never rebuilt!)
│ └──────────────────────────────────┘ │
│                                      │
│ ┌──────────────────────────────────┐ │
│ │ Article Actions:                 │ │
│ │ Save to Reading List     [ 🔖 ]  │ │  <-- ONLY this icon rebuilds!
│ └──────────────────────────────────┘ │      (Toggled to saved)
│                                      │
│ The Consumer widget represents the   │  <-- Static Body Text
│ pinnacle of rebuild isolation in...  │      (Never rebuilt!)
└──────────────────────────────────────┘
```

---

## 🌳 Widget Tree

```
CustomProvider<ThemeController>
 └── CustomProvider<FavoritesController>
      └── MaterialApp
           └── ArticleReaderScreen (Stateless - Never Rebuilds on Bookmark!)
                └── Scaffold
                     ├── AppBar (Contains Consumer<ThemeController>)
                     └── SingleChildScrollView
                          └── Column
                               ├── Card (Huge Header - Never Rebuilt!)
                               ├── Card (Toolbar)
                               │    └── CustomConsumer2<Favorites, Theme>
                               │         └── Row (staticLabel, Bookmark IconButton)
                               └── Text (Long Paragraphs - Never Rebuilt!)
```

---

## ⚙ Flutter Execution Flow

```
User taps Bookmark IconButton
              │
              ▼
1. Event calls `favorites.toggleBookmark(articleId)`
              │
              ▼
2. `FavoritesController` adds ID to set and calls `notifyListeners()`
              │
              ▼
3. Provider notifies registered subscriber elements
              │
              ▼
4. ONLY the `CustomConsumer2` micro-element is marked dirty!
   • `ArticleReaderScreen` is NOT dirty!
   • `HugeHeroCard` is NOT dirty!
   • `ParagraphText` is NOT dirty!
              │
              ▼
5. Next frame executes `CustomConsumer2.builder()`:
   • Reads `isSaved = true`
   • Icon changes to filled amber bookmark
   • Reuses `staticLabel` without re-creating it!
```

---

## ⚠ Common Beginner Mistakes

### 1. Wrapping the Entire `build()` Method in a `Consumer`

❌ **Anti-pattern**:
```dart
@override
Widget build(BuildContext context) {
  // DEFEATS THE ENTIRE PURPOSE OF CONSUMER!
  return Consumer<CartModel>(
    builder: (context, cart, _) {
      return Scaffold(
        appBar: AppBar(...),
        body: ..., // Whole screen still rebuilds!
      );
    },
  );
}
```

✔ **Correct**: Put `Consumer` as deep in the widget tree as possible, wrapping **only the exact leaf widget** that consumes the data:
```dart
Scaffold(
  appBar: AppBar(...),
  body: Center(
    child: Consumer<CartModel>(
      builder: (context, cart, _) => Text('${cart.items.length}'),
    ),
  ),
)
```

---

### 2. Not Utilizing the `child` Parameter

❌ **Sub-optimal**: Instantiating static widgets inside the `builder` callback instead of passing them to `child`.
✔ **Correct**: If a portion of the `Consumer` subtree does not depend on the model, pass it to `child` so Flutter reuses it across rebuilds.

---

### 3. Nesting Multiple Consumers Instead of `Consumer2`

❌ **Pyramid of Doom**:
```dart
Consumer<AuthModel>(
  builder: (ctx, auth, _) => Consumer<CartModel>(
    builder: (ctx, cart, _) => Consumer<ThemeModel>(
      builder: (ctx, theme, _) => ...,
    ),
  ),
)
```

✔ **Clean & Flat**:
```dart
Consumer3<AuthModel, CartModel, ThemeModel>(
  builder: (context, auth, cart, theme, child) => ...,
)
```

---

## 🚀 Professional Usage: The "Read-in-Init, Consume-in-Tree" Pattern

When a screen needs to fetch data once on load, but render reactively:

```dart
class OrderHistoryScreen extends StatefulWidget {
  const OrderHistoryScreen({super.key});

  @override
  State<OrderHistoryScreen> createState() => _OrderHistoryScreenState();
}

class _OrderHistoryScreenState extends State<OrderHistoryScreen> {
  @override
  void initState() {
    super.initState();
    // 1. Trigger network fetch once without listening!
    context.read<OrdersViewModel>().fetchOrders();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('My Orders')),
      // 2. React to status changes using Consumer!
      body: Consumer<OrdersViewModel>(
        builder: (context, model, _) {
          if (model.isLoading) return const Center(child: CircularProgressIndicator());
          if (model.hasError) return Center(child: Text(model.errorMessage));
          return ListView.builder(...);
        },
      ),
    );
  }
}
```

---

## 💡 Best Practices Checklist

- [ ] **Push `Consumer` down to leaf widgets**: Keep rebuilds scoped as narrowly as possible.
- [ ] **Use the `child` argument** for static headers, icons, and labels inside the consumer.
- [ ] **Use `Consumer2`, `Consumer3`** to listen to multiple models without horizontal pyramid nesting.
- [ ] **Never wrap the root `Scaffold` in `Consumer`** unless the entire screen genuinely changes.
- [ ] **Use `context.read` for event triggers** and `Consumer` for UI rendering.

---

## 🧪 Try It Yourself

1. Tap the **bookmark button** several times.
2. Open your debug console and observe: `⭐ [BookmarkConsumer]` prints on every tap, while `🎨 [ArticleReaderScreen]` does **not** print, confirming zero screen rebuilds!
3. Tap the **moon icon** in the AppBar and verify that dark mode transitions smoothly while keeping the bookmark state intact.

---

## 🎯 Challenge

Build a **Live Chat Room with Localized Typing Indicator**:
- Create a `ChatViewModel` with a message list and a `bool isFriendTyping` flag.
- Render a `ListView` of 20 messages.
- Use a `Consumer<ChatViewModel>` scoped strictly around a bottom typing indicator banner.
- Add a button that toggles `isFriendTyping`.
- Verify that toggling the typing indicator never rebuilds the 20 chat bubble cards in the `ListView`!

---

## ❓ Practice Questions

### Q1. What is the primary purpose of the `Consumer` widget in the `provider` package?
**Answer**: `Consumer<T>` is a widget that retrieves a value from `Provider<T>` and executes its `builder` callback, localizing the widget rebuild strictly to that specific subtree and preventing parent widgets from rebuilding when the provided data changes.

---

### Q2. How does `Consumer` differ from `context.watch<T>()` in terms of rebuild scope?
**Answer**: `context.watch<T>()` causes the entire enclosing widget's `build()` method to rebuild. `Consumer<T>` isolates the rebuild to only the widgets returned by its internal `builder` function, leaving sibling and parent widgets untouched.

---

### Q3. What are the three parameters of `Consumer`'s `builder` callback?
**Answer**:
1. `BuildContext context`: The build context of the `Consumer` widget.
2. `T value`: The current instance of the provided object.
3. `Widget? child`: The optional pre-built widget passed into `Consumer.child`.

---

### Q4. Why does `Consumer` have a `child` parameter?
**Answer**: The `child` parameter provides performance optimization. It allows developers to pass a static or expensive widget subtree that does not depend on the model. Flutter instantiates this child once and passes it into `builder`, reusing it across rebuilds without re-executing its build method.

---

### Q5. What is `Consumer2<A, B>` and when should you use it?
**Answer**: `Consumer2<A, B>` is a multi-type consumer that listens to two different providers simultaneously. You should use it when a widget requires data from two independent providers, avoiding the nested pyramid indentation of writing a `Consumer` inside another `Consumer`.

---

### Q6. Is `Consumer` implemented as a `StatefulWidget` or a `StatelessWidget`?
**Answer**: In the `provider` package, `Consumer` is a `StatelessWidget` (inheriting from `SingleChildStatelessWidget`). It achieves rebuild isolation simply by inserting a new element node in the tree and calling `Provider.of<T>(context)` inside its own build method.

---

### Q7. Why is wrapping a root `Scaffold` with `Consumer` considered an anti-pattern?
**Answer**: Wrapping the root `Scaffold` in a `Consumer` causes the entire screen (including app bars, background decorations, and static text) to rebuild whenever any minor property changes, completely defeating the purpose of localized rebuild isolation.

---

### Q8. What happens if no matching provider is found above `Consumer<T>` in the widget tree?
**Answer**: A `ProviderNotFoundException` is thrown, indicating that no ancestor provider of type `T` was found for the current context.

---

### Q9. Can you use `Consumer` to listen to an immutable model?
**Answer**: Yes! If an immutable model is provided by a parent `Provider` that is reconstructed with new values, `Consumer` will rebuild when the parent provides a new instance.

---

### Q10. What is the maximum number of providers a single multi-consumer widget can listen to in the `provider` package?
**Answer**: The `provider` package provides built-in multi-consumers up to `Consumer6<A, B, C, D, E, F>`.

---

## 📝 Summary

- **Surgical Rebuild Isolation**: `Consumer<T>` rebuilds only its `builder` closure, sparing parent widgets from unnecessary redraws.
- **The `child` Optimization**: Reuses static subtrees across rebuilds to maximize 60/120 FPS performance.
- **Multi-Model Listening**: Use `Consumer2` through `Consumer6` to listen to multiple providers without nested callback pyramids.
- **Push Down the Tree**: Place `Consumer` as close to the leaf widgets as possible.
- **How It Works**: Under the hood, `Consumer` is a tiny widget calling `Provider.of<T>(context)` within its own localized element.
