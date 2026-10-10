# Named Routes in Flutter

## 📖 Introduction
In large-scale applications with dozens or hundreds of screens, writing `Navigator.push(context, MaterialPageRoute(builder: (context) => SomeScreen()))` across every button callback scatters navigation logic and creates tight coupling between widgets. If a screen's constructor arguments change or a screen needs to be refactored, you must hunt through dozens of UI files to update every manual `MaterialPageRoute`.

**Named Routes** provide a centralized, web-like routing architecture for Flutter applications. Instead of instantiating widgets inline, you define a central dictionary table of string paths (such as `'/'`, `'/profile'`, `'/settings'`) in your [MaterialApp](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/05_Navigation/08_Named_Routes.md). Navigating to any screen becomes as clean as calling [Navigator.pushNamed(context, '/profile')](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/05_Navigation/08_Named_Routes.md).

---

## 🤔 Why Do We Need It?
1. **Centralized Route Registry**: All screens, path identifiers, and route builders are organized in a single dedicated configuration table.
2. **Decoupled Architecture**: Screen A does not need to import or know about Screen B's class name; it simply requests navigation to a route identifier string.
3. **Web URL Alignment**: On Flutter Web, named routes map directly to the browser's address bar (e.g. `myapp.com/#/settings`).
4. **Graceful 404 Fallbacks**: The `onUnknownRoute` callback catches any invalid or broken route strings and gracefully renders a custom "Page Not Found" screen instead of crashing.

---

## 🌍 Real-Life Analogy
Think of Named Routes as a **Corporate Telephone Switchboard or Extension Directory**:

```
+-------------------------------------------------------------+
|                     OFFICE PHONE SYSTEM                     |
|                                                             |
|   [ EXTENSION DIRECTORY (routes table) ]:                   |
|   • Ext 100 ('/')          -> Reception Desk (HomeScreen)   |
|   • Ext 201 ('/sales')     -> Sales Dept     (SalesScreen)  |
|   • Ext 305 ('/support')   -> IT Support     (SupportScreen)|
|                                                             |
|   Instead of running across the building and personally     |
|   knocking on the door of the IT technician (manual push),   |
|   you pick up the phone and dial "305" (pushNamed).         |
|                                                             |
|   The switchboard connects you instantly. If you dial an    |
|   invalid number (Ext 999), the operator says:              |
|   "That extension does not exist" (onUnknownRoute).         |
+-------------------------------------------------------------+
```

---

## 🧠 Internal Working
When you call `Navigator.pushNamed(context, '/settings')`:

```
                Navigator.pushNamed(context, '/settings')
                                   │
                                   ▼
             Navigator queries MaterialApp.routes table
                                   │
                    ┌──────────────┴──────────────┐
                    ▼                             ▼
        Found in routes map?             NOT found in routes map?
                    │                             │
                    ▼                             ▼
        Executes builder function       Checks onGenerateRoute
                    │                             │
                    ▼                             ▼
          Wraps in Route and            If still null, executes
          pushes onto stack               onUnknownRoute (404)
```

1. **Dictionary Lookup**: The `Navigator` inspects the `Map<String, WidgetBuilder> routes` provided to `MaterialApp`.
2. **Builder Execution**: If an entry exists for the requested path, Flutter executes its `WidgetBuilder` callback `(BuildContext) => Widget`.
3. **Fallback Chain**: If the route name is not present in `routes`, Flutter checks `onGenerateRoute`. If still unhandled, it falls back to `onUnknownRoute`.

---

## ⚙ Syntax

### 1. Declaring Routes in MaterialApp
```dart
MaterialApp(
  initialRoute: '/',
  routes: {
    '/': (context) => const HomeScreen(),
    '/profile': (context) => const ProfileScreen(),
    '/settings': (context) => const SettingsScreen(),
  },
  onUnknownRoute: (settings) {
    return MaterialPageRoute(builder: (_) => const NotFoundScreen());
  },
)
```

### 2. Navigating to a Named Route
```dart
Navigator.pushNamed(context, '/profile');
```

### 3. Replacing with a Named Route
```dart
Navigator.pushReplacementNamed(context, '/dashboard');
```

