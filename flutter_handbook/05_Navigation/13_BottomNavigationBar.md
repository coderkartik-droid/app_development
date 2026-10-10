# BottomNavigationBar & NavigationBar in Flutter

## 📖 Introduction
On modern large-screen smartphones, users navigate primarily with their thumbs while holding devices with one hand. Placing critical navigation controls at the very top of the screen forces uncomfortable hand readjustments (the "thumb stretch" problem).

To provide frictionless thumb-zone navigation, mobile operating systems standardized on bottom tab bars. In Flutter, primary bottom-level navigation is handled by two widgets:
- [BottomNavigationBar](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/05_Navigation/13_BottomNavigationBar.md): The classic Material 2 bottom navigation bar.
- [NavigationBar](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/05_Navigation/13_BottomNavigationBar.md): The modern **Material 3** navigation bar featuring pill-shaped selection indicators, refined elevation tokens, and built-in badge support.

Unlike standard stack navigation (`push`/`pop`), bottom bars switch between peer, top-level destinations (e.g. Home, Search, Library, Profile) while keeping the user at the root of the app.

---

## 🤔 Why Do We Need It?
1. **Thumb-Zone Ergonomics**: Places 3 to 5 core destinations within effortless reach of one-handed thumb gestures.
2. **Persistent Spatial Context**: Keeps core destinations permanently visible so users never feel lost inside sub-flows.
3. **State Preservation with `IndexedStack`**: Switching between tabs does not destroy or rebuild screens when paired with [IndexedStack](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/05_Navigation/13_BottomNavigationBar.md)—scroll positions, entered form text, and active media playback stay intact!
4. **Live Badging**: Effortlessly informs users of unread messages or new notifications on specific tabs using Flutter's `Badge` widget.

---

## 🌍 Real-Life Analogy
Think of a bottom navigation bar as a **Multi-Channel TV Remote or Car Radio Presets**:

```
+-------------------------------------------------------------+
|                      CAR RADIO PRESETS                      |
|                                                             |
|   Preset 1: News    | Preset 2: Music  | Preset 3: Sports   |
|   [== (ACTIVE) ==]  | [            ]   | [             ]    |
|                                                             |
|   You tap Preset 2:                                         |
|   -> The radio immediately switches to the music station.   |
|                                                             |
|   You tap Preset 1 again:                                   |
|   -> It switches back to the news station.                  |
|                                                             |
|   You didn't "close" the news station; both stations        |
|   continue existing side-by-side!                           |
+-------------------------------------------------------------+
```

---

## 🧠 Internal Working
To prevent losing screen state when switching tabs, understand how `IndexedStack` preserves widget memory:

```
                      Scaffold
                         │
        ┌────────────────┴────────────────┐
        ▼                                 ▼
   IndexedStack (body)            NavigationBar (M3)
   index: _currentIndex          selectedIndex: _currentIndex
        │                                 │
   ┌────┼────┐                            │
   ▼    ▼    ▼                            ▼
 Screen0 Screen1 Screen2           User taps Tab #1
(Alive) (Alive) (Alive)                   │
   │                                      ▼
   └─────────────────────────── setState(() => _currentIndex = 1)
```

1. **The State Destruction Trap**:
   If you write `body: _screens[_currentIndex]`, Flutter removes the old widget from the tree and mounts a fresh one on every tab tap. Scroll offsets reset to top, and entered text is destroyed.
2. **The `IndexedStack` Solution**:
   By using `body: IndexedStack(index: _currentIndex, children: _screens)`, **all tabs remain mounted in memory simultaneously**. Flutter simply toggles the visibility and hit-testing of inactive tabs, preserving their complete state!

---

## ⚙ Syntax

