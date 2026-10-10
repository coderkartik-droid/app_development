# Navigation Practice Questions & Examination Guide

Welcome to the comprehensive practice questions and examination suite for **Module 05: Navigation**. This chapter contains 32 in-depth questions designed to test and solidify your mastery across theoretical concepts, code tracing, bug hunting, and real-world system architecture in Flutter navigation.

---

## 📑 Table of Contents
1. [Category 1: Conceptual & Multiple Choice Questions (Q1–Q10)](#category-1-conceptual--multiple-choice-questions)
2. [Category 2: Code Tracing & Output Prediction (Q11–Q18)](#category-2-code-tracing--output-prediction)
3. [Category 3: Bug Hunting & Error Correction (Q19–Q26)](#category-3-bug-hunting--error-correction)
4. [Category 4: Scenario-Based Architecture & Design (Q27–Q32)](#category-4-scenario-based-architecture--design)

---

## Category 1: Conceptual & Multiple Choice Questions

### Q1. What data structure does Flutter's `Navigator` use internally to manage routes?
- A) First-In, First-Out (FIFO) Queue
- B) Last-In, First-Out (LIFO) Stack
- C) Doubly-Linked Circular Ring
- D) Binary Search Tree

<details>
<summary>👉 View Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
The `Navigator` operates strictly as a Last-In, First-Out (LIFO) stack. When you call `Navigator.push()`, a new `Route` is placed on top of the stack and becomes the visible screen. When you call `Navigator.pop()`, the topmost route is removed, revealing the route immediately beneath it.
</details>

---

### Q2. Which underlying widget is directly manipulated by `Navigator` to visually layer routes and dialogs?
- A) `Stack`
- B) `Overlay`
- C) `CustomMultiChildLayout`
- D) `RenderObjectWidget`

<details>
<summary>👉 View Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
The `NavigatorState` manages an `Overlay` widget. Each `Route` creates one or more `OverlayEntry` objects that are inserted into the `Overlay`. This allows pages, floating dialogs, tooltips, and transition animations to be layered on top of each other.
</details>

---

### Q3. Why does calling `Navigator.of(context)` inside the `build` method of the widget passed to `MaterialApp(home: ...)` fail with an assertion error?
- A) `MaterialApp` does not support `Navigator.of()` calls.
- B) The `BuildContext` belongs to an element *above* the `Navigator` created internally by `MaterialApp`.
- C) The `Navigator` requires asynchronous initialization before it can be read.
- D) You must always use named routes when building the root screen.

<details>
<summary>👉 View Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
`Navigator.of(context)` searches **upwards** through the widget element tree from the provided `context`. The `context` passed to the widget enclosing `MaterialApp` is an ancestor of `MaterialApp`. Since the `Navigator` is instantiated *inside* `MaterialApp`, searching up from a context outside or level with `MaterialApp` will fail to find it. Wrapping the child in a `Builder` provides a context located *below* the `Navigator`.
</details>

---

### Q4. Which method replaces the current route by pushing a new route and discarding the route immediately below it?
- A) `Navigator.pushAndRemoveUntil()`
- B) `Navigator.pushReplacement()`
- C) `Navigator.popAndPushNamed()`
- D) Both B and C

<details>
<summary>👉 View Answer & Explanation</summary>

**Correct Answer: D**

**Explanation:**
Both `pushReplacement()` and `popAndPushNamed()` replace the topmost screen. `pushReplacement()` pushes the new route while animating the exit of the previous route simultaneously. `popAndPushNamed()` first pops the current route and then pushes the new route.
</details>

---

### Q5. What is the predicate closure `(Route<dynamic> route) => false` in `pushAndRemoveUntil()` responsible for?
- A) Preventing the new route from ever being removed.
- B) Removing all preceding routes in the navigation stack so the new route becomes the root.
- C) Disabling the hardware back button on Android devices.
- D) Canceling the push transition if the network request fails.

<details>
<summary>👉 View Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
`pushAndRemoveUntil(newRoute, predicate)` iterates backwards through existing routes on the stack and applies the predicate function to each route. As long as the predicate returns `false`, that route is removed. Returning `false` for every route completely flushes the stack, leaving `newRoute` as the sole route at the root.
</details>

---

