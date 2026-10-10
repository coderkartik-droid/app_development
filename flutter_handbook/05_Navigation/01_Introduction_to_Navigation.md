# Introduction to Navigation in Flutter

## 📖 Introduction
A mobile application is rarely a single, static screen. Whether browsing an online store, chatting with friends, or ordering food, users constantly transition through a journey: from a splash screen to an onboarding carousel, through a login form, into a product catalog, and finally to a checkout receipt.

In Flutter, moving between different screens is called **Navigation**, and the individual screens or pages are referred to as **Routes**. Unlike traditional web development where pages correspond to separate HTML files loaded via browser URLs, a screen in Flutter is simply a **Widget**. 

Navigation in Flutter is fundamentally managed through a **Stack data structure** (Last-In, First-Out or LIFO). When you open a new screen, Flutter pushes a new widget on top of the stack. When the user taps the back button, Flutter pops the topmost widget off the stack, seamlessly revealing the previous screen underneath without reloading it or losing its scroll position.

---

## 🤔 Why Do We Need It?
Without a structured navigation system:
1. **Screen State Chaos**: You would have to manage massive conditional `if-else` blocks or switches inside a single giant widget to toggle screen visibility manually.
2. **Broken Platform Gestures**: The physical or virtual back button on Android and the swipe-from-left-edge gesture on iOS would fail to work, confusing users.
3. **No Smooth Transitions**: Native screen transition animations—such as screens sliding in horizontally from the right on iOS or fading and scaling up on Android—would have to be animated by hand using custom `AnimationController`s.
4. **Memory Waste**: Inactive screens wouldn't be paused, hidden, or disposed of properly, rapidly consuming device RAM.

With Flutter's Navigation system:
- Screens are encapsulated into self-contained widgets.
- History is maintained automatically via a navigation stack.
- Native platform conventions (Android back button, iOS swipe-to-back gesture) work out-of-the-box.
- Passing data forward and returning data backward is clean and asynchronous using standard Dart `Future`s.

---

## 🌍 Real-Life Analogy
Think of Flutter's navigation stack as a **Stack of Dinner Plates at a Cafeteria Buffet**:

```
+-------------------------------------------------------------+
|                      THE CAFETERIA STACK                    |
|                                                             |
|   +-----------------------------------------------------+   |
|   | Plate 3: Order Confirmation Screen (Topmost / Active)|  |  <- PUSH (User sees this)
|   +-----------------------------------------------------+   |
|   | Plate 2: Shopping Cart Screen       (Hidden Under)  |   |
|   +-----------------------------------------------------+   |
|   | Plate 1: Product Catalog Screen     (Base of Stack) |   |
|   +-----------------------------------------------------+   |
|                                                             |
|   RULES:                                                    |
|   1. You can only look at and eat from the TOP plate.       |
|   2. When done, you REMOVE (POP) the top plate.             |
|   3. The plate directly beneath it becomes visible again,   |
|      in the exact same condition you left it!               |
+-------------------------------------------------------------+
```

---

## 🧠 Internal Working
To master Flutter navigation, you must understand how `Navigator`, `Route`, and `Overlay` collaborate behind the scenes:

```
                            MaterialApp
                                 │
                                 ▼
                     Navigator (StatefulWidget)
                                 │
                                 ▼
                      NavigatorState (_history)
                                 │
               ┌─────────────────┴─────────────────┐
               ▼                                   ▼
      Overlay (Stack Layer)               Route Stack (List<Route>)
               │                                   │
               ▼                                   ▼
     Active OverlayEntry                 [ Route 1, Route 2 (Active) ]
               │                                   │
               ▼                                   ▼
     Renders on Device Screen            Topmost Route gets Focus
```

1. **`Navigator` Widget**: A specialized widget that manages a stack of `Route` objects. `MaterialApp` automatically creates a root `Navigator` for you.
2. **`NavigatorState`**: The persistent state object that holds the internal `_history` list of routes.
3. **`Route`**: An abstraction representing a single screen or page. It defines the widget to render, the entrance/exit transition animations, and whether the route is modal (blocking interaction with screens beneath it).
4. **`Overlay`**: Beneath the surface, `Navigator` renders routes as `OverlayEntry` objects inside a Flutter `Overlay`. This allows popups, dialogs, and transition animations to float above inactive screens.
5. **Platform Dispatch**: When the user presses the Android hardware back button or performs an iOS edge swipe, the operating system sends a pop request via the `SystemChannels.navigation` platform channel. Flutter's root `Navigator` catches this and pops the top route.