---

## 📋 Important Properties

| Property | Type | Description | Default |
| :--- | :--- | :--- | :--- |
| `initialRoute` | `String?` | The name of the first route to display when the app launches. | `'/'` |
| `routes` | `Map<String, WidgetBuilder>` | The application's top-level routing table mapping path strings to widget builders. | `{}` |
| `onUnknownRoute` | `RouteFactory?` | Fallback route builder triggered when a requested route name cannot be resolved. | `null` |
| `Navigator.pushNamed` | `Future<T?> Function(...)` | Pushes a named route onto the stack. | N/A |
| `Navigator.pushReplacementNamed`| `Future<T?> Function(...)`| Replaces the current top route with a named route. | N/A |

---

## 💻 Complete Runnable Example
The following production-ready application demonstrates:
1. A strongly typed route constants class (`AppRoutes`).
2. A centralized `routes` table in `MaterialApp`.
3. An `onUnknownRoute` 404 handler for catching unhandled routes.
4. Clean navigation between Home, Profile, and Settings screens.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const NamedRoutesMasterclassApp());
}

// 1. Centralized Route Name Constants (Eliminates string typo bugs!)
class AppRoutes {
  static const String home = '/';
  static const String profile = '/profile';
  static const String settings = '/settings';
  static const String invalidRoute = '/this_does_not_exist';
}

class NamedRoutesMasterclassApp extends StatelessWidget {
  const NamedRoutesMasterclassApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Named Routes Masterclass',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.teal),
        useMaterial3: true,
      ),

      // 2. Initial Route
      initialRoute: AppRoutes.home,

      // 3. Centralized Routing Table
      routes: {
        AppRoutes.home: (context) => const HomeScreen(),
        AppRoutes.profile: (context) => const ProfileScreen(),
        AppRoutes.settings: (context) => const SettingsScreen(),
      },

      // 4. Graceful 404 Fallback for unmapped routes
      onUnknownRoute: (RouteSettings settings) {
        return MaterialPageRoute(
          builder: (context) => NotFoundScreen(attemptedRouteName: settings.name ?? 'Unknown'),
        );
      },
    );
  }
}

