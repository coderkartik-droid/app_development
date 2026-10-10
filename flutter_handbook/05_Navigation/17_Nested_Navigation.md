# Nested Navigation in Flutter

## 📖 Introduction
In popular mobile apps like Instagram, YouTube, and Spotify, you can be several screens deep into a profile or playlist, switch to the Search tab, search for a song, and when you switch back to the Profile tab, your deep navigation history is still intact! Furthermore, when you navigate deeper into a tab, **the bottom navigation bar remains permanently visible at the bottom of the screen**.

If you use only Flutter's default root navigator, calling `Navigator.push()` pushes the new screen over the *entire display*, completely covering the bottom navigation bar. 

**Nested Navigation** is the architectural pattern of embedding independent, secondary [Navigator](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/05_Navigation/02_Navigator.md) widgets inside child subtrees (such as inside individual tabs of a bottom navigation bar or within split-view tablet panes). Each nested navigator maintains its own private navigation stack, its own `_history` list, and its own back-navigation sequence.

---

## 🤔 Why Do We Need It?
1. **Persistent Bottom Bars**: Keeps the primary `NavigationBar` permanently visible on screen while sub-pages drill down within the body area.
2. **Independent Tab Histories**: Tab A can be 4 screens deep (`Feed -> Post -> Comments -> UserProfile`), while Tab B remains on its root search page.
3. **Modal Isolation**: Dialogs or full-screen video players can be pushed to the root navigator (`rootNavigator: true`), while standard page transitions stay contained within the tab.
4. **Desktop & Tablet Split Views**: Allows the left pane (list) and right pane (detail) to push screens independently without disturbing one another.

---

## 🌍 Real-Life Analogy
Think of Nested Navigation as **Browsing with Multiple Tabs in a Desktop Web Browser**:

```
+-------------------------------------------------------------+
|                      WEB BROWSER WINDOW                     |
|                                                             |
|   Tab 1: Wikipedia [Back] [Forward]                         |
|   History: Physics -> Einstein -> Relativity                |
|                                                             |
|   Tab 2: YouTube   [Back] [Forward]                         |
|   History: Home -> Flutter Tutorial                         |
|                                                             |
|   When you click "Back" on Tab 1, it goes from Relativity   |
|   back to Einstein. It doesn't touch Tab 2 at all!          |
|   Each browser tab owns its private navigation history!     |
+-------------------------------------------------------------+
```

---

## 🧠 Internal Working
To master nested navigation, inspect the multi-tier Navigator tree:

```
                            MaterialApp
                                 │
                                 ▼
                    Root Navigator (Level 1)
                     _history: [ MainShellScreen ]
                                 │
                                 ▼
                          MainShellScreen
                       (Holds NavigationBar)
                                 │
                                 ▼
                     IndexedStack (Preserves Tabs)
                                 │
        ┌────────────────────────┴────────────────────────┐
        ▼                                                 ▼
 Tab 1 Navigator (Level 2)                      Tab 2 Navigator (Level 2)
 key: _tab1NavKey                              key: _tab2NavKey
 _history: [ Feed, Post, Author ]              _history: [ SearchHome ]
```

1. **Context Resolution**: When a widget calls `Navigator.of(context)`, it traverses *up* the tree. It will find the **nearest nested navigator** first!
2. **Escaping to the Root**: If a nested screen needs to push a full-screen view that covers the bottom bar, it must write:
   ```dart
   Navigator.of(context, rootNavigator: true).push(...);
   ```
3. **Android System Back Interception**: By default, the Android system back button communicates only with the root navigator. To prevent the back button from closing the app when a sub-tab is deep in history, you must intercept the back gesture using `PopScope` and delegate it to the active tab's nested navigator!

---

## ⚙ Syntax

### Declaring a Nested Navigator inside a Tab
```dart
Navigator(
  key: _tab1NavigatorKey, // GlobalKey<NavigatorState>
  onGenerateRoute: (RouteSettings settings) {
    return MaterialPageRoute(
      builder: (context) => const Tab1RootScreen(),
    );
  },
)
```

