# Navigation Quick Revision & Architectural Cheat Sheet

## 📌 Executive Overview

Navigation in Flutter is the mechanism by which users transition between different visual contexts, pages, dialogs, drawers, and modal sheets. At its foundation, Flutter models navigation as a **LIFO (Last-In, First-Out) stack of `Route` objects managed by a `NavigatorState` widget**, which directly coordinates with an underlying `Overlay`.

This chapter serves as your master high-yield cheat sheet, quick reference guide, and architectural decision matrix for the entire `05_Navigation` module.

---

## 🗺️ Master Navigation Decision Matrix

```
                               Need to change screen?
                                         │
                 ┌───────────────────────┴──────────────────────┐
                 ▼                                              ▼
        Replacing current screen?                        Stacking a new screen?
                 │                                              │
        ┌────────┴────────┐                            ┌────────┴────────┐
        ▼                 ▼                            ▼                 ▼
Replace single?     Clear entire history?         Need result back?   Standard drill-down?
  (e.g., Splash)      (e.g., Logout)             (e.g., Picker)      (e.g., Item Details)
        │                 │                            │                 │
        ▼                 ▼                            ▼                 ▼
 `pushReplacement` `pushAndRemoveUntil`          `await push(...)`    `push(...)`
```

| Use Case | Recommended Navigation Method | Stack Effect | Typical Scenario |
| :--- | :--- | :--- | :--- |
| **Drill-down / Details** | `Navigator.push()` / `pushNamed()` | Pushes $[B]$ onto $[A] \rightarrow [A, B]$ | Tapping a product card to view specifications. |
| **Back Button / Dismiss** | `Navigator.pop([result])` | Pops top $[B] \rightarrow [A]$ | Closing a modal, returning from settings. |
| **Splash to Auth / Home** | `Navigator.pushReplacement()` | Replaces top $[A]$ with $[B] \rightarrow [B]$ | Transitioning from animated splash to login. |
| **Authentication Exit / Logout** | `Navigator.pushNamedAndRemoveUntil(..., (r) => false)` | Clears entire stack, mounts new root | Logging out; user must not back-button into dashboard. |
| **Conditional Multi-Step Back** | `Navigator.popUntil(context, ModalRoute.withName('/home'))` | Pops routes until matching predicate | Canceling a 4-step wizard and returning home. |
| **Data Selection Dialog** | `final res = await Navigator.push(...)` | Pauses caller until `pop(value)` is called | Color picker, address selector, avatar chooser. |
| **Drawer Navigation** | `Navigator.pop(context)` then `pushNamed()` | Closes drawer panel before pushing new page | Hamburger menu item taps. |
| **Persistent Bottom Nav** | `IndexedStack` or Nested `Navigator` | Preserves sub-routes without reloading | Multi-tab apps (Instagram, YouTube). |

---

## ⚡ High-Yield Syntax Cheat Sheet

### 1. Imperative Stack Operations (Navigator 1.0)

```dart
// 1. Basic Push
Navigator.push(
  context,
  MaterialPageRoute(builder: (context) => const DetailScreen()),
);

// 2. Pop with Result
Navigator.pop(context, 'Success');

// 3. Await Result
final result = await Navigator.push<String>(
  context,
  MaterialPageRoute(builder: (context) => const SelectionScreen()),
);

// 4. Push Replacement
Navigator.pushReplacement(
  context,
  MaterialPageRoute(builder: (context) => const HomeScreen()),
);

// 5. Push and Remove All Previous Routes
Navigator.pushAndRemoveUntil(
  context,
  MaterialPageRoute(builder: (context) => const LoginScreen()),
  (Route<dynamic> route) => false, // false = discard all previous
);

// 6. Pop Until Specific Route Name
Navigator.popUntil(context, ModalRoute.withName('/dashboard'));
```

---

### 2. Centralized Named Routes (`onGenerateRoute`)