### Q6. Which widget should be used to display an app drawer on the right side of the screen instead of the default left side?
- A) `Scaffold.endDrawer`
- B) `Scaffold.rightDrawer`
- C) `Drawer(alignment: Alignment.topRight)`
- D) `Align(alignment: Alignment.centerRight, child: Drawer())`

<details>
<summary>👉 View Answer & Explanation</summary>

**Correct Answer: A**

**Explanation:**
`Scaffold` provides two dedicated drawer slots: `drawer` (opens from the leading/start side, left in LTR languages) and `endDrawer` (opens from the trailing/end side, right in LTR languages).
</details>

---

### Q7. How can you preserve the scroll position and state of each page when switching tabs in a `BottomNavigationBar`?
- A) Store all widget trees in global variables.
- B) Use an `IndexedStack` as the `Scaffold.body`.
- C) Re-instantiate the screens inside `setState()`.
- D) Set `maintainState: true` on `MaterialPageRoute`.

<details>
<summary>👉 View Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
If you switch screens in `Scaffold.body` using a simple ternary or `List[currentIndex]`, Flutter unmounts and disposes the old screen's `State` when you switch tabs. An `IndexedStack` keeps all child widgets mounted and alive in memory, simply changing which child is painted and receptive to hit tests, thus preserving scroll offsets and input field text.
</details>

---

### Q8. What happens if the `length` property of `DefaultTabController` is 3, but the `TabBar.tabs` list contains 4 tabs?
- A) The 4th tab is silently hidden from view.
- B) The controller automatically updates its length to 4.
- C) An assertion failure is thrown at runtime: `Controller's length (3) does not match the number of tabs (4)`.
- D) The app falls back to a single scrollable tab.

<details>
<summary>👉 View Answer & Explanation</summary>

**Correct Answer: C**

**Explanation:**
`TabBar` and `TabBarView` strictly validate that their item count equals `TabController.length`. Any mismatch immediately throws a runtime assertion error during the widget build/layout phase.
</details>

---

### Q9. What is the modern replacement for the deprecated `WillPopScope` widget in Flutter 3.12+?
- A) `BackScope`
- B) `PopScope`
- C) `NavigatorPopGuard`
- D) `CanPopScope`

<details>
<summary>👉 View Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
`PopScope` was introduced to support modern predictive back gesture animations on Android 14+ and iOS. It uses the `canPop` boolean and `onPopInvokedWithResult` callback instead of an asynchronous boolean-returning callback.
</details>

---

### Q10. What is the fundamental difference between a custom URL Scheme (`myapp://`) and an App Link / Universal Link (`https://myapp.com/`)?
- A) Custom schemes only work on Android; App Links only work on iOS.
- B) App Links / Universal Links require cryptographic domain verification via a hosted JSON file (`assetlinks.json` or `apple-app-site-association`) to prove domain ownership.
- C) Custom schemes require an active internet connection, whereas App Links work offline.
- D) App Links cannot pass query parameters or path variables.

<details>
<summary>👉 View Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
Anyone can register a custom scheme like `myapp://`, leading to potential link hijacking by other apps. App Links (Android) and Universal Links (iOS) use standard `https://` URLs verified against digital asset link configuration files hosted on the domain, ensuring that only verified apps can intercept the domain's traffic without browser disambiguation dialogs.
</details>

---

## Category 2: Code Tracing & Output Prediction

### Q11. What is the return value of `result` when the user taps Button B in the following code?

```dart
// Screen A
void openScreenB(BuildContext context) async {
  final result = await Navigator.push<String>(
    context,
    MaterialPageRoute(builder: (context) => const ScreenB()),
  );
  print('Result received: $result');
}

// Screen B
Widget build(BuildContext context) {
  return Scaffold(
    body: Center(
      child: ElevatedButton(
        onPressed: () {
          Navigator.pop(context, 'Confirmed');
        },
        child: const Text('Confirm'),
      ),
    ),
  );
}
```

- A) `Result received: null`
- B) `Result received: Confirmed`
- C) `Result received: Instance of 'MaterialPageRoute'`
- D) Nothing is printed because `pop` cannot send strings.

<details>
<summary>👉 View Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
`Navigator.pop(context, [result])` passes the optional second argument back to the awaiting `Future` returned by `Navigator.push()`. Since `'Confirmed'` is supplied, `result` resolves to `'Confirmed'` and `Result received: Confirmed` is printed to the console.
</details>

---

