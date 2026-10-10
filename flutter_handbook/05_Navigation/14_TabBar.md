# TabBar in Flutter

## 📖 Introduction
While a bottom navigation bar switches between broad, top-level application modules, users frequently need to switch between related sub-categories within a single screen. When viewing a messaging app, you switch between Chats, Status, and Calls. When browsing an app store, you switch between Top Free, Top Grossing, and Categories.

In Flutter, the [TabBar](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/05_Navigation/14_TabBar.md) widget renders a horizontal row of interactive tab labels, typically nestled directly inside the `bottom` slot of an `AppBar`. The active tab is emphasized by an animated indicator bar, bubble, or underline that slides smoothly between positions as tabs are tapped or swiped.

---

## 🤔 Why Do We Need It?
1. **Categorical Organization**: Organizes peer content within the same screen context without requiring navigation stack pushes.
2. **Synchronized Animation**: Automatically tracks swipe gestures, coordinating the sliding indicator bar with screen transitions.
3. **Scrollable Tab Rows**: When an app has 6, 8, or 10 categories, setting `isScrollable: true` transforms the fixed bar into a smooth, horizontally scrollable category strip.
4. **Rich Customization**: Customize underline indicators into rounded pill capsules, adjust label typography, and combine icons with text labels.

---

## 🌍 Real-Life Analogy
Think of a [TabBar](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/05_Navigation/14_TabBar.md) as the **Index Dividers in a Binder Notebook**:

```
+-------------------------------------------------------------+
|                     BINDER NOTEBOOK DIVIDERS                |
|                                                             |
|   +-----------+ +-----------+ +-----------+                 |
|   | [ CHATS ] | |  STATUS   | |   CALLS   |                 |
|   +===========+ +-----------+ +-----------+                 |
|   (Thick Tab)                                               |
|                                                             |
|   All three sections live inside the SAME notebook.         |
|   Tapping a tab flips open the pages for that exact         |
|   section instantly. You don't pick up a new notebook!      |
+-------------------------------------------------------------+
```

---

## 🧠 Internal Working
To understand `TabBar`, look at how it binds to its controller:

```
                     DefaultTabController (or TabController)
                     length: 3, animation: AnimationController
                                  │
         ┌────────────────────────┴────────────────────────┐
         ▼                                                 ▼
      TabBar                                          TabBarView
  Renders Tabs: [A, B, C]                        Renders Pages: [1, 2, 3]
  Indicator paints at index 0                     Displays Page 0
         │                                                 ▲
         │ User taps Tab B                                 │
         └─────────────► TabController.animateTo(1) ───────┘
                         Indicator slides smoothly to B
                         Pages animate horizontally to 2
```

1. **The TabController Engine**: A `TabBar` cannot function without a `TabController` (provided explicitly or via ancestor `DefaultTabController`). The controller tracks `index`, `previousIndex`, and a normalized double `animation.value` between `0.0` and `length - 1.0`.
2. **Indicator Painter**: `TabBar` uses a `CustomPainter` to draw the indicator line or box. As the controller's animation updates, the painter recalculates the bounding box coordinates of the active tab.

---

## ⚙ Syntax

### Basic TabBar in AppBar
```dart
AppBar(
  title: const Text('My App'),
  bottom: const TabBar(
    tabs: [
      Tab(icon: Icon(Icons.chat), text: 'Chats'),
      Tab(icon: Icon(Icons.photo_camera), text: 'Status'),
      Tab(icon: Icon(Icons.call), text: 'Calls'),
    ],
  ),
)
```

### Horizontally Scrollable TabBar
```dart
TabBar(
  isScrollable: true,
  tabAlignment: TabAlignment.start,
  tabs: categories.map((cat) => Tab(text: cat)).toList(),
)
```

---

## 📋 Important Properties