```dart
MaterialApp(
  initialRoute: AppRoutes.home,
  onGenerateRoute: (RouteSettings settings) {
    switch (settings.name) {
      case AppRoutes.home:
        return MaterialPageRoute(
          settings: settings,
          builder: (_) => const HomeScreen(),
        );

      case AppRoutes.profile:
        final args = settings.arguments as ProfileArguments;
        return MaterialPageRoute(
          settings: settings,
          builder: (_) => ProfileScreen(args: args),
        );

      default:
        return MaterialPageRoute(
          settings: settings,
          builder: (_) => NotFoundScreen(route: settings.name),
        );
    }
  },
);
```

---

### 3. Drawer Structure

```dart
Scaffold(
  appBar: AppBar(title: const Text('Home')),
  drawer: Drawer(
    child: ListView(
      padding: EdgeInsets.zero,
      children: [
        const UserAccountsDrawerHeader(
          accountName: Text('John Doe'),
          accountEmail: Text('john@example.com'),
          currentAccountPicture: CircleAvatar(child: Text('JD')),
        ),
        ListTile(
          leading: const Icon(Icons.settings),
          title: const Text('Settings'),
          onTap: () {
            Navigator.pop(context); // 1. Close drawer
            Navigator.pushNamed(context, '/settings'); // 2. Navigate
          },
        ),
      ],
    ),
  ),
);
```

---

### 4. BottomNavigationBar with State Switching

```dart
class TabScaffold extends StatefulWidget {
  const TabScaffold({super.key});

  @override
  State<TabScaffold> createState() => _TabScaffoldState();
}

class _TabScaffoldState extends State<TabScaffold> {
  int _currentIndex = 0;

  static const List<Widget> _screens = [
    FeedScreen(),
    SearchScreen(),
    ProfileScreen(),
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: IndexedStack( // Preserves scroll position & state across tabs
        index: _currentIndex,
        children: _screens,
      ),
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: _currentIndex,
        onTap: (index) => setState(() => _currentIndex = index),
        items: const [
          BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Feed'),
          BottomNavigationBarItem(icon: Icon(Icons.search), label: 'Search'),
          BottomNavigationBarItem(icon: Icon(Icons.person), label: 'Profile'),
        ],
      ),
    );
  }
}
```

---

### 5. TabBar & TabBarView (DefaultTabController)

```dart
DefaultTabController(
  length: 3,
  child: Scaffold(
    appBar: AppBar(
      title: const Text('Orders'),
      bottom: const TabBar(
        tabs: [
          Tab(icon: Icon(Icons.pending), text: 'Pending'),
          Tab(icon: Icon(Icons.local_shipping), text: 'Shipped'),
          Tab(icon: Icon(Icons.check_circle), text: 'Delivered'),
        ],
      ),
    ),
    body: const TabBarView(
      children: [
        PendingOrdersList(),
        ShippedOrdersList(),
        DeliveredOrdersList(),
      ],
    ),
  ),
)
```

---

### 6. Nested Navigation (Independent Tab Stacks)

```dart
// Each tab maintains its own independent Navigator widget
Navigator(
  key: tabNavigatorKey,
  onGenerateRoute: (RouteSettings settings) {
    return MaterialPageRoute(
      settings: settings,
      builder: (BuildContext context) => const TabRootScreen(),
    );
  },
)
```

---

## 🧠 Core Mental Models & Architecture

### The Route Stack Lifecycle

```
[Screen C]  <-- Topmost Route (Active screen receiving touch events)
    │
[Screen B]  <-- Intermediate Route (Kept alive in memory, inactive)
    │
[Screen A]  <-- Root Route (Base of navigation stack)
    │
 Navigator  <-- Coordinates animations, transitions, and overlays
```

- When you **`push`**, a new `Route` is layered on top. The previous route is paused and occluded.
- When you **`pop`**, the top `Route` plays its exit transition animation (e.g., slide right or fade out) and its widget element is unmounted and disposed from memory.
- An **`Overlay`** sits inside the `Navigator`. Every `Route` inserts one or more `OverlayEntry` instances into this overlay to display its visuals.