### Popping the Active Sub-Navigator via PopScope
```dart
PopScope(
  canPop: false,
  onPopInvokedWithResult: (didPop, result) {
    if (didPop) return;
    // Check if the current tab's nested navigator can pop
    final isSubNavPopped = _currentTabNavigatorKey.currentState?.maybePop() ?? false;
    if (!isSubNavPopped) {
      // Exit app or switch to first tab
    }
  },
  child: Scaffold(...),
)
```

---

## 📋 Comparison Table

| Feature | Single Root Navigator | Nested Tab Navigators |
| :--- | :--- | :--- |
| **Bottom Bar Visibility** | Covered upon `Navigator.push()` | Permanently visible during drill-downs |
| **Tab History Isolation** | ❌ Switching tabs resets or shares stack | ✅ Every tab preserves its own independent stack |
| **Back Button Logic** | Simple (automatic) | Requires `PopScope` to route back events to active tab |
| **Architecture Complexity**| Low | Intermediate / Advanced |
| **Ideal For** | Simple linear apps, forms | Instagram, YouTube, Spotify-style apps |

---

## 💻 Complete Runnable Example
The following production-ready application demonstrates:
1. Two primary tabs (Feed and Search) managed via `NavigationBar`.
2. Each tab owns its own independent nested [Navigator](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/05_Navigation/02_Navigator.md).
3. The bottom bar stays permanently visible as you drill down into sub-screens.
4. Handling the device back button with `PopScope` so pressing back unwinds the active tab's stack first before exiting!

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const NestedNavigationMasterclassApp());
}

class NestedNavigationMasterclassApp extends StatelessWidget {
  const NestedNavigationMasterclassApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Nested Navigation Masterclass',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const MainTabShell(),
    );
  }
}

class MainTabShell extends StatefulWidget {
  const MainTabShell({super.key});

  @override
  State<MainTabShell> createState() => _MainTabShellState();
}

class _MainTabShellState extends State<MainTabShell> {
  int _selectedTabIndex = 0;

  // 1. Separate GlobalKeys for each nested navigator
  final GlobalKey<NavigatorState> _feedNavKey = GlobalKey<NavigatorState>();
  final GlobalKey<NavigatorState> _searchNavKey = GlobalKey<NavigatorState>();

  GlobalKey<NavigatorState> get _currentNavKey =>
      _selectedTabIndex == 0 ? _feedNavKey : _searchNavKey;

  @override
  Widget build(BuildContext context) {
    // 2. Intercept back button to pop nested navigator before exiting!
    return PopScope(
      canPop: false,
      onPopInvokedWithResult: (didPop, result) async {
        if (didPop) return;

        final NavigatorState? currentNavigator = _currentNavKey.currentState;

        // If the active tab has sub-screens on its stack, pop it!
        if (currentNavigator != null && currentNavigator.canPop()) {
          currentNavigator.pop();
        } else if (_selectedTabIndex != 0) {
          // If at root of secondary tab, switch to home tab first!
          setState(() {
            _selectedTabIndex = 0;
          });
        } else {
          // At root of home tab: allow system exit
          Navigator.of(context).pop();
        }
      },
      child: Scaffold(
        // IndexedStack preserves both nested navigators in memory simultaneously!
        body: IndexedStack(
          index: _selectedTabIndex,
          children: [
            // TAB 1: Feed with its own Nested Navigator
            Navigator(
              key: _feedNavKey,
              onGenerateRoute: (settings) => MaterialPageRoute(
                settings: settings,
                builder: (context) => const FeedRootScreen(),
              ),
            ),

            // TAB 2: Search with its own Nested Navigator
            Navigator(
              key: _searchNavKey,
              onGenerateRoute: (settings) => MaterialPageRoute(
                settings: settings,
                builder: (context) => const SearchRootScreen(),
              ),
            ),
          ],
        ),

        // Persistent Bottom Navigation Bar
        bottomNavigationBar: NavigationBar(
          selectedIndex: _selectedTabIndex,
          onDestinationSelected: (index) {
            if (index == _selectedTabIndex) {
              // Tapping active tab pops it back to root! (Standard iOS/Android UX)
              _currentNavKey.currentState?.popUntil((r) => r.isFirst);
            } else {
              setState(() => _selectedTabIndex = index);
            }
          },
          destinations: const [
            NavigationDestination(icon: Icon(Icons.feed_outlined), label: 'Feed'),
            NavigationDestination(icon: Icon(Icons.search), label: 'Search'),
          ],
        ),
      ),
    );
  }
}