| Property | Type | Description | Default |
| :--- | :--- | :--- | :--- |
| `tabs` | `List<Widget>` | The tab widgets to display (typically `Tab` widgets). | *Required* |
| `controller` | `TabController?` | Controller coordinating the selection state. | Nearest `DefaultTabController` |
| `isScrollable` | `bool` | If `true`, tabs scroll horizontally instead of sharing equal widths. | `false` |
| `indicatorColor` | `Color?` | The color of the active indicator line. | Theme secondary/primary |
| `indicatorWeight` | `double` | The thickness of the underline indicator. | `2.0` |
| `indicatorSize` | `TabBarIndicatorSize` | Whether indicator matches the width of the `tab` or only the `label`. | `TabBarIndicatorSize.tab` |
| `indicator` | `Decoration?` | Custom decoration (e.g. rounded capsule pill). | `UnderlineTabIndicator` |
| `labelColor` | `Color?` | Text and icon color of the active tab. | Theme default |
| `unselectedLabelColor`| `Color?` | Text and icon color of inactive tabs. | Theme default |

---

## 💻 Complete Runnable Example
The following production-ready application demonstrates:
1. An embedded [TabBar](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/05_Navigation/14_TabBar.md) nested inside `AppBar.bottom`.
2. A custom rounded **Pill Capsule Indicator**.
3. Combining icons, text labels, and notification badges inside tabs.
4. Seamless coordination with [DefaultTabController](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/05_Navigation/16_DefaultTabController.md).

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const TabBarMasterclassApp());
}

class TabBarMasterclassApp extends StatelessWidget {
  const TabBarMasterclassApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'TabBar Masterclass',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const ChannelsTabScreen(),
    );
  }
}

class ChannelsTabScreen extends StatelessWidget {
  const ChannelsTabScreen({super.key});

  @override
  Widget build(BuildContext context) {
    // DefaultTabController coordinates TabBar with TabBarView automatically
    return DefaultTabController(
      length: 4,
      child: Scaffold(
        appBar: AppBar(
          title: const Text('Developer Community'),
          centerTitle: true,
          bottom: TabBar(
            isScrollable: true, // Allows smooth horizontal scrolling
            tabAlignment: TabAlignment.start,
            // Custom Rounded Pill Indicator
            indicator: BoxDecoration(
              borderRadius: BorderRadius.circular(50),
              color: Colors.deepPurple,
            ),
            indicatorSize: TabBarIndicatorSize.tab,
            labelColor: Colors.white,
            unselectedLabelColor: Colors.grey.shade700,
            labelStyle: const TextStyle(fontWeight: FontWeight.bold),
            tabs: const [
              Tab(
                icon: Icon(Icons.forum_outlined),
                text: 'Discussions',
              ),
              Tab(
                icon: Icon(Icons.code),
                text: 'Code Snippets',
              ),
              Tab(
                icon: Badge(
                  label: Text('5'),
                  child: Icon(Icons.bolt),
                ),
                text: 'Trending',
              ),
              Tab(
                icon: Icon(Icons.announcement_outlined),
                text: 'Announcements',
              ),
            ],
          ),
        ),
        body: const TabBarView(
          children: [
            _TabBodyContent(title: 'Discussions Hub', icon: Icons.forum_outlined),
            _TabBodyContent(title: 'Code Repository', icon: Icons.code),
            _TabBodyContent(title: 'Trending Articles', icon: Icons.bolt),
            _TabBodyContent(title: 'Latest Announcements', icon: Icons.announcement_outlined),
          ],
        ),
      ),
    );
  }
}

class _TabBodyContent extends StatelessWidget {
  final String title;
  final IconData icon;