---

## ⚠ Top 10 Critical Gotchas & How to Fix Them

| # | Gotcha / Mistake | Symptoms | The Fix |
| :---: | :--- | :--- | :--- |
| **1** | **Missing `context.mounted` check** | Crash: `Looking up a deactivated widget's ancestor is unsafe`. | Add `if (!context.mounted) return;` immediately following every `await` statement. |
| **2** | **Forgetting to pop Drawer before navigation** | Back button reopens drawer instead of exiting or popping page. | Call `Navigator.pop(context)` first inside `onTap` before calling `push()`. |
| **3** | **Black screen on `Navigator.pop()`** | App turns completely pitch black. | Root route was popped! Always verify `Navigator.canPop(context)` before popping programmatically. |
| **4** | **TabBar without TabController** | Crash: `No TabController for TabBar`. | Wrap the `Scaffold` with `DefaultTabController(length: N, ...)` or initialize an explicit `TabController` with `TickerProviderStateMixin`. |
| **5** | **TabBar length mismatch** | Crash: `Controller's length (3) does not match TabBar's length (2)`. | Ensure `DefaultTabController.length` equals the exact count of items in `TabBar.tabs` and `TabBarView.children`. |
| **6** | **Unbounded stack memory leak** | App slows down, lags, or crashes with OOM error. | Use `pushReplacement()` or `pushNamedAndRemoveUntil()` at auth and checkout transitions to purge obsolete history. |
| **7** | **Raw string typos in named routes** | App throws unhandled exception on route resolution. | Use static constants (e.g., `AppRoutes.home`) instead of string literals (`'/home'`). |
| **8** | **Casting `arguments` to wrong type** | Crash: `type 'Null' is not a subtype of type 'Map'`. | Use strongly typed DTO classes and defensive `is` checks inside `onGenerateRoute`. |
| **9** | **Nested Navigator back button exits whole app** | Pressing back closes the application instead of popping the nested tab. | Wrap nested Navigators in `PopScope` and delegate `maybePop()` to the nested navigator key. |
| **10** | **Using `Navigator.of(context)` inside root MaterialApp context** | Crash: `Navigator operation requested with a context that does not include a Navigator`. | The `context` passed to `build` in `MaterialApp` is above the `Navigator`. Use a builder or child widget below `MaterialApp`. |

---

## 📊 Summary Comparison: Navigation Paradigms

```
┌────────────────────────────────────────────────────────────────────────┐
│                      NAVIGATION PARADIGM COMPARISON                    │
├─────────────────────┬───────────────────┬──────────────────────────────┤
│ Feature             │ Navigator 1.0     │ GoRouter / Declarative Nav   │
├─────────────────────┼───────────────────┼──────────────────────────────┤
│ Mechanism           │ Imperative Stack  │ Declarative URL Sync         │
│ API Simplicity      │ ★★★★★ High        │ ★★★☆☆ Moderate               │
│ Flutter Web URLs    │ ★★☆☆☆ Poor        │ ★★★★★ Seamless               │
│ Deep Linking        │ ★★★☆☆ Basic       │ ★★★★★ Native & Reactive      │
│ State Restoration   │ Manual            │ Automatic                    │
│ Best For            │ Pure mobile apps  │ Cross-platform (Web/Desktop) │
└─────────────────────┴───────────────────┴──────────────────────────────┘
```

---

## 📝 Key Takeaway Checklist Before Production

- [ ] All route names are defined as constants inside an `AppRoutes` class.
- [ ] Every asynchronous gap is guarded with `if (!context.mounted) return;`.
- [ ] Authentication logout wipes the entire navigation stack via `pushNamedAndRemoveUntil`.
- [ ] Deep linking schemes are declared in Android manifest and iOS Info.plist.
- [ ] Persistent bottom tabs utilize `IndexedStack` to prevent rebuilding children on every tab switch.
- [ ] Error 404 pages are configured in `onUnknownRoute`.