---

## ⚙ Syntax

### 1. Pushing a New Route (`Navigator.push`)
To navigate to a new screen, push a `MaterialPageRoute` containing the destination widget:

```dart
Navigator.push(
  context,
  MaterialPageRoute(
    builder: (BuildContext context) => const DestinationScreen(),
  ),
);
```

### 2. Returning to the Previous Route (`Navigator.pop`)
To leave the current screen and return to the previous one:

```dart
Navigator.pop(context);
```

### 3. Checking If You Can Pop (`Navigator.canPop`)
Before popping, you can verify whether a previous route exists on the stack:

```dart
if (Navigator.canPop(context)) {
  Navigator.pop(context);
}
```

---

## 📋 Important Properties & Concepts Comparison

| Concept | What It Is | Responsibility in Flutter |
| :--- | :--- | :--- |
| **`Navigator`** | Widget | The manager coordinating the stack of screens. Exposes `push()`, `pop()`, etc. |
| **`Route`** | Abstract Class | Represents an individual page, dialog, or modal layer managed by a `Navigator`. |
| **`MaterialPageRoute`** | Concrete Class | A route that applies Material design transition animations (slide-up / fade on Android, slide-left on iOS). |
| **`CupertinoPageRoute`** | Concrete Class | An iOS-styled route with smooth horizontal slide and interactive edge swipe-to-back gesture. |
| **`BuildContext`** | Handle / Token | Used by `Navigator.of(context)` to find the nearest enclosing `NavigatorState` up the tree. |
| **`PopScope`** | Widget | Intercepts and controls system back-button gestures (replaces legacy `WillPopScope`). |

---

## 💻 Complete Runnable Example
The following production-grade application demonstrates a complete multi-screen navigation flow:
1. **Catalog Screen (Home Route)**: Displays a list of programming courses.
2. **Course Detail Screen (Second Route)**: Pushed onto the stack with course data passed into its constructor.
3. **Lesson Player Screen (Third Route)**: Pushed on top of the detail screen, then popped back with user progress feedback.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const NavigationMasterclassApp());
}

class NavigationMasterclassApp extends StatelessWidget {
  const NavigationMasterclassApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Navigation Masterclass',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const CourseCatalogScreen(),
    );
  }
}

// ---------------------------------------------------------------------------
// SCREEN 1: Course Catalog (Root of the Navigation Stack)
// ---------------------------------------------------------------------------
class CourseCatalogScreen extends StatelessWidget {
  const CourseCatalogScreen({super.key});

  final List<Map<String, String>> courses = const [
    {
      'title': 'Flutter & Dart: The Complete Guide',
      'instructor': 'Senior Flutter Architect',
      'duration': '42 Hours',
      'lessons': '180 Lessons',
    },
    {
      'title': 'State Management Masterclass (BLoC & Riverpod)',
      'instructor': 'Google Developer Expert',
      'duration': '28 Hours',
      'lessons': '95 Lessons',
    },
    {
      'title': 'Flutter Clean Architecture & Testing',
      'instructor': 'Lead Mobile Engineer',
      'duration': '35 Hours',
      'lessons': '120 Lessons',
    },
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Course Catalog'),
        centerTitle: true,
      ),
      body: SafeArea(
        child: ListView.separated(
          padding: const EdgeInsets.all(16.0),
          itemCount: courses.length,
          separatorBuilder: (_, __) => const SizedBox(height: 12),
          itemBuilder: (context, index) {
            final course = courses[index];
            return Card(
              elevation: 1,
              shape: RoundedRectangleBorder(
                borderRadius: BorderRadius.circular(12),
                side: BorderSide(color: Colors.grey.shade200),
              ),
              child: ListTile(
                contentPadding: const EdgeInsets.all(16.0),
                leading: CircleAvatar(
                  backgroundColor: Colors.deepPurple.shade100,
                  child: Text(
                    '${index + 1}',
                    style: const TextStyle(fontWeight: FontWeight.bold),
                  ),
                ),
                title: Text(
                  course['title']!,
                  style: const TextStyle(fontWeight: FontWeight.bold),
                ),
                subtitle: Text('${course['instructor']} • ${course['duration']}'),
                trailing: const Icon(Icons.arrow_forward_ios, size: 16),
                onTap: () {
                  // PUSH Screen 2 onto the navigation stack
                  Navigator.push(
                    context,
                    MaterialPageRoute(
                      builder: (context) => CourseDetailScreen(
                        courseTitle: course['title']!,
                        instructor: course['instructor']!,
                        totalLessons: course['lessons']!,
                      ),
                    ),
                  );
                },
              ),
            );
          },
        ),
      ),
    );
  }
}