### Modern Material 3 NavigationBar (Recommended)
```dart
Scaffold(
  body: IndexedStack(
    index: _currentIndex,
    children: const [
      HomeScreen(),
      ExploreScreen(),
      ProfileScreen(),
    ],
  ),
  bottomNavigationBar: NavigationBar(
    selectedIndex: _currentIndex,
    onDestinationSelected: (int index) {
      setState(() => _currentIndex = index);
    },
    destinations: const [
      NavigationDestination(icon: Icon(Icons.home_outlined), selectedIcon: Icon(Icons.home), label: 'Home'),
      NavigationDestination(icon: Icon(Icons.explore_outlined), label: 'Explore'),
      NavigationDestination(icon: Icon(Icons.person_outlined), label: 'Profile'),
    ],
  ),
)
```

---

## 📋 Comparison & Important Properties

| Feature | `BottomNavigationBar` (Material 2) | `NavigationBar` (Material 3 - Recommended) |
| :--- | :--- | :--- |
| **Child Items** | `List<BottomNavigationBarItem>` | `List<NavigationDestination>` |
| **Current Index** | `currentIndex` | `selectedIndex` |
| **Selection Callback** | `onTap: (index) => ...` | `onDestinationSelected: (index) => ...` |
| **Indicator Style** | Icon tint color changes | Distinct pill-shaped capsule background indicator |
| **Default Height** | `56.0` logical pixels | `80.0` logical pixels (Material 3 standard) |
| **More than 3 items** | Background turns white unless `type: BottomNavigationBarType.fixed` | Handled gracefully out of the box without layout bugs |

---

## 💻 Complete Runnable Example
The following production-ready application demonstrates:
1. Modern Material 3 [NavigationBar](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/05_Navigation/13_BottomNavigationBar.md) with pill indicators and selected icon variants.
2. An [IndexedStack](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/05_Navigation/13_BottomNavigationBar.md) preserving scroll position on long lists when switching tabs.
3. Notification `Badge` indicators on specific tabs.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const NavigationBarMasterclassApp());
}

class NavigationBarMasterclassApp extends StatelessWidget {
  const NavigationBarMasterclassApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'NavigationBar Masterclass',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const MainBottomNavShell(),
    );
  }
}

class MainBottomNavShell extends StatefulWidget {
  const MainBottomNavShell({super.key});

  @override
  State<MainBottomNavShell> createState() => _MainBottomNavShellState();
}

class _MainBottomNavShellState extends State<MainBottomNavShell> {
  int _currentIndex = 0;

  // List of root tab screens
  final List<Widget> _screens = const [
    HomeFeedTab(),
    ExploreSearchTab(),
    NotificationsTab(),
    ProfileTab(),
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      // CRITICAL: IndexedStack preserves scroll positions and state across tab switches!
      body: IndexedStack(
        index: _currentIndex,
        children: _screens,
      ),

      // Material 3 Navigation Bar
      bottomNavigationBar: NavigationBar(
        selectedIndex: _currentIndex,
        onDestinationSelected: (int index) {
          setState(() {
            _currentIndex = index;
          });
        },
        destinations: const [
          NavigationDestination(
            icon: Icon(Icons.home_outlined),
            selectedIcon: Icon(Icons.home, color: Colors.deepPurple),
            label: 'Home',
          ),
          NavigationDestination(
            icon: Icon(Icons.search),
            selectedIcon: Icon(Icons.search, color: Colors.deepPurple),
            label: 'Explore',
          ),
          NavigationDestination(
            icon: Badge(
              label: Text('3'),
              child: Icon(Icons.notifications_outlined),
            ),
            selectedIcon: Badge(
              label: Text('3'),
              child: Icon(Icons.notifications, color: Colors.deepPurple),
            ),
            label: 'Alerts',
          ),
          NavigationDestination(
            icon: Icon(Icons.person_outline),
            selectedIcon: Icon(Icons.person, color: Colors.deepPurple),
            label: 'Profile',
          ),
        ],
      ),
    );
  }
}

