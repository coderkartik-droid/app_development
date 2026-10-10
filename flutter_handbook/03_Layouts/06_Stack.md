# Stack

## 📖 Introduction

Up to this point, all the layout widgets you have learned—`Row`, `Column`, and `Wrap`—place elements **next to each other**, either horizontally or vertically.

However, modern UI design frequently requires something completely different: **overlapping elements along the Z-axis (depth)**.

Think about common interface patterns:
* An unread notification badge (`"3"`) hovering on the top-right corner of a bell icon.
* A semi-transparent gradient and text floating on top of a background hero image.
* A floating "Online" green status dot resting on the bottom-right corner of a user's circular profile picture.
* A full-screen dark loading overlay with a spinning progress indicator blocking interactions.

None of these can be built with `Row` or `Column` because elements must draw **on top of one another**.

This is where the **`Stack`** widget shines.

The `Stack` widget allows you to layer multiple child widgets one over another along the Z-axis. Children are painted in the order they are listed: the first child sits at the very bottom, and each subsequent child paints directly on top of the previous one.

---

## 🤔 Why Do We Need It?

Without `Stack`, 3D visual layering on a 2D screen is impossible:

1. **Z-Axis Depth:** Mobile apps are not flat posters; they have visual depth. Badges, floating buttons, gradient overlays, and tooltips need to sit above underlying content.
2. **Absolute Coordinate Pinning:** While `Row` and `Column` calculate fluid sibling relationships, you often need to say: *"Pin this close button exactly 16 pixels from the top-right corner of this banner, regardless of what is happening inside the banner."*
3. **Overlays and Modals:** When an app needs to freeze the UI, show a loading spinner, or darken the background for a bottom sheet, `Stack` lets you place an overlay directly over the page content.

---

## 🌍 Real-Life Analogy

Think of **layers in Photoshop**, **a deck of playing cards**, or **putting toppings on a pizza**:

```text
       Top Layer:     [ Notification Badge "5" ]
                            ▲
                            │ Painted on top
                            ▼
       Bottom Layer:  [ Bell Icon Container    ]
```

* When you make a pizza, you start with the dough base (Layer 1).
* You spread tomato sauce over it (Layer 2).
* You sprinkle cheese over the sauce (Layer 3).
* Finally, you add pepperoni slices on top (Layer 4).
* In a `Stack`, the first widget in the list is the dough base at the bottom, and the last widget is the pepperoni on the very top.

---

## 🧠 Internal Working

How does Flutter’s layout engine execute a `Stack`?

```text
Parent (e.g., Container with width: 300px, height: 200px)
       │
       ▼
    [Stack] (alignment: Alignment.center)
       │
       ├─► 1. Classifies Children into Two Groups:
       │      • Non-Positioned Children (normal widgets like Container, Image, Text)
       │      • Positioned Children (wrapped in Positioned(top: ..., right: ...))
       │
       ├─► 2. Sizes Non-Positioned Children:
       │      Non-positioned children size themselves according to parent constraints.
       │      The Stack's overall natural size is determined by its largest non-positioned child!
       │
       ├─► 3. Positions and Sizes Positioned Children:
       │      Takes coordinates (top, left, right, bottom) and pins them relative to Stack edges.
       │
       ├─► 4. Paints in List Order:
       │      Child 0 (Base) painted first
       │      Child 1 painted on top of Child 0
       │      Child 2 painted on top of Child 1
       │
       ▼
Final layered composite rendered on screen
```

### Painting Order (The Z-Index in Flutter)
Flutter does not have a CSS `z-index` property like `z-index: 99`. 

In Flutter, **Z-index is strictly determined by list order**:

```dart
Stack(
  children: [
    WidgetA(), // Painted at the bottom (Z = 0)
    WidgetB(), // Painted over WidgetA  (Z = 1)
    WidgetC(), // Painted on the very top (Z = 2)
  ],
)
```

If `WidgetC` overlaps `WidgetA`, `WidgetC` covers `WidgetA`. To bring `WidgetA` to the front, simply move it to the end of the `children` list.

---

## ⚙ Syntax

The constructor of a `Stack`:

```dart
Stack({
  Key? key,
  AlignmentGeometry alignment = AlignmentDirectional.topStart,
  TextDirection? textDirection,
  StackFit fit = StackFit.loose,
  Clip clipBehavior = Clip.hardEdge,
  List<Widget> children = const <Widget>[],
})
```

### Parameter Breakdown:
* **`children`**: The list of widgets layered from bottom to top.
* **`alignment`**: How non-positioned (or partially positioned) children are aligned within the Stack boundary (defaults to `topStart`).
* **`fit`**: Controls how non-positioned children are sized:
  * `StackFit.loose`: Non-positioned children can be any size up to the Stack's max constraints.
  * `StackFit.expand`: Non-positioned children are forced to stretch and match the Stack's full width and height.
  * `StackFit.passthrough`: Passes parent constraints directly to children unchanged.
* **`clipBehavior`**: Decides whether children that overflow the Stack’s bounds should be clipped (`Clip.hardEdge`) or allowed to draw outside (`Clip.none`).

---

## 📋 Important Properties

| Property | Type | Default Value | Purpose |
|---|---|---|---|
| `children` | `List<Widget>` | `const []` | The stack of widgets layered along the Z-axis. |
| `alignment` | `AlignmentGeometry` | `topStart` | Default position for children not wrapped in `Positioned`. |
| `fit` | `StackFit` | `StackFit.loose` | How non-positioned children are constrained (`loose` or `expand`). |
| `clipBehavior` | `Clip` | `Clip.hardEdge` | Whether to cut off children positioned outside the Stack frame. |

---

## 💻 Complete Runnable Example

Below is a complete, runnable Flutter app showcasing three practical production patterns: an active online user avatar, an e-commerce hero card with a gradient overlay, and a notification bell with an unread badge.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const StackDemoApp());
}

class StackDemoApp extends StatelessWidget {
  const StackDemoApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: StackDemoScreen(),
    );
  }
}