// ---------------------------------------------------------------------------
// SCREEN 2: Course Detail Screen (Second Layer on Stack)
// ---------------------------------------------------------------------------
class CourseDetailScreen extends StatelessWidget {
  final String courseTitle;
  final String instructor;
  final String totalLessons;

  const CourseDetailScreen({
    super.key,
    required this.courseTitle,
    required this.instructor,
    required this.totalLessons,
  });

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Course Details'),
      ),
      body: SafeArea(
        child: Padding(
          padding: const EdgeInsets.all(24.0),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: [
              Container(
                height: 160,
                decoration: BoxDecoration(
                  color: Colors.deepPurple.shade50,
                  borderRadius: BorderRadius.circular(16),
                  border: Border.all(color: Colors.deepPurple.shade200),
                ),
                child: const Center(
                  child: Icon(Icons.play_circle_filled, size: 72, color: Colors.deepPurple),
                ),
              ),
              const SizedBox(height: 20),
              Text(
                courseTitle,
                style: const TextStyle(fontSize: 22, fontWeight: FontWeight.bold),
              ),
              const SizedBox(height: 8),
              Text(
                'Instructor: $instructor',
                style: TextStyle(color: Colors.grey.shade700, fontSize: 16),
              ),
              Text(
                'Curriculum: $totalLessons',
                style: TextStyle(color: Colors.grey.shade700, fontSize: 16),
              ),
              const Spacer(),

              // Button to push Screen 3
              FilledButton.icon(
                onPressed: () {
                  Navigator.push(
                    context,
                    MaterialPageRoute(
                      builder: (context) => LessonPlayerScreen(
                        lessonTitle: 'Lesson 1: Introduction & Architecture',
                      ),
                    ),
                  );
                },
                icon: const Icon(Icons.play_arrow),
                label: const Text('Start First Lesson', style: TextStyle(fontSize: 16)),
                style: FilledButton.styleFrom(
                  padding: const EdgeInsets.symmetric(vertical: 16),
                ),
              ),
              const SizedBox(height: 12),

              // Button to pop back to Screen 1 manually
              OutlinedButton(
                onPressed: () {
                  Navigator.pop(context);
                },
                style: OutlinedButton.styleFrom(
                  padding: const EdgeInsets.symmetric(vertical: 16),
                ),
                child: const Text('Back to Course Catalog'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

// ---------------------------------------------------------------------------
// SCREEN 3: Lesson Player Screen (Topmost Layer on Stack)
// ---------------------------------------------------------------------------
class LessonPlayerScreen extends StatelessWidget {
  final String lessonTitle;

  const LessonPlayerScreen({
    super.key,
    required this.lessonTitle,
  });

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Now Playing'),
      ),
      body: SafeArea(
        child: Padding(
          padding: const EdgeInsets.all(24.0),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: [
              const Icon(Icons.video_library, size: 80, color: Colors.deepPurple),
              const SizedBox(height: 20),
              Text(
                lessonTitle,
                textAlign: TextAlign.center,
                style: const TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
              ),
              const SizedBox(height: 12),
              const Text(
                'Video playback active. You are at the top of the 3-layer navigation stack.',
                textAlign: TextAlign.center,
                style: TextStyle(color: Colors.grey),
              ),
              const SizedBox(height: 40),

              // Button to pop back to Screen 2
              FilledButton.icon(
                onPressed: () {
                  Navigator.pop(context);
                },
                icon: const Icon(Icons.check_circle),
                label: const Text('Complete & Return to Details'),
                style: FilledButton.styleFrom(
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
- `Navigator.push(context, MaterialPageRoute(...))`:
  - **What it does**: Tells the nearest `Navigator` to push a new route onto the stack.
  - **Why `context` is required**: `Navigator.push` internally calls `Navigator.of(context)`. Flutter uses this `BuildContext` to search up the element tree until it locates the `NavigatorState` injected by `MaterialApp`.
  - **What happens if removed**: The app remains static on the current screen with no transition.
- `MaterialPageRoute(builder: (context) => CourseDetailScreen(...))`:
  - **What it does**: Wraps the destination widget in a platform-appropriate route.
  - **Internal behavior**: On Android, it slides in vertically from the bottom while fading in. On iOS, it slides in from the right edge with a subtle parallax shadow behind the retiring screen.
- `Navigator.pop(context)`:
  - **What it does**: Removes the topmost route from the navigation stack and disposes of its associated route state.
  - **Internal behavior**: The previous screen (`CourseDetailScreen` or `CourseCatalogScreen`) is immediately brought back into foreground focus.
- `appBar: AppBar(...)` on destination screens:
  - **What it does**: Flutter automatically inspects the navigation stack. If `Navigator.canPop(context)` is `true`, `AppBar` automatically renders a leading back arrow button `←` that calls `Navigator.maybePop(context)` when tapped!

---

## 📱 ASCII Phone Screen Preview

```
[ Screen 1: Catalog ]         [ Screen 2: Detail ]         [ Screen 3: Player ]
+---------------------+      +---------------------+      +---------------------+
|   Course Catalog    |      | [←] Course Details  |      | [←]  Now Playing    |
+---------------------+      +---------------------+      +---------------------+
|                     | PUSH |                     | PUSH |                     |
| 1. Flutter & Dart > |----->| +-----------------+ |----->|     [ 📹 VIDEO ]    |
|    42 Hours         |      | |    [ ▶ PLAY ]   | |      |                     |
|                     |      | +-----------------+ |      | Lesson 1: Intro...  |
| 2. State Mgmt     > |      |                     |      |                     |
|    28 Hours         |      | Flutter & Dart Guide|      | +-----------------+ |
|                     |      |                     |      | | [✓] Complete &  | |
| 3. Clean Arch     > |      | [ Start Lesson  ]   |      | |     Return      | |
|    35 Hours         |      | [ Back to Catalog ] |      | +-----------------+ |
+---------------------+      +---------------------+      +---------------------+
    (Base of Stack)              (Layer 2 on Stack)           (Top of Stack)
           ▲                            │                            │
           │                            │ POP                        │ POP
           └────────────────────────────┴────────────────────────────┘
```

---

## 🌳 Widget Tree
```
MaterialApp
 └── Navigator [State: NavigatorState, _history: [Route 1, Route 2, Route 3]]
      └── Overlay
           ├── OverlayEntry (Route 1: CourseCatalogScreen) [Inactive in background]
           │    └── Scaffold
           │         ├── AppBar
           │         └── ListView.separated -> Card -> ListTile
           ├── OverlayEntry (Route 2: CourseDetailScreen) [Inactive in background]
           │    └── Scaffold
           │         ├── AppBar (Auto-renders back button)
           │         └── Column (Details & Buttons)
           └── OverlayEntry (Route 3: LessonPlayerScreen) [ACTIVE ON TOP]
                └── Scaffold
                     ├── AppBar (Auto-renders back button)
                     └── Column (Player Controls)
```

---

## ⚙ Flutter Execution Flow
1. **App Launch**: `MaterialApp` mounts its root `Navigator`. It creates `Route 1` wrapping `CourseCatalogScreen` and pushes it onto the empty stack.
2. **User Tap**: The user taps the first course card in `CourseCatalogScreen`.
3. **Push Invocation**: `Navigator.push()` is executed with `MaterialPageRoute(builder: (ctx) => CourseDetailScreen(...))`.
4. **Transition**:
   - `NavigatorState` creates `Route 2`.
   - The route mounts an `OverlayEntry` containing `CourseDetailScreen`.
   - The entrance animation plays (e.g. 300ms slide-in).
   - `Route 1` is marked as inactive and stops receiving user touches.
5. **Next Push**: User taps "Start First Lesson", pushing `Route 3` (`LessonPlayerScreen`) onto the stack.
6. **Pop Action**:
   - User taps the back arrow or the "Complete" button on `LessonPlayerScreen`.
   - `Navigator.pop(context)` executes.
   - `Route 3` plays its exit transition (sliding out).
   - `Route 3` is unmounted and disposed from memory.
   - `Route 2` becomes the active top route again.

---

## ⚠ Common Beginner Mistakes

### 1. Using a `BuildContext` Above the `Navigator`
```dart
// ❌ CRASH: Navigator.of() called with a context that does not contain a Navigator!
void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        body: Center(
          child: ElevatedButton(
            onPressed: () {
              // ERROR: This 'context' belongs to MyApp, which is ABOVE MaterialApp!
              Navigator.push(context, MaterialPageRoute(...)); 
            },
            child: const Text('Go'),
          ),
        ),
      ),
    );
  }
}

// ✅ GOOD: Extract home screen into its own Widget, or wrap in a Builder!
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      home: HomeScreen(), // HomeScreen's context is BELOW MaterialApp!
    );
  }
}
```

### 2. Popping the Root Screen Resulting in a Black Screen
```dart
// ❌ BAD: Blindly calling pop on the very first screen
Navigator.pop(context);
// If only 1 screen is on the stack, the app pops into an empty, black, unrendered void!

// ✅ GOOD: Always check canPop() before popping programmatically
if (Navigator.canPop(context)) {
  Navigator.pop(context);
}
```

### 3. Infinite Screen Stacking (Memory Leak)
```dart
// ❌ BAD: Pushing Home from Profile, then pushing Profile from Home repeatedly!
// Stack: Home -> Profile -> Home -> Profile -> Home -> Profile (Consumes massive RAM!)
Navigator.push(context, MaterialPageRoute(builder: (_) => const HomeScreen()));

// ✅ GOOD: Use pushReplacement or pop back to the existing screen
Navigator.pop(context);
```

---

## 🚀 Professional Usage
- **E-Commerce Apps (e.g., Amazon, Shopify)**: Browsing uses deep stack navigation (`Home -> Category -> Subcategory -> Product -> Reviews`). The user can pop back through each level step-by-step.
- **Media Apps (e.g., Spotify, YouTube)**: Tapping a song pushes the player view or expands a mini-player, while tapping back collapses it back down to the playlist.
- **Authentication Flows (e.g., Uber, Twitter)**: Once login or signup succeeds, apps discard the entire onboarding stack using `pushAndRemoveUntil`, ensuring the user cannot accidentally navigate back to the login screen.

---

## 💡 Best Practices
1. **Pass data through constructors for simple routes**: Directly passing arguments into screen constructors (e.g. `CourseDetailScreen(course: c)`) provides compile-time null-safety and autocompletion.
2. **Never pop the root route**: Guard custom back buttons with `if (Navigator.canPop(context))`.
3. **Leverage the automatic AppBar back button**: Do not manually add leading `IconButton`s with `Navigator.pop` unless you are overriding the default icon or behavior; `AppBar` handles this automatically.
4. **Prefer `MaterialPageRoute` for cross-platform apps**: Automatically adapts transition animations to match the platform (Material on Android, Cupertino on iOS).

---

## 🧪 Try It Yourself
1. Replace `MaterialPageRoute` with `CupertinoPageRoute` (imported from `package:flutter/cupertino.dart`) and observe the smooth iOS slide-left transition and edge-swipe gesture.
2. Add a `FloatingActionButton` on `CourseCatalogScreen` that displays the current number of courses.
3. Test popping `LessonPlayerScreen` directly back to `CourseCatalogScreen` by calling `Navigator.pop(context)` twice.

---

## 🎯 Challenge
Build a 3-step checkout wizard:
- Screen 1: Cart Items Summary (Button: *"Proceed to Shipping"*).
- Screen 2: Shipping Address Input (Button: *"Proceed to Payment"*).
- Screen 3: Payment Confirmation (Button: *"Place Order"*).
- When "Place Order" is clicked on Screen 3, display a success dialog and return all the way back to Screen 1.

---

## ❓ Practice Questions

### Q1. What data structure does Flutter's `Navigator` use to manage screens?
- A) Queue (First-In, First-Out)
- B) Stack (Last-In, First-Out)
- C) Binary Search Tree
- D) Hash Map
*Answer: B* — Screens are pushed onto the top of the stack and popped off in reverse order.

### Q2. What error occurs if `Navigator.of(context)` is called using a `BuildContext` that is an ancestor of `MaterialApp`?
- A) A compile-time syntax error
- B) `Navigator operation requested with a context that does not include a Navigator.`
- C) The app restarts automatically
- D) The screen turns transparent
*Answer: B* — `Navigator.of()` searches *up* the tree. If the context is above `MaterialApp`, no `Navigator` ancestor exists.

### Q3. What is the role of `MaterialPageRoute` in Flutter?
- A) It defines the SQL database route
- B) It wraps a destination widget in a modal route that provides platform-specific transition animations
- C) It connects the app to Google Maps
- D) It prevents memory leaks in controllers
*Answer: B* — It provides native entrance and exit transition animations based on the host OS.