// ---------------------------------------------------------------------------
// TAB 1: Long Scrollable Feed (Tests State Preservation)
// ---------------------------------------------------------------------------
class HomeFeedTab extends StatelessWidget {
  const HomeFeedTab({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Home Feed'), centerTitle: true),
      body: ListView.builder(
        padding: const EdgeInsets.all(16),
        itemCount: 40,
        itemBuilder: (context, index) {
          return Card(
            margin: const EdgeInsets.only(bottom: 12),
            child: ListTile(
              leading: CircleAvatar(child: Text('${index + 1}')),
              title: Text('Feed Story Article #${index + 1}'),
              subtitle: const Text('Scroll down, switch tabs, and come back—scroll position is preserved!'),
            ),
          );
        },
      ),
    );
  }
}

// ---------------------------------------------------------------------------
// TAB 2: Explore & Search
// ---------------------------------------------------------------------------
class ExploreSearchTab extends StatelessWidget {
  const ExploreSearchTab({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Explore Topics'), centerTitle: true),
      body: const Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(Icons.explore, size: 72, color: Colors.deepPurple),
            SizedBox(height: 16),
            Text('Trending Topics & Categories', style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
          ],
        ),
      ),
    );
  }
}

// ---------------------------------------------------------------------------
// TAB 3: Notifications
// ---------------------------------------------------------------------------
class NotificationsTab extends StatelessWidget {
  const NotificationsTab({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Notifications'), centerTitle: true),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: const [
          ListTile(
            leading: Icon(Icons.star, color: Colors.amber),
            title: Text('New follower alert'),
            subtitle: Text('Sarah started following your repositories.'),
          ),
          Divider(),
          ListTile(
            leading: Icon(Icons.comment, color: Colors.blue),
            title: Text('Comment on your post'),
            subtitle: Text('Alex: "Great tutorial on IndexedStack!"'),
          ),
        ],
      ),
    );
  }
}

// ---------------------------------------------------------------------------
// TAB 4: Profile
// ---------------------------------------------------------------------------
class ProfileTab extends StatelessWidget {
  const ProfileTab({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('My Profile'), centerTitle: true),
      body: const Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            CircleAvatar(radius: 48, child: Icon(Icons.person, size: 48)),
            SizedBox(height: 16),
            Text('Alex Morgan', style: TextStyle(fontSize: 22, fontWeight: FontWeight.bold)),
            Text('Senior Flutter Developer', style: TextStyle(color: Colors.grey)),
          ],
        ),
      ),
    );
  }
}
```

---

## 🔍 Line-by-Line Example Explanation
- `body: IndexedStack(index: _currentIndex, children: _screens)`:
  - **What it does**: Holds all 4 tab widgets in memory, displaying only the child matching `index`.
  - **Why it matters**: Without `IndexedStack`, if you scroll to item #30 on `HomeFeedTab`, switch to `Profile`, and return to `Home`, the list snaps back to item #1! `IndexedStack` keeps item #30 right where you left it.
- `NavigationBar(...)`:
  - **What it does**: Renders the Material 3 navigation bar with pill selection capsules and standard 80px height.
  - **Why it is superior**: Eliminates the bugs of legacy `BottomNavigationBar` where adding a 4th item caused the bar background to turn white and labels to vanish.
- `NavigationDestination(icon: Badge(label: Text('3'), child: ...))`:
  - **What it does**: Embeds a live notification pill count over the alert bell icon.
- `selectedIcon: Icon(Icons.home, color: Colors.deepPurple)`:
  - **What it does**: Switches from an outlined icon (`home_outlined`) to a solid filled icon (`home`) when active, fulfilling Material 3 design guidelines.

---

## 📱 ASCII Phone Screen Preview

```
+------------------------------------------+
|                Home Feed                 |
+------------------------------------------+
|  +------------------------------------+  |
|  | (1) Feed Story Article #1          |  |
|  +------------------------------------+  |
|  | (2) Feed Story Article #2          |  |
|  +------------------------------------+  |
|  | (3) Feed Story Article #3          |  |
|  +------------------------------------+  |
|  | (Scroll position is preserved!)    |  |
|  +------------------------------------+  |
|                                          |
| ======================================== |
|   (===)                                  |
|   | ⌂ |       🔍        🔔 [3]      👤   |
|   Home     Explore     Alerts    Profile |
|  [Tab 0]    [Tab 1]    [Tab 2]   [Tab 3] |
+------------------------------------------+
```

---

## 🌳 Widget Tree
```
MaterialApp
 └── MainBottomNavShell (StatefulWidget)
      └── Scaffold
           ├── body: IndexedStack [index: _currentIndex]
           │    ├── HomeFeedTab (Alive & preserved in memory)
           │    ├── ExploreSearchTab (Alive & preserved in memory)
           │    ├── NotificationsTab (Alive & preserved in memory)
           │    └── ProfileTab (Alive & preserved in memory)
           └── bottomNavigationBar: NavigationBar (Material 3)
                ├── NavigationDestination (Home)
                ├── NavigationDestination (Explore)
                ├── NavigationDestination (Alerts, with Badge)
                └── NavigationDestination (Profile)