class StackDemoScreen extends StatelessWidget {
  const StackDemoScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFFF1F5F9),
      appBar: AppBar(
        title: const Text(
          'Stack Widget Deep Dive',
          style: TextStyle(fontWeight: FontWeight.bold),
        ),
        backgroundColor: Colors.blueGrey.shade900,
        foregroundColor: Colors.white,
        elevation: 2,
        actions: [
          // Pattern 1: Notification Bell with Badge
          Padding(
            padding: const EdgeInsets.only(right: 16.0),
            child: Center(
              child: Stack(
                clipBehavior: Clip.none,
                children: [
                  IconButton(
                    icon: const Icon(Icons.notifications, size: 28),
                    onPressed: () {},
                  ),
                  Positioned(
                    right: 6,
                    top: 6,
                    child: Container(
                      padding: const EdgeInsets.all(4),
                      decoration: const BoxDecoration(
                        color: Colors.red,
                        shape: BoxShape.circle,
                      ),
                      constraints: const BoxConstraints(minWidth: 18, minHeight: 18),
                      child: const Text(
                        '3',
                        style: TextStyle(
                          color: Colors.white,
                          fontSize: 10,
                          fontWeight: FontWeight.bold,
                        ),
                        textAlign: TextAlign.center,
                      ),
                    ),
                  ),
                ],
              ),
            ),
          ),
        ],
      ),
      body: SafeArea(
        child: SingleChildScrollView(
          padding: const EdgeInsets.all(16.0),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              _buildSectionHeader('1. User Avatar with Online Status Dot'),
              Container(
                padding: const EdgeInsets.all(16),
                decoration: BoxDecoration(
                  color: Colors.white,
                  borderRadius: BorderRadius.circular(12),
                  border: Border.all(color: Colors.grey.shade300),
                ),
                child: Row(
                  children: [
                    // Stack for Profile Avatar + Green Online Dot
                    Stack(
                      children: [
                        const CircleAvatar(
                          radius: 32,
                          backgroundColor: Colors.blueGrey,
                          child: Icon(Icons.person, size: 40, color: Colors.white),
                        ),
                        Positioned(
                          bottom: 2,
                          right: 2,
                          child: Container(
                            width: 16,
                            height: 16,
                            decoration: BoxDecoration(
                              color: Colors.greenAccent.shade700,
                              shape: BoxShape.circle,
                              border: Border.all(color: Colors.white, width: 2.5),
                            ),
                          ),
                        ),
                      ],
                    ),
                    const SizedBox(width: 16),
                    Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        const Text(
                          'Alex Morgan',
                          style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
                        ),
                        const SizedBox(height: 4),
                        Text(
                          'Active Now',
                          style: TextStyle(color: Colors.green.shade700, fontSize: 13),
                        ),
                      ],
                    ),
                  ],
                ),
              ),

              const SizedBox(height: 28),

              _buildSectionHeader('2. Hero Product Banner with Gradient & Badges'),
              // Stack for Banner Image + Dark Gradient Overlay + Text + Discount Tag
              ClipRRect(
                borderRadius: BorderRadius.circular(16),
                child: SizedBox(
                  height: 200,
                  width: double.infinity,
                  child: Stack(
                    fit: StackFit.expand, // Forces background to fill the SizedBox
                    children: [
                      // Layer 1: Background Mock Image/Container
                      Container(
                        color: Colors.indigo.shade800,
                        child: const Icon(
                          Icons.laptop_mac,
                          size: 100,
                          color: Colors.white24,
                        ),
                      ),

                      // Layer 2: Subtle Dark Gradient Overlay for readability
                      Container(
                        decoration: BoxDecoration(
                          gradient: LinearGradient(
                            begin: Alignment.topCenter,
                            end: Alignment.bottomCenter,
                            colors: [
                              Colors.transparent,
                              Colors.black.withValues(alpha: 0.8),
                            ],
                          ),
                        ),
                      ),

                      // Layer 3: Pinned Discount Badge (Top Right)
                      Positioned(
                        top: 14,
                        right: 14,
                        child: Container(
                          padding: const EdgeInsets.symmetric(horizontal: 10, vertical: 4),
                          decoration: BoxDecoration(
                            color: Colors.amber,
                            borderRadius: BorderRadius.circular(20),
                          ),
                          child: const Text(
                            '🔥 40% OFF',
                            style: TextStyle(
                              fontWeight: FontWeight.bold,
                              fontSize: 12,
                              color: Colors.black,
                            ),
                          ),
                        ),
                      ),

                      // Layer 4: Title & Price pinned to Bottom Left
                      const Positioned(
                        left: 16,
                        bottom: 16,
                        right: 16,
                        child: Column(
                          crossAxisAlignment: CrossAxisAlignment.start,
                          children: [
                            Text(
                              'MacBook Pro M3 Max',
                              style: TextStyle(
                                color: Colors.white,
                                fontSize: 20,
                                fontWeight: FontWeight.bold,
                              ),
                            ),
                            SizedBox(height: 4),
                            Text(
                              'Starting at \$2,499 • Limited stock',
                              style: TextStyle(color: Colors.white70, fontSize: 13),
                            ),
                          ],
                        ),
                      ),
                    ],
                  ),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }

  static Widget _buildSectionHeader(String title) {
    return Padding(
      padding: const EdgeInsets.only(bottom: 8.0),
      child: Text(
        title,
        style: const TextStyle(fontSize: 15, fontWeight: FontWeight.bold),
      ),
    );
  }
}
```

---

## 🔍 Example Explanation

### 1. `Stack(clipBehavior: Clip.none, children: [...])` in Notification Bell
* **What it does:** Layers a small red circular badge on top of the bell `IconButton`.
* **Why it exists:** Notification numbers must sit partially over the shoulder of the bell icon without displacing the bell in the AppBar.
* **What Flutter does internally:** Measures the `IconButton` (48x48), then positions the badge at `right: 6, top: 6`. `clipBehavior: Clip.none` ensures that if the badge spills slightly beyond the icon's bounding box, it does not get chopped off.
* **What happens if removed:** If you use a `Row`, the badge appears next to the bell, doubling the AppBar button's width and misaligning the interface.

### 2. `Stack` on Avatar + Online Status Dot
* **What it does:** Positions a small 16x16 green dot with a 2.5px white border at the bottom-right corner of a 64x64 circular avatar.
* **Why it exists:** Creates the universal "Active Now" messaging indicator.
* **What Flutter does internally:** Sets the Stack size based on the `CircleAvatar` (radius 32 = 64px diameter). The `Positioned` widget then places the green dot at `bottom: 2, right: 2`.
* **What happens if removed:** Without `Stack`, placing an indicator at the exact corner of a circular avatar is nearly impossible.

### 3. `Stack(fit: StackFit.expand, children: [...])` in Hero Banner
* **What it does:** Stretches the dark background container and gradient overlay to completely cover the 200px-high banner card.
* **Why it exists:** Backgrounds must fill the card, while text and badges float over them.
* **What Flutter does internally:** Forces `minWidth = maxWidth` and `minHeight = maxHeight` on non-positioned children so they fill every pixel of the `SizedBox`.
* **What happens if removed:** The gradient container might shrink to zero height if it has no child, exposing harsh background contrasts.

---

## 📱 Phone Screen Preview

```text
+------------------------------------------+
| Stack Widget Deep Dive              (🔔³) |
+------------------------------------------+
|                                          |
| 1. User Avatar with Online Status Dot    |
| +--------------------------------------+ |
| |  ( 👤 )  Alex Morgan                 | |
| |    ●     Active Now                  | |
| +--------------------------------------+ |
|                                          |
| 2. Hero Product Banner with Gradient     |
| +--------------------------------------+ |
| | [💻 Icon]               [🔥 40% OFF] | |
| |                                      | |
| | ----------------- (Gradient) ------- | |
| | MacBook Pro M3 Max                   | |
| | Starting at $2,499 • Limited stock   | |
| +--------------------------------------+ |
+------------------------------------------+
```

### What the user sees:
* **Top Right:** A bell icon with a red circle badge containing `"3"` sitting directly on its shoulder.
* **Card 1:** A user profile picture with a crisp green online indicator dot overlapping its bottom-right rim.
* **Card 2:** A rich product hero banner featuring a computer graphic, a vibrant orange discount badge in the top right, and bold white text floating legibly over a bottom gradient shadow.

---

## 🌳 Widget Tree

```text
StackDemoApp
└── MaterialApp
    └── StackDemoScreen
        └── Scaffold
            ├── AppBar
            │   └── actions: [ Padding -> Center -> Stack ]
            │       ├── IconButton (Icons.notifications)
            │       └── Positioned (top: 6, right: 6)
            │           └── Container (Red badge -> Text "3")
            └── SafeArea
                └── SingleChildScrollView
                    └── Column
                        ├── Container
                        │   └── Row
                        │       ├── Stack
                        │       │   ├── CircleAvatar (Profile picture)
                        │       │   └── Positioned (bottom: 2, right: 2)
                        │       │       └── Container (Green dot)
                        │       └── Column (Alex Morgan / Active Now)
                        └── ClipRRect
                            └── SizedBox (height: 200)
                                └── Stack (fit: StackFit.expand)
                                    ├── Container (Indigo base)
                                    ├── Container (LinearGradient overlay)
                                    ├── Positioned (top: 14, right: 14)
                                    │   └── Container (Discount badge)
                                    └── Positioned (left: 16, bottom: 16)
                                        └── Column (Title & Price)