### Q12. If the user taps the hardware system back button instead of the button on Screen B in Q11, what is printed?
- A) `Result received: Confirmed`
- B) `Result received: null`
- C) An unhandled `NullThrownError` exception.
- D) The `Future` never completes.

<details>
<summary>👉 View Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
When a route is popped by a system gesture (such as Android back button or iOS swipe back) or by an `AppBar` leading back arrow without an explicit result argument, `Navigator.pop()` defaults to returning `null`.
</details>

---

### Q13. Trace the navigation stack state through the following sequence of operations:
1. App starts at `[Screen A]`
2. `Navigator.push(context, Screen B)`
3. `Navigator.pushReplacement(context, Screen C)`
4. `Navigator.push(context, Screen D)`
5. `Navigator.pop(context)`

What screen is currently displayed?
- A) `Screen A`
- B) `Screen B`
- C) `Screen C`
- D) `Screen D`

<details>
<summary>👉 View Answer & Explanation</summary>

**Correct Answer: C**

**Step-by-Step Trace:**
1. Start: `[A]`
2. Push B: `[A, B]`
3. PushReplacement C: B is replaced by C $\rightarrow$ `[A, C]`
4. Push D: `[A, C, D]`
5. Pop: D is popped off $\rightarrow$ `[A, C]`

The topmost screen is now **`Screen C`**.
</details>

---

### Q14. What does `Navigator.canPop(context)` return when called from `Screen A` in Step 1 of Question 13?
- A) `true`
- B) `false`
- C) `null`
- D) It throws a runtime error.

<details>
<summary>👉 View Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
`Navigator.canPop(context)` checks whether there is more than one route in the stack. When the app is at the root route (`Screen A`), popping would close the entire application and leave a black screen. Therefore, `canPop()` returns `false`.
</details>

---

### Q15. Consider the following route configuration:

```dart
MaterialApp(
  initialRoute: '/home',
  routes: {
    '/home': (context) => const HomeScreen(),
    '/details': (context) => const DetailsScreen(),
  },
  onGenerateRoute: (settings) {
    if (settings.name == '/profile') {
      return MaterialPageRoute(builder: (_) => const ProfileScreen());
    }
    return null;
  },
  onUnknownRoute: (settings) {
    return MaterialPageRoute(builder: (_) => const FallbackScreen());
  },
)
```

If the user triggers `Navigator.pushNamed(context, '/orders')`, which widget is rendered?
- A) `HomeScreen`
- B) `DetailsScreen`
- C) `FallbackScreen`
- D) App crashes with `RouteNotFoundException`

<details>
<summary>👉 View Answer & Explanation</summary>

**Correct Answer: C**

**Explanation:**
Flutter first checks `routes['/orders']` (not found). Then it calls `onGenerateRoute(settings)`. `onGenerateRoute` checks for `'/profile'`, and since it does not match, returns `null`. When both fail, Flutter invokes `onUnknownRoute`, which returns `MaterialPageRoute(builder: (_) => const FallbackScreen())`.
</details>

---

### Q16. What is printed to the console when the following button is clicked?

```dart
onPressed: () {
  Navigator.push(
    context,
    MaterialPageRoute(
      settings: const RouteSettings(arguments: 500),
      builder: (context) {
        final args = ModalRoute.of(context)?.settings.arguments;
        return Text('Arg: $args');
      },
    ),
  );
}
```

- A) `Arg: null`
- B) `Arg: 500`
- C) An assertion error because `RouteSettings` must be a `String`.
- D) `Arg: Instance of RouteSettings`

<details>
<summary>👉 View Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
`RouteSettings` accepts `arguments` of type `Object?`. By passing `arguments: 500` to `MaterialPageRoute`, `ModalRoute.of(context)?.settings.arguments` retrieves `500`.
</details>

---

### Q17. Trace the following nested tab controller index transition:

```dart
final tabController = TabController(length: 3, vsync: this);
tabController.animateTo(2);
```

What is `tabController.index` and `tabController.previousIndex` immediately after the animation completes?
- A) `index: 2`, `previousIndex: 0`
- B) `index: 0`, `previousIndex: 2`
- C) `index: 2`, `previousIndex: 2`
- D) `index: 1`, `previousIndex: 0`

<details>
<summary>👉 View Answer & Explanation</summary>

**Correct Answer: A**