```

---

## ⚙ Flutter Execution Flow
1. **Initial Mount**: `IndexedStack` initializes all 4 tabs. Tab 0 (`HomeFeedTab`) is marked visible.
2. **User Scrolls**: User scrolls down to item #25 on Tab 0.
3. **Tab Switch**: User taps the "Alerts" destination in the `NavigationBar`.
4. **Callback**: `onDestinationSelected(2)` fires.
5. **State Update**: `setState(() => _currentIndex = 2)` rebuilds the shell.
6. **Stack Switching**: `IndexedStack` hides Tab 0 and renders Tab 2 (`NotificationsTab`). Tab 0's scroll offset of item #25 remains safe in memory.
7. **Return Tap**: User taps "Home". Tab 0 becomes visible again instantly at item #25 with zero reloading!

---

## ⚠ Common Beginner Mistakes

### 1. The Screen State Loss Bug (Using Direct Indexing)
```dart
// ❌ MAJOR UX BUG: Destroys screen state on every tab switch!
body: _screens[_currentIndex], // Disposes old screen! Recreates new screen!
// Result: Scroll positions reset to top, active videos stop, text fields cleared!

// ✅ GOOD: Use IndexedStack to preserve all tab states
body: IndexedStack(
  index: _currentIndex,
  children: _screens,
),
```

### 2. The 4-Item White Background Glitch in `BottomNavigationBar`
```dart
// ❌ LEGACY M2 BUG: Adding 4 items shifts type to 'shifting'
BottomNavigationBar(
  items: [item1, item2, item3, item4], // Background turns white, labels vanish!
)