  const _TabBodyContent({required this.title, required this.icon});

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Icon(icon, size: 72, color: Colors.deepPurple),
          const SizedBox(height: 16),
          Text(title, style: const TextStyle(fontSize: 22, fontWeight: FontWeight.bold)),
          const SizedBox(height: 8),
          const Text('Swipe horizontally or tap tabs above to navigate.', style: TextStyle(color: Colors.grey)),
        ],
      ),
    );
  }
}
```

---

## 🔍 Line-by-Line Example Explanation
- `DefaultTabController(length: 4, child: Scaffold(...))`:
  - **What it does**: Creates and injects an unmanaged `TabController` for 4 tabs into the widget tree.
  - **Why it matters**: `TabBar` and `TabBarView` both look up this controller, keeping their animations in sync without requiring a `StatefulWidget`.
- `indicator: BoxDecoration(...)`:
  - **What it does**: Replaces the default flat underline indicator with a solid rounded purple capsule.
  - **Why it is popular**: Produces modern, pill-shaped category tabs like those found in YouTube, Twitter, and Spotify.
- `isScrollable: true` & `tabAlignment: TabAlignment.start`:
  - **What it does**: Ensures that having 4 long tabs doesn't squash text or throw `RenderFlex` overflow errors on smaller screens.
- `TabBarView(children: [...])`:
  - **What it does**: Renders the swipeable content pages corresponding to each tab. (Covered in detail in Chapter 15).

---

## 📱 ASCII Phone Screen Preview

```
+------------------------------------------+
|           Developer Community            |
|                                          |
| (============)                           |
| (  [💬] Dis.. )  [<>] Code   [⚡]5  [📢]  |
| (============)                           |
+------------------------------------------+
|                                          |
|                                          |
|                [ 💬 FORUM ]              |
|                                          |
|              Discussions Hub             |
|    Swipe horizontally or tap tabs...     |
|                                          |
|                                          |
+------------------------------------------+
```

---

## 🌳 Widget Tree
```
MaterialApp
 └── DefaultTabController [length: 4]
      └── Scaffold
           ├── AppBar
           │    └── bottom: TabBar [isScrollable: true, indicator: BoxDecoration]
           │         ├── Tab ("Discussions")
           │         ├── Tab ("Code Snippets")
           │         ├── Tab ("Trending", with Badge)
           │         └── Tab ("Announcements")
           └── body: TabBarView
                ├── _TabBodyContent (Discussions)
                ├── _TabBodyContent (Code Snippets)
                ├── _TabBodyContent (Trending)
                └── _TabBodyContent (Announcements)
```

---

## ⚙ Flutter Execution Flow
1. **Init**: `DefaultTabController` establishes initial index `0`.
2. **Painting**: `TabBar` renders 4 tabs. Tab 0 receives the purple pill indicator; its label is styled white.
3. **User Action**: The user taps the "Trending" tab (index 2).
4. **Animation**: `TabController` animates its value from `0.0` to `2.0`.
5. **Simultaneous Transition**:
   - The purple pill indicator slides smoothly across the horizontal bar.
   - `TabBarView` scrolls the body horizontally to page index 2.
6. **Settle**: The indicator snaps flush under "Trending", and the "Trending Articles" view is displayed.

---

## ⚠ Common Beginner Mistakes

### 1. Tab Count Mismatch Crash
```dart
// ❌ CRASH: length is 3, but 4 tabs provided!
DefaultTabController(
  length: 3, // Controller expects 3
  child: Scaffold(
    appBar: AppBar(
      bottom: TabBar(
        tabs: [Tab(text: 'A'), Tab(text: 'B'), Tab(text: 'C'), Tab(text: 'D')], // 4 tabs!
      ),
    ),
  ),
)
// Result: Crash! Assertion failed: "controller.length == tabs.length"

// ✅ GOOD: Ensure length strictly matches tabs.length and children.length!
```

### 2. Forgetting `isScrollable: true` with Many Tabs
```dart
// ❌ LAYOUT CRASH: 6 tabs crammed into equal space
TabBar(
  tabs: [Tab1, Tab2, Tab3, Tab4, Tab5, Tab6], // Overflows on small screens!
)
// Result: RenderFlex overflowed by 112 pixels!