**Explanation:**
Initially, the controller starts at index 0. When animated to index 2, `previousIndex` retains the starting point (`0`) while `index` updates to the active destination (`2`).
</details>

---

### Q18. What will occur if this code executes on an Android device:

```dart
Navigator.popUntil(context, (route) => route.isFirst);
```

- A) The entire app closes and exits to the Android home screen.
- B) The Navigator pops screens one by one until only the very first (root) screen remains.
- C) It pops only the top two screens.
- D) It throws a stack underflow exception.

<details>
<summary>👉 View Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
`route.isFirst` is a property on `Route` that evaluates to `true` only for the bottommost route in the Navigator stack. `popUntil` pops all routes until this predicate evaluates to `true`, effectively bringing the user back to the application's root page.
</details>

---

## Category 3: Bug Hunting & Error Correction

### Q19. Identify the fatal bug in this asynchronous login handler and fix it.

```dart
// BUGGY CODE
void _submitLogin(BuildContext context) async {
  final bool success = await AuthService.login(emailController.text);
  if (success) {
    Navigator.of(context).pushReplacementNamed('/dashboard');
  }
}
```

<details>
<summary>👉 View Answer & Explanation</summary>

**The Bug:**
The asynchronous gap `await AuthService.login()` can take multiple seconds. If the user presses the back button or closes the dialog while the network call is in flight, the `BuildContext` is unmounted. Calling `Navigator.of(context)` across an unmounted async gap throws the Flutter error: `Looking up a deactivated widget's ancestor is unsafe`.

**The Fix:**
Verify `context.mounted` before interacting with the context:
```dart
void _submitLogin(BuildContext context) async {
  final bool success = await AuthService.login(emailController.text);
  
  // Guard check
  if (!context.mounted) return;

  if (success) {
    Navigator.of(context).pushReplacementNamed('/dashboard');
  }
}
```
</details>

---

### Q20. Why does tapping the drawer tile in this code fail to navigate, or cause weird stacking bugs?

```dart
// BUGGY CODE
ListTile(
  leading: const Icon(Icons.settings),
  title: const Text('Settings'),
  onTap: () {
    Navigator.pushNamed(context, '/settings');
  },
)
```

<details>
<summary>👉 View Answer & Explanation</summary>

**The Bug:**
The `Drawer` itself is an active route on top of the `Scaffold`'s page. Pushing `/settings` without first closing the drawer leaves the drawer mounted. When the user returns from `/settings` via the back button, the drawer is still open! In some architectures, it pushes `/settings` on top of the drawer overlay.

**The Fix:**
Always close the drawer using `Navigator.pop(context)` *before* pushing the new route:
```dart
ListTile(
  leading: const Icon(Icons.settings),
  title: const Text('Settings'),
  onTap: () {
    Navigator.pop(context); // 1. Close drawer
    Navigator.pushNamed(context, '/settings'); // 2. Navigate
  },
)
```
</details>

---

### Q21. Spot the bug in this TabBar implementation.

```dart
// BUGGY CODE
class OrderScreen extends StatelessWidget {
  const OrderScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Orders'),
        bottom: const TabBar(
          tabs: [
            Tab(text: 'Active'),
            Tab(text: 'Completed'),
          ],
        ),
      ),
      body: const TabBarView(
        children: [
          Center(child: Text('Active Orders')),
          Center(child: Text('Completed Orders')),
        ],
      ),
    );
  }
}
```

<details>
<summary>👉 View Answer & Explanation</summary>

**The Bug:**
Neither `TabBar` nor `TabBarView` has access to a `TabController`. Running this code causes the runtime assertion: `No TabController for TabBar. When creating a TabBar, you must either provide an explicit TabController using the "controller" property, or you must supply a DefaultTabController ancestor.`

**The Fix:**
Wrap the `Scaffold` with `DefaultTabController`:
```dart
class OrderScreen extends StatelessWidget {
  const OrderScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return DefaultTabController(
      length: 2,
      child: Scaffold(
        appBar: AppBar(
          title: const Text('Orders'),
          bottom: const TabBar(
            tabs: [
              Tab(text: 'Active'),
              Tab(text: 'Completed'),
            ],
          ),
        ),
        body: const TabBarView(
          children: [
            Center(child: Text('Active Orders')),
            Center(child: Text('Completed Orders')),
          ],
        ),
      ),
    );
  }
}
```
</details>

---