// ---------------------------------------------------------------------------
// SCREEN 1: Home (Route: '/')
// ---------------------------------------------------------------------------
class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Named Routes Home'),
        centerTitle: true,
      ),
      body: SafeArea(
        child: Padding(
          padding: const EdgeInsets.all(24.0),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: [
              Card(
                color: Colors.teal.shade50,
                elevation: 0,
                shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(16)),
                child: const Padding(
                  padding: EdgeInsets.all(20.0),
                  child: Column(
                    children: [
                      Icon(Icons.alt_route, size: 54, color: Colors.teal),
                      SizedBox(height: 12),
                      Text(
                        'Centralized Routing Table',
                        style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
                      ),
                      SizedBox(height: 6),
                      Text(
                        "Navigating via Navigator.pushNamed(context, '/path')",
                        textAlign: TextAlign.center,
                        style: TextStyle(color: Colors.grey),
                      ),
                    ],
                  ),
                ),
              ),
              const SizedBox(height: 32),

              FilledButton.icon(
                onPressed: () {
                  // Navigate to Profile via named route
                  Navigator.pushNamed(context, AppRoutes.profile);
                },
                icon: const Icon(Icons.person),
                label: const Text("Go to Profile (pushNamed('/profile'))"),
                style: FilledButton.styleFrom(padding: const EdgeInsets.symmetric(vertical: 16)),
              ),
              const SizedBox(height: 16),

              FilledButton.tonalIcon(
                onPressed: () {
                  // Navigate to Settings via named route
                  Navigator.pushNamed(context, AppRoutes.settings);
                },
                icon: const Icon(Icons.settings),
                label: const Text("Go to Settings (pushNamed('/settings'))"),
                style: FilledButton.styleFrom(padding: const EdgeInsets.symmetric(vertical: 16)),
              ),
              const SizedBox(height: 16),

              OutlinedButton.icon(
                onPressed: () {
                  // Attempt navigating to an unregistered route to test onUnknownRoute
                  Navigator.pushNamed(context, AppRoutes.invalidRoute);
                },
                icon: const Icon(Icons.bug_report),
                label: const Text('Test 404 Route (onUnknownRoute)'),
                style: OutlinedButton.styleFrom(
                  foregroundColor: Colors.redAccent,
                  padding: const EdgeInsets.symmetric(vertical: 16),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

// ---------------------------------------------------------------------------
// SCREEN 2: Profile (Route: '/profile')
// ---------------------------------------------------------------------------
class ProfileScreen extends StatelessWidget {
  const ProfileScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('User Profile'),
      ),
      body: SafeArea(
        child: Padding(
          padding: const EdgeInsets.all(24.0),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: [
              const CircleAvatar(
                radius: 48,
                child: Icon(Icons.person, size: 48),
              ),
              const SizedBox(height: 16),
              const Text(
                'Alex Morgan',
                textAlign: TextAlign.center,
                style: TextStyle(fontSize: 22, fontWeight: FontWeight.bold),
              ),
              const Text(
                'alex@flutter.dev',
                textAlign: TextAlign.center,
                style: TextStyle(color: Colors.grey),
              ),
              const Spacer(),

              FilledButton(
                onPressed: () => Navigator.pop(context),
                style: FilledButton.styleFrom(padding: const EdgeInsets.symmetric(vertical: 16)),
                child: const Text('Back to Home'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

// ---------------------------------------------------------------------------
// SCREEN 3: Settings (Route: '/settings')
// ---------------------------------------------------------------------------
class SettingsScreen extends StatelessWidget {
  const SettingsScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('App Settings'),
      ),
      body: SafeArea(
        child: ListView(
          padding: const EdgeInsets.all(16.0),
          children: [
            ListTile(
              leading: const Icon(Icons.notifications),
              title: const Text('Push Notifications'),
              trailing: Switch(value: true, onChanged: (_) {}),
            ),
            ListTile(
              leading: const Icon(Icons.security),
              title: const Text('Two-Factor Authentication'),
              trailing: Switch(value: false, onChanged: (_) {}),
            ),
          ],
        ),
      ),
    );
  }
}

// ---------------------------------------------------------------------------
// SCREEN 4: 404 Not Found (Triggered via onUnknownRoute)
// ---------------------------------------------------------------------------
class NotFoundScreen extends StatelessWidget {
  final String attemptedRouteName;

  const NotFoundScreen({super.key, required this.attemptedRouteName});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Page Not Found (404)'),
        backgroundColor: Colors.red.shade100,
      ),
      body: SafeArea(
        child: Padding(
          padding: const EdgeInsets.all(24.0),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: [
              const Icon(Icons.error_outline, size: 84, color: Colors.redAccent),
              const SizedBox(height: 20),
              const Text(
                '404 - Route Not Found',
                textAlign: TextAlign.center,
                style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
              ),
              const SizedBox(height: 8),
              Text(
                'The requested route "$attemptedRouteName" does not exist in the routing table.',
                textAlign: TextAlign.center,
                style: const TextStyle(color: Colors.grey),
              ),
              const SizedBox(height: 32),

              FilledButton.icon(
                onPressed: () => Navigator.pop(context),
                icon: const Icon(Icons.arrow_back),
                label: const Text('Go Back'),
                style: FilledButton.styleFrom(
                  backgroundColor: Colors.redAccent,
                  padding: const EdgeInsets.symmetric(vertical: 16),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

---

## 🔍 Line-by-Line Example Explanation
- `class AppRoutes { static const String home = '/'; ... }`:
  - **What it does**: Declares compile-time constants for every route path string.
  - **Why it exists**: Hardcoding strings like `Navigator.pushNamed(context, '/profille')` introduces silent typo bugs; constants provide compile-time autocompletion and error checking.
- `initialRoute: AppRoutes.home`:
  - **What it does**: Informs `MaterialApp` which route in the `routes` table to render when the app boots.
  - **Fatal collision rule**: If you provide `initialRoute: '/'`, you must **NOT** also provide `home: const HomeScreen()`; doing so causes an assertion crash because both define the root screen!
- `onUnknownRoute: (RouteSettings settings) { ... }`:
  - **What it does**: Acts as a safety net when `pushNamed` requests a path that does not exist in the `routes` table.
  - **Why it exists**: Prevents the application from crashing, rendering a friendly 404 screen instead.

---

## 📱 ASCII Phone Screen Preview

```
[ Screen 1: Home ('/') ]       [ Screen 2: Profile ('/profile') ]  [ Screen 4: 404 Not Found ]
+----------------------+      +---------------------------------+  +--------------------------+
|  Named Routes Home   |      | [←]  User Profile               |  | [←] Page Not Found (404) |
+----------------------+      +---------------------------------+  +--------------------------+
|                      |      |                                 |  |                          |
| [ ☲ Central Table ]  | PUSH |             [ 👤 ]              |  |      [ ⚠ 404 ERROR ]     |
|                      |----->|          Alex Morgan            |  |                          |
| [ Go to Profile ]    |      |        alex@flutter.dev         |  | Route Not Found:         |
| [ Go to Settings ]   |      |                                 |  | "/this_does_not_exist"   |
| [ Test 404 Route ]   |----->| [ Back to Home ]                |  |                          |
+----------------------+ (404)+---------------------------------+  | [ Go Back ]              |
                                                                   +--------------------------+
```

---

## 🌳 Widget Tree
```
MaterialApp [initialRoute: '/', routes: {...}, onUnknownRoute: ...]
 └── Navigator [State: NavigatorState, _history: [HomeScreen]]
      └── Overlay
           ├── OverlayEntry (HomeScreen) [Route: '/']
           └── OverlayEntry (ProfileScreen) [Pushed via: pushNamed('/profile')]
```

---

## ⚙ Flutter Execution Flow
1. **Boot**: `MaterialApp` reads `initialRoute: '/'`, looks up `'/'` in `routes`, and launches `HomeScreen`.
2. **Push Named**: User taps "Go to Profile". `Navigator.pushNamed(context, '/profile')` runs.
3. **Table Match**: `NavigatorState` finds `/profile` in `routes` and calls its builder `(context) => const ProfileScreen()`.
4. **Transition**: A `MaterialPageRoute` is automatically synthesized and pushed onto the stack.
5. **Unknown Route Trigger**: When the user taps "Test 404 Route", Flutter fails to find `/this_does_not_exist` in `routes`. It invokes `onUnknownRoute`, successfully rendering `NotFoundScreen`.

---

## ⚠ Common Beginner Mistakes

### 1. Providing Both `home` and `initialRoute: '/'`
```dart
// ❌ CRASH: Defining the root route twice!
MaterialApp(
  home: const HomeScreen(), // Defines '/'
  initialRoute: '/',         // Also defines '/'!
  routes: {
    '/': (context) => const HomeScreen(),
  },
)
// Result: Crash! Cannot provide both a home and an initialRoute of '/'

// ✅ GOOD: Use either home OR initialRoute, not both!
MaterialApp(
  initialRoute: '/',
  routes: {
    '/': (context) => const HomeScreen(),
  },
)
```

### 2. Hardcoding Magic Strings Everywhere
```dart
// ❌ PRONE TO TYPOS:
Navigator.pushNamed(context, '/proflie'); // Typo! Crash or 404!

// ✅ GOOD: Use static const route constants
Navigator.pushNamed(context, AppRoutes.profile);
```

### 3. Trying to Pass Path Parameters Like `/users/:id` in `routes`
```dart
// ❌ UNSUPPORTED in basic routes map:
routes: {
  '/users/:id': (context) => UserScreen(), // Flutter does NOT parse dynamic URL segments in routes map!
}
// For dynamic path parsing, use onGenerateRoute or GoRouter!
```

---

## 🚀 Professional Usage
- **Web Applications**: Generating human-readable browser history URLs (`/dashboard`, `/analytics`, `/settings`).
- **Deep Linking Preparation**: Matching incoming push notification payloads (e.g. `{"route": "/profile"}`) directly to app screens.
- **Enterprise Design Systems**: Decoupling large feature teams so Team A can navigate to Team B's feature using only its route identifier.

---

## 💡 Best Practices
1. **Always wrap route strings in a static class**: e.g. `AppRoutes.home`, `AppRoutes.checkout`.
2. **Always implement `onUnknownRoute`**: Protects users from crashes if invalid deep links or deprecated paths are requested.
3. **Do not use `routes` for dynamic path parameters**: If you need `/products/123`, use `onGenerateRoute` (covered in Chapter 09) instead of basic `routes`.

---

## 🧪 Try It Yourself
1. Add an `AppRoutes.about` route to the dictionary table and verify that navigating to it works.
2. Remove `onUnknownRoute` and observe what happens when you trigger an unregistered route name.
3. Test using `Navigator.pushReplacementNamed` instead of `pushNamed`.

---

## 🎯 Challenge
Build an "App Store Deep Link Simulator":
- Text field where the user can type any arbitrary route string (e.g. `'/profile'`, `'/settings'`, `'/cart'`).
- A button: *"Simulate Deep Link"*.
- When tapped, use `Navigator.pushNamed` to navigate to whatever string was entered.
- Handle both valid routes and invalid 404 routes gracefully!

---

## ❓ Practice Questions

### Q1. What data structure is used for the `routes` property in `MaterialApp`?
- A) `List<Route>`
- B) `Map<String, WidgetBuilder>`
- C) `Set<String>`
- D) `Queue<Widget>`
*Answer: B* — It maps string route paths to widget builder functions `Widget Function(BuildContext)`.

### Q2. What assertion failure occurs if you provide both `home: const MyHome()` and `initialRoute: '/'`?
- A) A memory leak warning
- B) An assertion failure stating that both `home` and `initialRoute: '/'` cannot be provided simultaneously
- C) The app runs in black and white
- D) The back button is disabled
*Answer: B* — Both properties attempt to define the root route, causing a collision.

### Q3. What is the role of `onUnknownRoute` in `MaterialApp`?
- A) It deletes broken routes
- B) It acts as a fallback route factory that handles unregistered or invalid route names (like a 404 page)
- C) It connects to the internet to download missing screens
- D) It locks the application
*Answer: B* — Generates a fallback route when the requested route identifier is missing.