// ---------------------------------------------------------------------------
// TAB 1 SCREENS: Feed -> Post -> Author
// ---------------------------------------------------------------------------
class FeedRootScreen extends StatelessWidget {
  const FeedRootScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Feed')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(Icons.article_outlined, size: 72, color: Colors.deepPurple),
            const SizedBox(height: 16),
            const Text('Feed Stream (Tab 1 Root)', style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
            const SizedBox(height: 24),
            FilledButton.icon(
              onPressed: () {
                // Pushes inside Tab 1's nested navigator!
                Navigator.push(
                  context,
                  MaterialPageRoute(builder: (ctx) => const PostDetailScreen()),
                );
              },
              icon: const Icon(Icons.arrow_forward),
              label: const Text('Open Post (Drill Down)'),
            ),
          ],
        ),
      ),
    );
  }
}

class PostDetailScreen extends StatelessWidget {
  const PostDetailScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Post Discussion')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Text('Post #142: Flutter Architecture', style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            const Text('Notice the bottom navigation bar is STILL VISIBLE!'),
            const SizedBox(height: 24),
            FilledButton(
              onPressed: () {
                // Drill even deeper inside Tab 1!
                Navigator.push(
                  context,
                  MaterialPageRoute(builder: (ctx) => const AuthorProfileScreen()),
                );
              },
              child: const Text('Drill into Author Profile (3rd layer)'),
            ),
          ],
        ),
      ),
    );
  }
}

class AuthorProfileScreen extends StatelessWidget {
  const AuthorProfileScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Author: Sarah Connor')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const CircleAvatar(radius: 40, child: Icon(Icons.person, size: 40)),
            const SizedBox(height: 16),
            const Text('Sarah Connor (Lead Architect)', style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            const Text('Layer 3 inside Tab 1.\nNow tap the "Search" tab below to verify independent history!'),
          ],
        ),
      ),
    );
  }
}

// ---------------------------------------------------------------------------
// TAB 2 SCREENS: Search Root
// ---------------------------------------------------------------------------
class SearchRootScreen extends StatelessWidget {
  const SearchRootScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Search & Explore')),
      body: const Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(Icons.search, size: 72, color: Colors.indigo),
            SizedBox(height: 16),
            Text('Search Tab Root (Independent Stack)', style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
            SizedBox(height: 8),
            Text('Switch back to Feed—it is still on Sarah Connor!'),
          ],
        ),
      ),
    );
  }
}
```

---

## 🔍 Line-by-Line Example Explanation
- `Navigator(key: _feedNavKey, onGenerateRoute: ...)`:
  - **What it does**: Declares a full nested `Navigator` widget inside Tab 0.
  - **Why it matters**: Any `Navigator.push(context)` called from inside `FeedRootScreen` targets this nested navigator, keeping navigation confined inside the body above the bottom bar.
- `PopScope(canPop: false, onPopInvokedWithResult: ...)`:
  - **What it does**: Intercepts system back-button presses and redirects them to the active tab's nested navigator.
  - **Fatal bug if omitted**: Without this, pressing the Android back button when sitting on `AuthorProfileScreen` would exit the entire app instead of backing out to `PostDetailScreen`!
- `_currentNavKey.currentState?.popUntil((r) => r.isFirst)` on tab re-tap:
  - **What it does**: Unwinds the tab back to its root screen when the user taps an already active tab (standard Instagram/Twitter behavior).

---

## 📱 ASCII Phone Screen Preview

```
[ Tab 1: Feed Root ]            [ Tab 1: Post Detail ]          [ Switched to Tab 2: Search ]
+----------------------------+  +----------------------------+  +----------------------------+
| Feed                       |  | [←] Post Discussion        |  | Search & Explore           |
+----------------------------+  +----------------------------+  +----------------------------+
|                            |  |                            |  |                            |
|       [ 📄 ARTICLE ]       |  | Post #142: Architecture    |  |        [ 🔍 SEARCH ]       |
|    Feed Stream (Root)      |  |                            |  |                            |
|                            |  | (Bottom bar stays visible!)|  | Search Tab Root            |
| [ Open Post (Drill Down) ] |  |                            |  | (Tab 1 state preserved!)   |
|                            |  | [ Drill to Author ]        |  |                            |
+----------------------------+  +----------------------------+  +----------------------------+
|  (===)                     |  |  (===)                     |  |               (===)        |
|  | 📄 |          🔍        |  |  | 📄 |          🔍        |  |     📄        | 🔍 |       |
|   Feed         Search      |  |   Feed         Search      |  |    Feed       Search       |
+----------------------------+  +----------------------------+  +----------------------------+
```

---

## 🌳 Widget Tree
```
MaterialApp
 └── Root Navigator
      └── MainTabShell (StatefulWidget)
           └── PopScope (Intercepts system back gestures)
                └── Scaffold
                     ├── body: IndexedStack
                     │    ├── Navigator [key: _feedNavKey]
                     │    │    └── Overlay -> FeedRootScreen -> PostDetailScreen -> AuthorProfileScreen
                     │    └── Navigator [key: _searchNavKey]
                     │         └── Overlay -> SearchRootScreen
                     └── bottomNavigationBar: NavigationBar (Persistent across all pushes!)