```

---

## ⚙ Flutter Execution Flow

```text
1. Stack evaluates non-positioned children first (CircleAvatar)
   │
2. CircleAvatar calculates its dimensions: width = 64px, height = 64px
   │
3. Stack sets its own dimensions to 64px x 64px
   │
4. Stack processes Positioned child (Green dot):
   - Computes bottom offset: 64px - 16px - 2px = 46px Y-coordinate
   - Computes right offset:  64px - 16px - 2px = 46px X-coordinate
   │
5. Paint order executed:
   - CircleAvatar painted first onto canvas
   - Green dot painted over the avatar's bottom-right pixels
   │
6. Canvas composited and pushed to the display
```

---

## ⚠ Common Beginner Mistakes

### Mistake 1: Placing all children inside `Positioned` widgets

**Wrong Code:**
```dart
// ❌ COLLAPSED STACK: The Stack shrinks to zero because it has NO non-positioned child!
Stack(
  children: [
    Positioned(
      top: 10,
      left: 10,
      child: Text('Where am I?'),
    ),
  ],
)
```

**Why it is wrong:**
A `Stack` sizes itself based on its **non-positioned children**. If every child in the `Stack` is wrapped in `Positioned`, the `Stack` has no intrinsic size reference and collapses to zero width and height (unless forced by an explicit parent like `SizedBox` or `Container`).

**Correct Code:**
```dart
// ✅ CORRECT: Either provide a non-positioned child or give the Stack explicit constraints
SizedBox(
  width: 200,
  height: 100,
  child: Stack(
    children: const [
      Positioned(
        top: 10,
        left: 10,
        child: Text('Properly Visible!'),
      ),
    ],
  ),
)
```

---

### Mistake 2: Getting the painting order backwards

**Wrong Code:**
```dart
// ❌ INVISIBLE TEXT: The background container paints on TOP of the text!
Stack(
  children: [
    Text('Important Header Text'),
    Container(color: Colors.blue, width: 200, height: 200),
  ],
)
```

**Why it is wrong:**
In Flutter, the last child in the list paints on the very top. The opaque blue `Container` covers the `Text`, making it completely invisible.

**Correct Code:**
```dart
// ✅ CORRECT: Background at index 0, foreground text on top
Stack(
  children: [
    Container(color: Colors.blue, width: 200, height: 200),
    const Text('Important Header Text', style: TextStyle(color: Colors.white)),
  ],
)
```

---

## 🚀 Professional Usage

`Stack` is indispensable in flagship commercial applications:

* **Instagram / WhatsApp (Stories Viewer):**
  Full-screen `Stack`:
  `[Full-screen Story Photo/Video] -> [Top Progress Bars] -> [Top User Info Row] -> [Bottom Reply Input Bar]`.
* **Google Maps / Uber (Map Screen):**
  A full-screen interactive Map at the bottom, with a `Stack` layering `[Floating Search Bar]` at the top and `[Current Location GPS Button]` at the bottom-right.
* **Amazon / Food Delivery Apps (Product Cards):**
  A product thumbnail with `Stack` layering:
  `[Product Image] -> [Favorite Heart Icon (Top Right)] -> [Discount Ribbon (Top Left)] -> [Price Tag (Bottom)]`.
* **Streaming Apps (Netflix / YouTube):**
  Video player `Stack`:
  `[Raw Video Feed] -> [Play/Pause Center Overlay] -> [Bottom Scrub Bar] -> [Top Title & Back Button]`.

---

## 💡 Best Practices

1. **Always Anchor with One Non-Positioned Child:** Keep at least one natural, non-positioned widget (like the background image or base container) so the `Stack` automatically derives a healthy size.
2. **Use `ClipRRect` for Rounded Cards:** If you place an image or container inside a `Stack` that has rounded corners, wrap the `Stack` in a `ClipRRect` to prevent children from leaking outside the curved borders.
3. **Use `clipBehavior: Clip.none` for Overflowing Badges:** When placing notification dots or close buttons that deliberately peek halfway outside the parent card boundary, set `clipBehavior: Clip.none`.
4. **Prefer `StackFit.expand` for Overlays:** When making semi-transparent scrims or loading spinners, use `Stack(fit: StackFit.expand, ...)` inside a bounded container to ensure edge-to-edge coverage.

---

## 🧪 Try It Yourself

1. Copy the runnable example into your editor.
2. In the `actions` notification bell, locate `Positioned(right: 6, top: 6)`.
3. Change `right` to `20` and `top` to `20`.
4. Hot reload the app and observe how the red badge moves into the middle of the bell.
5. In Card 1, change the `alignment` of the green online dot to `bottom: 0, left: 0` and see it shift to the other side.

---

## 🎯 Challenge

Build a **Social Media Video Thumbnail Card**:
* A rectangular container with rounded corners (width: 220, height: 140).
* **Layer 1 (Bottom):** A dark blue gradient container representing the video cover.
* **Layer 2 (Center):** A circular play icon button (`Icons.play_arrow`, white, radius: 24, with a semi-transparent black background).
* **Layer 3 (Top Right):** A bookmark icon button (`Icons.bookmark_border`, white).
* **Layer 4 (Bottom Right):** A tiny dark pill badge displaying the video duration (`"03:45"`).

---

## ❓ Practice Questions

### 1. Multiple Choice Question (MCQ)
How is the painting order (Z-index) determined in a Flutter `Stack`?
* A) By using the `zIndex` integer property on each child
* B) By the chronological order of widgets in the `children` list (first at bottom, last on top)
* C) By the physical height of each child widget
* D) By the alphabetic name of the widget class

*(Correct Answer: B)*

---

### 2. True or False
**Statement:** A `Stack` can only contain widgets wrapped inside a `Positioned` widget.
*(Answer: False. A `Stack` can contain both positioned and non-positioned widgets. In fact, non-positioned widgets are what give the Stack its natural size).*

---

### 3. Output Prediction
If a `Stack` contains `[Container(color: Colors.red), Container(color: Colors.green)]` and both have equal size, which color will be visible on the screen?
*(Answer: Green. The second child is painted directly over the first child, covering it completely).*

---

### 4. Theory
What is the purpose of `clipBehavior: Clip.none` on a `Stack`?
*(Answer: It prevents Flutter from clipping child widgets whose coordinates fall outside the bounding box of the Stack, allowing elements like notification badges to overlap external borders).*

---

### 5. Multiple Choice Question (MCQ)
Which `StackFit` property forces all non-positioned children to stretch and match the Stack's maximum constraints?
* A) `StackFit.loose`
* B) `StackFit.passthrough`
* C) `StackFit.expand`
* D) `StackFit.tight`

*(Correct Answer: C)*

---

### 6. Coding Exercise
Write the code for a `Stack` that places a `CircularProgressIndicator` directly in the center of an image container.
*(Answer:*
```dart
Stack(
  alignment: Alignment.center,
  children: [
    Image.asset('assets/placeholder.png', width: 200, height: 200),
    const CircularProgressIndicator(),
  ],
)
```
*)*

---

### 7. Theory
Why does a `Stack` collapse to zero size if every single child is wrapped in a `Positioned` widget?
*(Answer: Because `Positioned` children do not contribute to the Stack's intrinsic sizing calculations. Without a non-positioned child or an explicit parent constraint, the Stack has no size reference).*

---

### 8. Output Prediction
Where will a child widget appear in a `Stack` if it is NOT wrapped in a `Positioned` widget, assuming default Stack properties?
*(Answer: At the top-left corner (`AlignmentDirectional.topStart`), which is the default `alignment` of a Stack).*

---

### 9. Multiple Choice Question (MCQ)
Which companion widget is used to pinpoint exact pixel offsets (`top`, `bottom`, `left`, `right`) inside a `Stack`?
* A) `Align`
* B) `Padding`
* C) `Positioned`
* D) `Transform`

*(Correct Answer: C)*

---

### 10. True or False
**Statement:** You can place an `Expanded` widget directly inside a `Stack`.
*(Answer: False. `Expanded` only works inside a `Row`, `Column`, or `Flex`. Placing it inside a `Stack` throws an assertion exception).*

---

## 📝 Summary

* **Core Purpose:** `Stack` arranges multiple child widgets in overlapping layers along the Z-axis (depth).
* **Layering Order:** Strictly determined by list order: index `0` is at the very bottom; the final index is painted on the very top.
* **Child Types:** Accepts both **Non-Positioned** children (which determine Stack size) and **Positioned** children (which pin to specific coordinates).
* **Sizing Mode:** `StackFit.loose` allows children to size themselves; `StackFit.expand` forces them to fill the Stack's bounds.
* **Overflow Handling:** Controlled via `clipBehavior` (`Clip.hardEdge` to cut off overflowing edges, or `Clip.none` to let badges hang outside).
* **Top Real-World Use Cases:** Floating badges, online status indicators, image gradient overlays, video player controls, and full-screen loading dialogs.