### Q22. What causes this `onGenerateRoute` implementation to throw a runtime error when accessing `args.email`?

```dart
// BUGGY CODE
class ProfileArgs {
  final String username;
  final String email;
  ProfileArgs(this.username, this.email);
}

// In onGenerateRoute:
case '/profile':
  final args = settings.arguments as ProfileArgs;
  return MaterialPageRoute(builder: (_) => ProfileScreen(args: args));
```

<details>
<summary>👉 View Answer & Explanation</summary>

**The Bug:**
If a caller triggers `Navigator.pushNamed(context, '/profile')` without supplying `arguments`, or passes a `Map` instead of `ProfileArgs`, the direct forced cast `as ProfileArgs` throws a fatal `TypeError: type 'Null' is not a subtype of type 'ProfileArgs'`.

**The Fix:**
Perform defensive pattern matching or type validation:
```dart
case '/profile':
  final dynamic rawArgs = settings.arguments;
  if (rawArgs is ProfileArgs) {
    return MaterialPageRoute(builder: (_) => ProfileScreen(args: rawArgs));
  }
  return MaterialPageRoute(
    builder: (_) => const Scaffold(
      body: Center(child: Text('Error: Missing or invalid ProfileArgs')),
    ),
  );
```
</details>

---

### Q23. Identify the bug causing all tabs in this `BottomNavigationBar` to reset to page 0 whenever the parent rebuilds.

```dart
// BUGGY CODE
class MainShell extends StatelessWidget {
  const MainShell({super.key});

  @override
  Widget build(BuildContext context) {
    int selectedIndex = 0; // BUG!

    return Scaffold(
      body: IndexedStack(
        index: selectedIndex,
        children: const [FeedScreen(), ExploreScreen(), ProfileScreen()],
      ),
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: selectedIndex,
        onTap: (index) {
          selectedIndex = index;
        },
        items: const [
          BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Home'),
          BottomNavigationBarItem(icon: Icon(Icons.explore), label: 'Explore'),
          BottomNavigationBarItem(icon: Icon(Icons.person), label: 'Profile'),
        ],
      ),
    );
  }
}
```

<details>
<summary>👉 View Answer & Explanation</summary>

**The Bug:**
`MainShell` is a `StatelessWidget`, and `selectedIndex` is declared as a local variable inside the `build` method. Re-assigning `selectedIndex = index` does not trigger a rebuild, and even if rebuilt, `selectedIndex` would immediately re-initialize to `0`.

**The Fix:**
Convert `MainShell` to a `StatefulWidget` and call `setState(() => _selectedIndex = index)`.
</details>

---

### Q24. What is wrong with the following logout logic?

```dart
// BUGGY CODE
void _logout(BuildContext context) {
  Navigator.pushReplacementNamed(context, '/login');
}
```

<details>
<summary>👉 View Answer & Explanation</summary>

**The Bug:**
`pushReplacementNamed` only replaces the **topmost** screen. If the user navigated `Home -> Settings -> Account -> Logout`, calling `pushReplacementNamed('/login')` replaces `Account` with `/login`. If the user presses the Android back button from the `/login` screen, they will pop right back into `Settings` and `Home`!

**The Fix:**
Clear the entire navigation stack so the user cannot navigate back:
```dart
void _logout(BuildContext context) {
  Navigator.pushNamedAndRemoveUntil(
    context,
    '/login',
    (Route<dynamic> route) => false, // Clears everything
  );
}
```
</details>

---

### Q25. Why does this single `TabController` throw a `TickerProvider` assertion error?

```dart
// BUGGY CODE
class MyTabsState extends State<MyTabs> {
  late TabController _controller;

  @override
  void initState() {
    super.initState();
    _controller = TabController(length: 3, vsync: this); // Compile error!
  }
}
```

<details>
<summary>👉 View Answer & Explanation</summary>

**The Bug:**
`TabController` requires a `TickerProvider` as its `vsync` argument to drive smooth tab transition animations. A standard `State` class does not implement `TickerProvider`. Passing `this` causes a type mismatch.

**The Fix:**
Mix in `SingleTickerProviderStateMixin` (for one controller) or `TickerProviderStateMixin` (for multiple):
```dart
class MyTabsState extends State<MyTabs> with SingleTickerProviderStateMixin {
  late TabController _controller;

  @override
  void initState() {
    super.initState();
    _controller = TabController(length: 3, vsync: this);
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
}
```
</details>