// ✅ GOOD: Enable horizontal scrolling
TabBar(
  isScrollable: true,
  tabAlignment: TabAlignment.start,
  tabs: [...],
)
```

---

## 🚀 Professional Usage
- **Social Media (e.g., X/Twitter)**: User profile sub-tabs: Posts, Replies, Highlights, Media, Likes.
- **Communication (e.g., WhatsApp, Telegram)**: Primary header tabs: Chats, Calls, Channels.
- **E-Commerce Product Categories**: Department pills: All, Electronics, Fashion, Home, Toys.

---

## 💡 Best Practices
1. **Always match lengths**: `DefaultTabController.length == TabBar.tabs.length == TabBarView.children.length`.
2. **Use `isScrollable: true` for 4 or more tabs**: Guarantees responsive rendering across all screen widths.
3. **Use custom `indicator` decorations for modern apps**: Upgrade from the default flat line to rounded pill shapes.
4. **Keep tab labels short**: 1 or 2 words maximum to ensure high legibility.

---

## 🧪 Try It Yourself
1. Change `indicatorSize: TabBarIndicatorSize.label` and observe how the indicator fits only the text width.
2. Remove `isScrollable: true` and observe how tabs divide equal screen width.
3. Customize `indicatorColor` to bright amber and change indicator thickness with `indicatorWeight: 4.0`.

---

## 🎯 Challenge
Build an "App Store Top Charts" TabBar:
- 3 Tabs: "Top Free", "Top Paid", "Top Grossing".
- Underline indicator matching your brand accent color.
- Below each tab, render a list of 10 ranked apps with ranking badges (1, 2, 3...).

---

## ❓ Practice Questions

### Q1. What happens if `TabBar.tabs.length` does not equal `TabController.length`?
- A) Flutter ignores the extra tabs
- B) Flutter crashes with an assertion failure
- C) The app restarts
- D) The extra tabs are hidden
*Answer: B* — The lengths must match exactly.

### Q2. Where is a `TabBar` most commonly placed within a `Scaffold`?
- A) In the `drawer`
- B) In the `bottom` slot of an `AppBar`
- C) In the `floatingActionButton`
- D) Inside `MaterialApp`
*Answer: B* — `AppBar.bottom` is designed specifically to host preferred-size widgets like `TabBar`.

### Q3. How do you allow a `TabBar` to scroll horizontally when there are many tabs?
- A) Wrap it in a `SingleChildScrollView`
- B) Set `isScrollable: true`
- C) Set `scrollDirection: Axis.horizontal`
- D) Set `physics: BouncingScrollPhysics()`
*Answer: B* — `isScrollable: true` enables horizontal category scrolling.

### Q4. What is the difference between `TabBarIndicatorSize.tab` and `TabBarIndicatorSize.label`?
- A) `tab` is for tablets; `label` is for phones
- B) `tab` spans the full width of the tab compartment; `label` constrains the indicator to the exact width of the text/icon
- C) `label` is deprecated
- D) There is no difference
*Answer: B* — Controls whether the indicator extends across the whole cell or fits the text label.

### Q5. What controller is required by `TabBar` to manage tab selection and animation?
- A) `ScrollController`
- B) `TabController` (or an ancestor `DefaultTabController`)
- C) `PageController`
- D) `TextEditingController`
*Answer: B* — `TabController` coordinates tab state and animation timing.

### Q6. Which property customizes the indicator from a plain underline into a rounded pill or capsule?
*Answer*: The `indicator` property (e.g. `indicator: BoxDecoration(borderRadius: BorderRadius.circular(50), color: Colors.blue)`).

### Q7. What property changes the color of inactive tabs?
*Answer*: `unselectedLabelColor`.

### Q8. Output Prediction: What will be displayed if `DefaultTabController` is omitted and no explicit `controller` is passed to `TabBar`?
*Answer*: Flutter throws a runtime assertion failure stating that no `TabController` was found in the tree.

### Q9. Can you nest a `Badge` widget inside a `Tab`'s icon?
*Answer*: Yes, `Tab(icon: Badge(label: Text('3'), child: Icon(Icons.notifications)))`.

### Q10. Write a minimal code snippet declaring a 2-tab `TabBar` with Chat and Call icons.
*Answer*:
```dart
TabBar(
  tabs: const [
    Tab(icon: Icon(Icons.chat), text: 'Chats'),
    Tab(icon: Icon(Icons.call), text: 'Calls'),
  ],
)
```

---

## 📝 Summary
- [TabBar](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/05_Navigation/14_TabBar.md) renders a horizontal row of category tabs, typically placed in `AppBar.bottom`.
- Always coordinate with a `TabController` or wrap in an ancestor `DefaultTabController`.
- Enable `isScrollable: true` when presenting 4 or more tabs to prevent layout overflow.
- Customize the active indicator into modern pill shapes using `indicator: BoxDecoration(...)`.