### Q4. Why is using a dedicated `AppRoutes` class with `static const String` constants recommended?
- A) It saves battery
- B) It prevents silent runtime typos in route strings and provides IDE autocomplete
- C) It is required by the Dart VM
- D) It encrypts the route names
*Answer: B* — Constants catch typos at compile time instead of failing at runtime.

### Q5. Can the standard `routes` table parse dynamic path segments like `/products/:id`?
- A) Yes, Flutter natively parses URL parameters in `routes`
- B) No, standard `routes` only does exact string matching; use `onGenerateRoute` or `GoRouter` for path parameters
- C) Only on Android
- D) Only in release mode
*Answer: B* — The `routes` table requires exact string matches.

### Q6. What method replaces the current screen with a named route?
*Answer*: `Navigator.pushReplacementNamed(context, '/routeName')`.

### Q7. Output Prediction: What will be displayed if the user navigates to an unregistered route and `onUnknownRoute` is defined?
*Answer*: The widget returned by the `onUnknownRoute` callback (e.g. `NotFoundScreen`).

### Q8. How do you push a named route and delete all previous screens?
*Answer*:
```dart
Navigator.pushNamedAndRemoveUntil(context, '/home', (route) => false);
```

### Q9. What is the default value of `initialRoute` in `MaterialApp`?
*Answer*: `'/'`.

### Q10. Write a minimal code snippet defining a `routes` table with a home screen and a profile screen.
*Answer*:
```dart
routes: {
  '/': (context) => const HomeScreen(),
  '/profile': (context) => const ProfileScreen(),
}
```

---

## 📝 Summary
- **Named Routes** decouple screen invocation from widget implementations using a central dictionary table.
- Declare paths in `MaterialApp(routes: { ... })` and navigate via `Navigator.pushNamed(context, '/path')`.
- Prevent string typos by storing route paths as `static const String` constants in an `AppRoutes` class.
- Always provide an `onUnknownRoute` callback to handle broken or invalid routes gracefully.
- For dynamic path parameters (e.g. `/user/42`), transition to `onGenerateRoute`.