---

### Q26. Spot the bug causing a memory leak when using `TabController`.

```dart
class CustomTabState extends State<CustomTab> with SingleTickerProviderStateMixin {
  late TabController _controller;

  @override
  void initState() {
    super.initState();
    _controller = TabController(length: 2, vsync: this);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(...);
  }
  // Missing dispose!
}
```

<details>
<summary>👉 View Answer & Explanation</summary>

**The Bug:**
`TabController` holds internal animation tickers and listeners registered with the engine. Omitting `_controller.dispose()` leaks animation tickers and memory when the widget is permanently unmounted from the tree.

**The Fix:**
Always override `dispose()`:
```dart
@override
void dispose() {
  _controller.dispose();
  super.dispose();
}
```
</details>

---

## Category 4: Scenario-Based Architecture & Design

### Q27. Design Scenario: Multi-Step Checkout Wizard
You are tasked with building an eCommerce checkout flow:
1. `CartScreen`
2. `ShippingAddressScreen`
3. `PaymentMethodScreen`
4. `OrderReviewScreen`
5. `OrderConfirmationScreen`

**Requirements:**
- The user can press the back button while in `ShippingAddressScreen`, `PaymentMethodScreen`, or `OrderReviewScreen` to edit previous choices.
- When the user confirms the purchase on `OrderReviewScreen`, `OrderConfirmationScreen` is presented.
- From `OrderConfirmationScreen`, pressing the back button or a "Return to Store" button must return the user directly to `CartScreen` or `HomeScreen`, without allowing them to re-enter the payment or address screens.

**Question:** Write the navigation call to execute upon successful purchase on `OrderReviewScreen`.

<details>
<summary>👉 View Answer & Explanation</summary>

**Recommended Architecture:**
Use `Navigator.pushAndRemoveUntil` targeting the root store or home screen:

```dart
void onOrderPlacedSuccessfully(BuildContext context) {
  Navigator.pushAndRemoveUntil(
    context,
    MaterialPageRoute(builder: (_) => const OrderConfirmationScreen()),
    ModalRoute.withName('/home'), // Keeps /home intact, purges all checkout routes!
  );
}
```
This guarantees that when the user presses back from `OrderConfirmationScreen`, they land cleanly on `/home`, and the completed checkout flow is safely wiped from memory.
</details>

---

### Q28. Design Scenario: Persistent Multi-Tab Navigation
In a social media application (like Instagram), tapping between the "Feed", "Search", and "Profile" tabs must maintain the independent navigation stack of each tab. For example, if the user drills into `Profile -> Followers -> FollowerDetail` on Tab 3, switches to Tab 1 (Feed), and then switches back to Tab 3, they should still see `FollowerDetail`. Furthermore, pressing the Android back button should pop `FollowerDetail` first before exiting the app.

**Question:** How is this architectural requirement implemented in pure Flutter without third-party packages?

<details>
<summary>👉 View Answer & Explanation</summary>

**Architecture Explanation:**
1. Maintain a master `Scaffold` containing a `BottomNavigationBar` and an `IndexedStack` in its `body`.
2. Inside each slot of the `IndexedStack`, embed an independent **Nested `Navigator`** widget, each with its own `GlobalKey<NavigatorState>()`.
3. Wrap the master `Scaffold` in a `PopScope`. In `onPopInvokedWithResult`, check if the active tab's nested navigator `canPop()`. If so, call `nestedNavigatorKey.currentState!.maybePop()`; only if the nested navigator cannot pop do you allow the system back button to exit or switch to the primary tab.
</details>

---

### Q29. Design Scenario: Deep Link Authentication Interception
A user clicks a promotional link in an email: `https://store.com/orders/9482`.
When the app launches, the user's local authentication token has expired.

**Question:** How should the app architecture structure this flow so the user is directed to Login, but after logging in, is automatically forwarded to `orders/9482` instead of the generic homepage?

<details>
<summary>👉 View Answer & Explanation</summary>

**Architectural Solution:**
1. When parsing the incoming deep link route (`/orders/9482`), check the authentication state in `onGenerateRoute` or your router delegate.
2. If unauthenticated, redirect to `AppRoutes.login`, passing the intended destination route as an argument payload:
   ```dart
   class LoginArguments {
     final String? redirectRoute;
     const LoginArguments({this.redirectRoute});
   }
   ```