// ✅ FIX: Use modern NavigationBar (M3) OR explicitly set fixed type
BottomNavigationBar(
  type: BottomNavigationBarType.fixed, // Fixes M2 bug
  items: [...],
)
```

---

## 🚀 Professional Usage
- **Social Media (e.g., Instagram, X/Twitter)**: Home Feed $\to$ Search $\to$ Post Creation $\to$ Notifications $\to$ Profile.
- **Music & Podcasts (e.g., Spotify)**: Home $\to$ Search $\to$ Your Library. Mini-player floats directly above the `NavigationBar`.
- **E-Commerce (e.g., Amazon)**: Home $\to$ Profile $\to$ Cart $\to$ Menu.

---

## 💡 Best Practices
1. **Prefer Material 3 `NavigationBar`**: Avoid legacy `BottomNavigationBar`; `NavigationBar` provides superior theming, pill indicators, and zero layout bugs with $>3$ items.
2. **Always pair with `IndexedStack`**: Guarantees that users don't lose their place when switching between tabs.
3. **Limit tabs to between 3 and 5**: Having fewer than 3 items wastes horizontal space; having more than 5 crowds the screen.
4. **Use solid icons for selected states**: Pair `home_outlined` (inactive) with filled `home` (active) for clear visual affordance.

---

## 🧪 Try It Yourself
1. Scroll down `HomeFeedTab` to item #30, switch to `Profile`, switch back to `Home`, and observe that the scroll position did not reset.
2. Replace `IndexedStack` with `body: _screens[_currentIndex]` and observe how switching tabs now wipes the scroll position.
3. Add a live badge counter that increments whenever a button is clicked.

---

## 🎯 Challenge
Build an "App Store Tab Shell":
- Tab 1: Games List (with search bar).
- Tab 2: Apps List.
- Tab 3: Arcade.
- Tab 4: Search Tab with an active `TextField`.
- Verify that typing text into the Search tab persists even when switching between Games and Apps!

---

## ❓ Practice Questions

### Q1. What is the main UX purpose of bottom navigation bars?
- A) To look like desktop websites
- B) To place primary, top-level app destinations within the ergonomic "thumb-zone" for one-handed mobile use
- C) To hide secondary settings
- D) To reduce network usage
*Answer: B* — Bottom placement optimizes thumb ergonomics on large modern smartphone screens.

### Q2. Why does `body: _screens[_currentIndex]` cause scroll positions to reset when switching tabs?
- A) Flutter lists do not support scrolling
- B) Because Flutter destroys and unmounts the old screen widget when the index changes, creating a brand new widget instance upon return
- C) It only happens in debug mode
- D) `_screens` must be const
*Answer: B* — Direct indexing tears down the widget subtree on every switch.

### Q3. Which widget preserves the state and scroll positions of all tabs simultaneously?
- A) `Stack`
- B) `IndexedStack`
- C) `PageView`
- D) `Wrap`
*Answer: B* — `IndexedStack` keeps all child widgets mounted, toggling only visual rendering.

### Q4. What is the recommended Material 3 replacement for `BottomNavigationBar`?
- A) `TabBar`
- B) `NavigationBar`
- C) `Drawer`
- D) `CupertinoTabBar`
*Answer: B* — `NavigationBar` is the official Material 3 specification widget.

### Q5. What child widgets are required inside the `destinations` property of `NavigationBar`?
- A) `BottomNavigationBarItem`
- B) `NavigationDestination`
- C) `ListTile`
- D) `Tab`
*Answer: B* — `NavigationBar` requires a list of `NavigationDestination` widgets.

### Q6. How do you display an unread notification count badge on a tab icon?
*Answer*: Wrap the icon widget inside Flutter's `Badge(label: Text('5'), child: Icon(...))` widget.

### Q7. What property is used in `NavigationBar` to listen to tab taps?
*Answer*: `onDestinationSelected: (int index) { ... }`.

### Q8. Output Prediction: What will `_currentIndex` be if the user taps the third item in a zero-indexed bar?
*Answer*: `2`.

### Q9. What visual indicator does Material 3 `NavigationBar` draw behind the active tab icon?
*Answer*: A pill-shaped capsule background indicator (tonal indicator).

### Q10. Write a minimal code snippet defining a 3-tab `NavigationBar`.
*Answer*:
```dart
NavigationBar(
  selectedIndex: _index,
  onDestinationSelected: (i) => setState(() => _index = i),
  destinations: const [
    NavigationDestination(icon: Icon(Icons.home), label: 'Home'),
    NavigationDestination(icon: Icon(Icons.search), label: 'Search'),
    NavigationDestination(icon: Icon(Icons.person), label: 'Profile'),
  ],
)
```

---

## 📝 Summary
- Bottom navigation bars provide thumb-friendly navigation across 3 to 5 primary destinations.
- Use modern Material 3 [NavigationBar](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/05_Navigation/13_BottomNavigationBar.md) over legacy `BottomNavigationBar`.
- Always wrap body screens in an [IndexedStack](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/05_Navigation/13_BottomNavigationBar.md) to preserve scroll positions, text field inputs, and media states.
- Pair outlined icons for unselected states with solid filled icons for selected states.
- Embed the `Badge` widget directly on icons to highlight live notifications.