### Q4. What does `Navigator.canPop(context)` return?
- A) `true` if there are 2 or more routes on the stack and popping is safe; `false` if currently at the root route
- B) `true` only if the keyboard is closed
- C) `true` if the device has a physical back button
- D) The number of routes on the stack
*Answer: A* — It prevents popping the root route, which would leave the user looking at a blank black screen.

### Q5. Why does Flutter's `AppBar` automatically display a back arrow `←` on secondary screens?
- A) It is hardcoded on every AppBar
- B) The AppBar checks `Navigator.canPop(context)`. If `true`, it automatically injects a `BackButton` leading widget
- C) It requires the user to specify `hasBackArrow: true`
- D) Only on Android devices
*Answer: B* — `AppBar` inspects the enclosing `Navigator` and renders a back button whenever navigation history exists.

### Q6. What is the difference between `Navigator.push` and `Navigator.pop`?
*Answer*: `Navigator.push` adds a new route on top of the navigation stack, making it the active visible screen. `Navigator.pop` removes the topmost route from the stack, revealing the previous screen underneath.

### Q7. How does `CupertinoPageRoute` differ visually from `MaterialPageRoute`?
*Answer*: `CupertinoPageRoute` strictly mimics iOS navigation: it slides horizontally from right to left, includes a subtle drop-shadow over the previous route, and enables the interactive swipe-to-back gesture from the left screen edge.

### Q8. Output Prediction: What happens on screen when `Navigator.pop(context)` is executed while on Screen 2 in a 2-screen stack?
*Answer*: Screen 2 plays its exit transition animation (slides down/out) and unmounts, revealing Screen 1 in its exact previous state without reloading.

### Q9. Can you pass data from a parent screen to a child screen using standard constructor arguments during `Navigator.push`?
*Answer*: Yes, constructor argument passing (`MaterialPageRoute(builder: (context) => DetailScreen(data: myData))`) is the recommended, type-safe approach in Flutter.

### Q10. Write a minimal one-line statement that safely pops the current screen only if a previous screen exists.
*Answer*:
```dart
if (Navigator.canPop(context)) Navigator.pop(context);
```

---

## 📝 Summary
- Flutter screens are widgets managed on a **LIFO Stack** by the `Navigator` widget.
- Pushing a route (`Navigator.push`) places a new screen on top; popping a route (`Navigator.pop`) removes the top screen.
- `MaterialPageRoute` and `CupertinoPageRoute` provide native platform transitions and gestures.
- `MaterialApp` automatically injects a root `Navigator` into your widget tree.
- Always ensure `BuildContext` is below `MaterialApp` when invoking navigation commands.