```

---

## ⚙ Flutter Execution Flow
1. **Initial Mount**: `IndexedStack` initializes both nested `Navigator` widgets with their respective root screens.
2. **User Drills Down**: Inside Tab 1, user clicks `[Open Post]`, then `[Drill into Author]`.
3. **Local Push**: `_feedNavKey` pushes `PostDetailScreen` and `AuthorProfileScreen` on Tab 1's stack. The bottom bar does not move or hide.
4. **Tab Switch**: User taps the "Search" tab. Tab 1 is hidden; Tab 2 is rendered.
5. **Tab Re-Selection**: User taps "Feed" again. Tab 1 is restored showing `AuthorProfileScreen` exactly as left.
6. **Back Button**: User taps Android back button. `PopScope` intercepts it and calls `_feedNavKey.currentState!.pop()`, smoothly backing out to `PostDetailScreen`!

---

## ⚠ Common Beginner Mistakes

### 1. The Exit-App on Back Button Bug
```dart
// ❌ MAJOR FLAW: Omitting PopScope
// When user is on Tab 1, Layer 3 and presses the Android back button:
// Android talks to the ROOT navigator. Root navigator has only 1 route!
// Result: The app EXITS immediately, discarding all user work!

// ✅ GOOD: Wrap in PopScope and forward back events to active nested navigator!
```

### 2. Forgetting `rootNavigator: true` for Full-Screen Modals
```dart
// ❌ MISTAKE: Pushing a full-screen video player without rootNavigator
Navigator.push(context, MaterialPageRoute(builder: (_) => VideoPlayerScreen()));
// The video player is constrained to the body; bottom bar stays visible OVER the video!