3. Inside `LoginScreen`, once the credentials succeed:
   ```dart
   if (args?.redirectRoute != null) {
     Navigator.pushReplacementNamed(context, args!.redirectRoute!);
   } else {
     Navigator.pushReplacementNamed(context, AppRoutes.home);
   }
   ```
</details>

---

### Q30. Architectural Evaluation: Navigator 1.0 vs GoRouter
Your engineering team is starting a project that must run concurrently on iOS, Android, and Flutter Web. The web version requires bookmarkable URLs, browser back/forward button support, and query parameters (e.g. `/search?query=flutter&filter=recent`).

**Question:** Why is standard Navigator 1.0 (`Navigator.pushNamed`) insufficient for this requirement, and what makes `GoRouter` or Declarative Navigator 2.0 necessary?

<details>
<summary>👉 View Answer & Explanation</summary>

**Evaluation:**
- **Navigator 1.0 Limitations on Web:**
  - Navigator 1.0 is an imperative stack. It tells Flutter *how* to transition (`push`, `pop`), but it does not inherently synchronize with the browser's address bar.
  - When a user changes the URL in the browser bar manually or pastes a link, Navigator 1.0 has no built-in route parsing architecture to reconstruct the parent route stack beneath the destination screen.
  - Query parameters and browser history (Forward / Back buttons) are cumbersome to synchronize imperatively.
- **Why GoRouter / Router 2.0 Solves This:**
  - Operates declaratively based on URL state as the single source of truth.
  - Maps URL paths and query parameters directly to widget trees.
  - Seamlessly keeps the browser address bar, browser history, and Flutter app state in lockstep.
</details>

---

### Q31. Design Scenario: Dynamic Bottom Sheet Selection Returning Data
You have a `UserProfileScreen`. The user taps an "Edit Avatar" button, which opens a modal bottom sheet displaying a grid of 6 avatar icons.

**Question:** Write the code to open the modal bottom sheet, capture the user's chosen avatar string, and update the screen state.

<details>
<summary>👉 View Answer & Explanation</summary>

```dart
Future<void> _changeAvatar(BuildContext context) async {
  final String? selectedAvatar = await showModalBottomSheet<String>(
    context: context,
    builder: (BuildContext sheetContext) {
      return Wrap(
        children: [
          ListTile(
            leading: const Icon(Icons.face),
            title: const Text('Robot Avatar'),
            onTap: () => Navigator.pop(sheetContext, 'avatar_robot.png'),
          ),
          ListTile(
            leading: const Icon(Icons.person),
            title: const Text('Ninja Avatar'),
            onTap: () => Navigator.pop(sheetContext, 'avatar_ninja.png'),
          ),
        ],
      );
    },
  );

  if (!context.mounted) return;

  if (selectedAvatar != null) {
    setState(() {
      _currentAvatar = selectedAvatar;
    });
  }
}
```
</details>

---

### Q32. Design Scenario: Fallback Route Handling
Your app allows merchants to push dynamic in-app notification banners pointing to route names received from a remote Firebase server. If a merchant sends a notification with route name `'/black_friday_deals'`, but the user has not updated their app to the version that includes that screen, what architecture prevents an unhandled exception crash?

<details>
<summary>👉 View Answer & Explanation</summary>

**Architectural Solution:**
Implement `MaterialApp.onUnknownRoute`:

```dart
MaterialApp(
  onGenerateRoute: AppRouter.generateRoute,
  onUnknownRoute: (RouteSettings settings) {
    return MaterialPageRoute(
      settings: settings,
      builder: (_) => Scaffold(
        appBar: AppBar(title: const Text('Page Unavailable')),
        body: Center(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              const Icon(Icons.warning_amber_rounded, size: 64, color: Colors.orange),
              const SizedBox(height: 16),
              Text(
                'The page "${settings.name}" is not available.\nPlease update your app to the latest version.',
                textAlign: TextAlign.center,
              ),
              const SizedBox(height: 24),
              ElevatedButton(
                onPressed: () => Navigator.of(_).pop(),
                child: const Text('Go Back'),
              ),
            ],
          ),
        ),
      ),
    );
  },
)
```
This guarantees that any unknown route name sent by a server payload is caught gracefully, presenting an actionable notice to the user rather than crashing the Flutter engine.
</details>