// ✅ GOOD: Push to root navigator to cover the bottom bar
Navigator.of(context, rootNavigator: true).push(
  MaterialPageRoute(builder: (_) => VideoPlayerScreen()),
);
```

---

## 🚀 Professional Usage
- **Instagram**: Every bottom tab (Feed, Explore, Reels, Shop, Profile) maintains an independent navigation stack.
- **YouTube Mobile**: Navigating deep into a channel's playlists while the mini-player floats above the bottom bar.
- **Enterprise Tablet POS Systems**: Left panel lists menu categories; right panel drills down into ingredient modifiers independently.

---

## 💡 Best Practices
1. **Always wrap the shell in `PopScope`**: Manage device back buttons so users unwind tab history before exiting.
2. **Implement pop-to-root on tab re-tap**: Tapping the active tab button should call `popUntil((r) => r.isFirst)`.
3. **Use `rootNavigator: true` for dialogs and modals**: Ensures overlays cover bottom navigation bars.
4. **Use `IndexedStack` to wrap sub-navigators**: Prevents destroying tab stacks when toggling between tabs.

---

## 🧪 Try It Yourself
1. Drill down into Tab 1, switch to Tab 2, switch back to Tab 1, and verify that the screen remains at Layer 3.
2. Tap the Android back button and verify that `PopScope` pops the nested screen instead of closing the app.
3. Re-tap the "Feed" tab while on Layer 3 and observe how it pops instantly back to the root feed screen.

---

## 🎯 Challenge
Add a Full-Screen Image Modal:
- Inside `AuthorProfileScreen`, add a button *"View Fullscreen Photo"*.
- Use `Navigator.of(context, rootNavigator: true).push(...)` to push a full-screen image viewer that covers the entire display, including the bottom navigation bar!

---

## ❓ Practice Questions

### Q1. What is the main purpose of Nested Navigation in Flutter?
- A) To make apps run faster
- B) To maintain independent navigation stacks inside individual tabs or sub-trees while keeping bottom bars permanently visible
- C) To bypass Android security
- D) To create circular buttons
*Answer: B* — Keeps navigation histories separate within tabs and preserves chrome bars.

### Q2. How does `Navigator.of(context)` resolve when called inside a nested tab screen?
- A) It always finds the root app navigator
- B) It searches upward and finds the nearest enclosing nested `Navigator` for that tab
- C) It throws a compile error
- D) It restarts the app
*Answer: B* — Context lookups resolve to the nearest ancestor navigator in the tree.

### Q3. How do you push a screen that covers the bottom navigation bar when using nested navigators?
- A) `Navigator.pushFullScreen(context, route)`
- B) `Navigator.of(context, rootNavigator: true).push(route)`
- C) `Navigator.pop(context)`
- D) `Navigator.replace(route)`
*Answer: B* — `rootNavigator: true` bypasses local tab navigators and targets the top-level app navigator.

### Q4. What happens when the user presses the Android back button inside a nested navigator if `PopScope` is NOT implemented?
- A) The nested screen pops normally
- B) The app exits immediately because the root navigator has only 1 route on its stack
- C) The phone restarts
- D) The screen turns red
*Answer: B* — Root navigator receives the system back event and exits the app.

### Q5. What standard UX interaction should occur when a user re-taps an already-active bottom navigation tab?
- A) The app should crash
- B) The active tab's nested navigator should pop all sub-screens back to its root route (`popUntil((r) => r.isFirst)`)
- C) The app should log out
- D) Nothing should happen
*Answer: B* — Standard mobile UX unwinds the tab to its initial root screen.

### Q6. Which widget should wrap nested navigators to keep their stacks alive in memory?
*Answer*: `IndexedStack`.

### Q7. What is required to control a nested navigator from outside its widget subtree?
*Answer*: A dedicated `GlobalKey<NavigatorState>` assigned to the nested `Navigator(key: myKey)`.

### Q8. Output Prediction: What will `currentNavigator.canPop()` return when sitting on `FeedRootScreen`?
*Answer*: `false`. It is at the base of Tab 1's nested stack.

### Q9. Can you nest more than two levels of Navigators in Flutter?
*Answer*: Yes, Flutter's element tree supports arbitrarily deep nested navigators.

### Q10. Write a minimal code snippet pushing a route to the root navigator.
*Answer*:
```dart
Navigator.of(context, rootNavigator: true).push(
  MaterialPageRoute(builder: (context) => const FullscreenModal()),
);
```

---

## 📝 Summary
- **Nested Navigation** embeds independent `Navigator` widgets inside child subtrees (like bottom tabs).
- Keeps the `NavigationBar` permanently visible as users drill into sub-pages.
- Each tab preserves its own independent history stack inside an `IndexedStack`.
- Intercept the system back button with `PopScope` to unwind nested stacks before exiting.
- Use `Navigator.of(context, rootNavigator: true)` when full-screen modals must cover bottom bars.
