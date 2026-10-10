# Positioned

## 📖 Introduction

In the previous chapter, you learned how the `Stack` widget layers children on top of each other. 

By default, any child placed inside a `Stack` drifts towards the top-left corner (`Alignment.topLeft`). But in almost every real-world application, you need to anchor widgets to specific places across the stack:
* A close button (`"X"`) glued exactly **12 pixels from the top** and **12 pixels from the right**.
* A bottom action bar pinned along the **bottom edge**, stretching from the far left to the far right.
* A floating status tag positioned **20 pixels above the bottom edge** and **16 pixels from the left**.

To achieve this level of surgical precision inside a `Stack`, Flutter provides the **`Positioned`** widget.

The `Positioned` widget acts as a digital pushpin. It controls where a child of a `Stack` is positioned and how it is sized by specifying exact distances from the stack’s top, bottom, left, and right borders.

---

## 🤔 Why Do We Need It?

Without `Positioned`, placing widgets at specific coordinates inside a `Stack` would require complicated and fragile hacks:

1. **Pixel-Perfect Coordinate Pinning:** You cannot use margins or paddings reliably when overlapping items. `Positioned` gives you direct, declarative coordinate controls: `top: 10`, `right: 15`.
2. **Dynamic Stretching:** If you specify both `left: 0` and `right: 0`, `Positioned` forces the child to stretch horizontally across the entire width of the Stack, eliminating the need to query `MediaQuery` or hardcode widths.
3. **Dedicated Stack Communication:** Just as `Expanded` only talks to a `Flex` parent, `Positioned` is the only widget built specifically to communicate layout constraints directly to a `RenderStack`.

---

## 🌍 Real-Life Analogy

Think of **pushpins on a cork noticeboard** or **magnets on a refrigerator**:

```text
 ┌────────────────────────────────────────┐
 │ 📌 top: 12, right: 12                  │
 │                                [ Close ]
 │                                        │
 │                                        │
 │ 📌 bottom: 16, left: 16                │
 │ [ User Label ]                         │
 └────────────────────────────────────────┘
```

* You have a large cork bulletin board (the `Stack`).
* You take a photo and pin it onto the board.
* Then, you take a small sticky note and pin it exactly **1 inch from the top edge** and **1 inch from the right edge**.
* Even if you swap out the background photo, your sticky note remains pinned at that exact distance from the board's edge.
* That pushpin is the `Positioned` widget.

---

## 🧠 Internal Working

How does Flutter’s layout engine execute a `Positioned` widget?

```text
Stack (Bounding Box: Width = 300px, Height = 200px)
  │
  ├─► Positioned(top: 20, right: 15, width: 80, height: 40)
  │   │
  │   ├─► Calculates X Coordinate:
  │   │   X = Stack Width (300) - right (15) - width (80) = 205px
  │   │
  │   ├─► Calculates Y Coordinate:
  │   │   Y = top (20px)
  │   │
  │   ├─► Passes Tight Constraints to Child:
  │   │   minWidth = 80px, maxWidth = 80px
  │   │   minHeight = 40px, maxHeight = 40px
  │   │
  │   ▼
  RenderObject paints child at exactly (X: 205, Y: 20) with size 80x40
```

### The Sizing Equation Rule
For each axis (horizontal and vertical), you have three potential values:
* **Horizontal Axis:** `left`, `right`, and `width`
* **Vertical Axis:** `top`, `bottom`, and `height`

You can supply **at most two** of these three parameters on each axis:
* If you provide `left` and `right`: The widget **stretches** to fill the distance between those two edges. `width` is computed automatically.
* If you provide `left` and `width`: The widget is positioned at `left` and sized to `width`. `right` is ignored.
* If you supply **all three** (`left`, `right`, AND `width`), Flutter throws a runtime exception because the layout is mathematically over-constrained!

---

## ⚙ Syntax

The constructor of `Positioned`:

```dart
const Positioned({
  Key? key,
  double? left,
  double? top,
  double? right,
  double? bottom,
  double? width,
  double? height,
  required Widget child,
})
```

### Parameter Breakdown:
* **`child`**: The widget you want to pin and position inside the `Stack`.
* **`left`**: Distance in pixels from the left edge of the `Stack`.
* **`top`**: Distance in pixels from the top edge of the `Stack`.
* **`right`**: Distance in pixels from the right edge of the `Stack`.
* **`bottom`**: Distance in pixels from the bottom edge of the `Stack`.
* **`width`**: Explicit width for the child (cannot be used if both `left` and `right` are set).
* **`height`**: Explicit height for the child (cannot be used if both `top` and `bottom` are set).

### Alternative Constructors:
* **`Positioned.fill`**: Pins `top: 0, right: 0, bottom: 0, left: 0` simultaneously, stretching the child to completely cover the Stack.
* **`Positioned.directional`**: Adapts coordinates to text direction (uses `start` and `end` instead of `left` and `right` for RTL/LTR support).
* **`Positioned.fromRect`**: Positions a child based on a geometric `Rect` object.

---

## 📋 Important Properties

| Property | Type | Default Value | Purpose |
|---|---|---|---|
| `child` | `Widget` | *Required* | The target widget being placed. |
| `top` | `double?` | `null` | Offset from the top boundary of the `Stack`. |
| `bottom` | `double?` | `null` | Offset from the bottom boundary of the `Stack`. |
| `left` | `double?` | `null` | Offset from the left boundary of the `Stack`. |
| `right` | `double?` | `null` | Offset from the right boundary of the `Stack`. |
| `width` | `double?` | `null` | Fixed width constraint for the child. |
| `height` | `double?` | `null` | Fixed height constraint for the child. |

---

## 💻 Complete Runnable Example

Below is a complete, runnable Flutter application demonstrating `Positioned` with real-world scenarios: an e-commerce flash-sale banner, a story card with a close button, and `Positioned.fill` for full-card touch detection.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const PositionedDemoApp());
}

class PositionedDemoApp extends StatelessWidget {
  const PositionedDemoApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: PositionedDemoScreen(),
    );
  }
}

class PositionedDemoScreen extends StatelessWidget {
  const PositionedDemoScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFFF3F4F6),
      appBar: AppBar(
        title: const Text(
          'Positioned Widget Deep Dive',
          style: TextStyle(fontWeight: FontWeight.bold),
        ),
        backgroundColor: Colors.teal.shade800,
        foregroundColor: Colors.white,
        elevation: 2,
      ),
      body: SafeArea(
        child: SingleChildScrollView(
          padding: const EdgeInsets.all(16.0),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              _buildSectionHeader('1. Pinning to Multiple Corners'),
              // Container serving as the Stack boundary
              Container(
                height: 180,
                width: double.infinity,
                decoration: BoxDecoration(
                  color: Colors.white,
                  borderRadius: BorderRadius.circular(16),
                  border: Border.all(color: Colors.grey.shade300),
                ),
                child: Stack(
                  children: [
                    // Corner 1: Top-Left Tag
                    Positioned(
                      top: 12,
                      left: 12,
                      child: Container(
                        padding: const EdgeInsets.symmetric(horizontal: 10, vertical: 4),
                        decoration: BoxDecoration(
                          color: Colors.teal.shade50,
                          borderRadius: BorderRadius.circular(6),
                          border: Border.all(color: Colors.teal),
                        ),
                        child: Text(
                          'FEATURED',
                          style: TextStyle(
                            color: Colors.teal.shade800,
                            fontWeight: FontWeight.bold,
                            fontSize: 11,
                          ),
                        ),
                      ),
                    ),

                    // Corner 2: Top-Right Close Button
                    Positioned(
                      top: 8,
                      right: 8,
                      child: IconButton(
                        icon: const Icon(Icons.close, color: Colors.grey),
                        onPressed: () {},
                      ),
                    ),

                    // Center Content (Unpositioned or Centered)
                    const Center(
                      child: Column(
                        mainAxisSize: MainAxisSize.min,
                        children: [
                          Icon(Icons.workspace_premium, size: 48, color: Colors.amber),
                          SizedBox(height: 6),
                          Text(
                            'Upgrade to Pro Membership',
                            style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
                          ),
                        ],
                      ),
                    ),

                    // Horizontal Stretch: Bottom Bar across left & right
                    Positioned(
                      bottom: 0,
                      left: 0,
                      right: 0, // left: 0 + right: 0 stretches edge-to-edge!
                      child: Container(
                        padding: const EdgeInsets.symmetric(vertical: 8),
                        decoration: BoxDecoration(
                          color: Colors.teal.shade800,
                          borderRadius: const BorderRadius.only(
                            bottomLeft: Radius.circular(15),
                            bottomRight: Radius.circular(15),
                          ),
                        ),
                        alignment: Alignment.center,
                        child: const Text(
                          'Claim 50% Discount Today',
                          style: TextStyle(
                            color: Colors.white,
                            fontSize: 12,
                            fontWeight: FontWeight.bold,
                          ),
                        ),
                      ),
                    ),
                  ],
                ),
              ),

              const SizedBox(height: 28),

              _buildSectionHeader('2. Positioned.fill (Transparent Clickable Layer)'),
              Container(
                height: 120,
                width: double.infinity,
                decoration: BoxDecoration(
                  gradient: LinearGradient(
                    colors: [Colors.purple.shade600, Colors.deepPurple.shade800],
                  ),
                  borderRadius: BorderRadius.circular(14),
                ),
                child: Stack(
                  children: [
                    // Visual Background Graphics
                    Positioned(
                      right: -15,
                      bottom: -15,
                      child: Icon(
                        Icons.card_giftcard,
                        size: 110,
                        color: Colors.white.withValues(alpha: 0.15),
                      ),
                    ),
                    const Positioned(
                      top: 24,
                      left: 20,
                      child: Column(
                        crossAxisAlignment: CrossAxisAlignment.start,
                        children: [
                          Text(
                            'Refer a Friend & Earn ₹500',
                            style: TextStyle(
                              color: Colors.white,
                              fontSize: 18,
                              fontWeight: FontWeight.bold,
                            ),
                          ),
                          SizedBox(height: 6),
                          Text(
                            'Tap anywhere on this card to share link',
                            style: TextStyle(color: Colors.white70, fontSize: 13),
                          ),
                        ],
                      ),
                    ),

                    // Positioned.fill wraps an InkWell to make the entire Stack clickable
                    Positioned.fill(
                      child: Material(
                        color: Colors.transparent,
                        child: InkWell(
                          borderRadius: BorderRadius.circular(14),
                          onTap: () {
                            ScaffoldMessenger.of(context).showSnackBar(
                              const SnackBar(content: Text('Referral link copied!')),
                            );
                          },
                        ),
                      ),
                    ),
                  ],
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

### 1. `Positioned(top: 12, left: 12)` & `Positioned(top: 8, right: 8)`
* **What it does:** Anchors the `"FEATURED"` badge to the top-left corner and the `"X"` close button to the top-right corner.
* **Why it exists:** Provides exact boundary alignment regardless of what is happening in the middle of the card.
* **What Flutter does internally:** Measures the dimensions of the child and places its top-left rendering origin at coordinates `(12, 12)` and `(StackWidth - 8 - buttonWidth, 8)`.
* **What happens if removed:** If the `Positioned` wrappers are removed, both widgets collapse onto each other at `(0, 0)`.

### 2. `Positioned(bottom: 0, left: 0, right: 0)`
* **What it does:** Stretches the green discount bar horizontally across the entire bottom edge of the card.
* **Why it exists:** Specifying both `left: 0` and `right: 0` causes Flutter to calculate `width = StackWidth - left - right`. This eliminates hardcoding widths.
* **What Flutter does internally:** Supplies a tight width constraint equal to the Stack's width, while letting height depend on the container's vertical padding.
* **What happens if removed:** The bottom bar shrinks to only fit the words `"Claim 50% Discount Today"`, leaving ugly empty gaps on both sides.

### 3. `Positioned.fill(child: Material(child: InkWell(...)))`
* **What it does:** Stretches a transparent ripple touch surface over all 120 pixels of the referral card.
* **Why it exists:** In modern apps, cards often have complex visual text and background art, but tapping *anywhere* on the card should trigger the action.
* **What Flutter does internally:** Automatically supplies `top: 0, bottom: 0, left: 0, right: 0`.
* **What happens if removed:** The card becomes a static, non-interactive visual graphic.

---

## 📱 Phone Screen Preview

```text
+------------------------------------------+
| Positioned Widget Deep Dive              |
+------------------------------------------+
|                                          |
| 1. Pinning to Multiple Corners           |
| +--------------------------------------+ |
| | [FEATURED]                       (X) | |
| |                                      | |
| |                 (⭐)                 | |
| |       Upgrade to Pro Membership      | |
| |                                      | |
| | [=== Claim 50% Discount Today =====] | |
| +--------------------------------------+ |
|                                          |
| 2. Positioned.fill (Clickable Layer)     |
| +--------------------------------------+ |
| | Refer a Friend & Earn ₹500           | |
| | Tap anywhere on this card...   (🎁)  | |
| +--------------------------------------+ |
+------------------------------------------+
```

### What the user sees:
* **Top Card:** Cleanly separated corners—badge on the top-left, close icon on the top-right, centered text in the middle, and a full-width banner docked along the bottom.
* **Bottom Card:** A rich purple card with a gift icon peeking out from the bottom-right corner. Tapping anywhere shows a Material ink splash ripple and a SnackBar.

---

## 🌳 Widget Tree

```text
PositionedDemoApp
└── MaterialApp
    └── PositionedDemoScreen
        └── Scaffold
            ├── AppBar
            │   └── Text ("Positioned Widget Deep Dive")
            └── SafeArea
                └── SingleChildScrollView
                    └── Column
                        ├── Container
                        │   └── Stack
                        │       ├── Positioned (top: 12, left: 12) -> Container ("FEATURED")
                        │       ├── Positioned (top: 8, right: 8) -> IconButton (close)
                        │       ├── Center -> Column (Icon + Title)
                        │       └── Positioned (bottom: 0, left: 0, right: 0) -> Container (Discount bar)
                        ├── SizedBox (height: 28)
                        └── Container
                            └── Stack
                                ├── Positioned (right: -15, bottom: -15) -> Icon (gift)
                                ├── Positioned (top: 24, left: 20) -> Column (Referral text)
                                └── Positioned.fill -> Material -> InkWell (Clickable layer)
```

---

## ⚙ Flutter Execution Flow

```text
1. Stack establishes total bounds from parent (e.g. Width: 358px, Height: 180px)
   │
2. Center widget (non-positioned) calculates intrinsic size and aligns in the middle
   │
3. Positioned children evaluate coordinates:
   ├── Tag: Top offset = 12px, Left offset = 12px
   ├── Close Button: Top offset = 8px, Right offset = 8px (X = 358 - 48 - 8 = 302px)
   └── Bottom Bar: Left = 0, Right = 0 -> Width forced to 358px, Bottom = 0
   │
4. Paint phase executes in order:
   - Center widget painted
   - Corner tags painted on top
   - Bottom bar painted across lower boundary
   │
5. Final composite sent to screen
```

---

## ⚠ Common Beginner Mistakes

### Mistake 1: Placing `Positioned` outside a `Stack`

**Wrong Code:**
```dart
// ❌ RUNTIME CRASH: Positioned must be an immediate child of a Stack!
Row(
  children: [
    Positioned(
      left: 10,
      child: Text('Hello'),
    ),
  ],
)
```

**Why it is wrong:**
`Positioned` only communicates with `RenderStack`. Placing it inside a `Row`, `Column`, or `Container` causes a crash:
`Positioned widgets must be placed within a Stack widgets.`

**Correct Code:**
```dart
// ✅ CORRECT: Always place Positioned as an immediate child of Stack
Stack(
  children: const [
    Positioned(
      left: 10,
      child: Text('Hello'),
    ),
  ],
)
```

---

### Mistake 2: Over-constraining with `left`, `right`, AND `width`

**Wrong Code:**
```dart
// ❌ RUNTIME ASSERTION: Cannot supply left, right, and width simultaneously!
Positioned(
  left: 10,
  right: 10,
  width: 150,
  child: Container(color: Colors.red),
)
```

**Why it is wrong:**
If `left` is 10 and `right` is 10 on a 400px screen, the width is mathematically forced to 380px. Providing `width: 150` creates a mathematical contradiction.

**Correct Code:**
```dart
// ✅ CORRECT: Choose either left + right (stretch) OR left + width
Positioned(
  left: 10,
  right: 10, // Stretches automatically
  child: Container(color: Colors.red),
)
```

---

## 🚀 Professional Usage

`Positioned` is used throughout major consumer mobile products:

* **WhatsApp (Audio / Video Call Screen):**
  Full-screen video stream with `Positioned(top: 40, left: 16)` for the caller's name, `Positioned(top: 40, right: 16)` for the switch camera button, and `Positioned(bottom: 30, left: 0, right: 0)` for the red end-call button row.
* **Amazon / E-commerce (Product Image Badges):**
  A product image with `Positioned(top: 8, left: 8)` for the `"Prime"` badge and `Positioned(bottom: 8, right: 8)` for a wishlist heart button.
* **Instagram (Stories Camera):**
  Camera preview with `Positioned(top: 20, right: 20)` for the flash toggle, and `Positioned(bottom: 40, left: 0, right: 0)` centering the circular shutter button.
* **Food Delivery Apps (Active Order Tracker):**
  Live GPS map with `Positioned(bottom: 16, left: 16, right: 16)` holding the driver ETA card docked over the map.

---

## 💡 Best Practices

1. **Use `Positioned.fill` for Tap Surfaces:** Instead of wrapping the whole Stack in an `InkWell` (which can interfere with child gesture detectors), place a `Positioned.fill` containing a transparent `InkWell` as the topmost child.
2. **Use Negative Values for Peeking Elements:** You can pass negative coordinates (e.g. `right: -20, bottom: -20`) combined with `Stack(clipBehavior: Clip.none)` to make decorative circles or icons bleed outside the card edge.
3. **Keep `Positioned` as Direct Children:** Do not place intermediate widgets like `Padding` between `Stack` and `Positioned`. Put the `Padding` *inside* `Positioned`.
4. **Prefer `left: 0, right: 0` Over `MediaQuery`:** To stretch a child across the entire Stack width, use `left: 0, right: 0`. It is faster, cleaner, and automatically respects the Stack's true size.

---

## 🧪 Try It Yourself

1. Copy the runnable example above into your Flutter project.
2. In Card 1, locate the bottom banner `Positioned(bottom: 0, left: 0, right: 0)`.
3. Change `left: 0, right: 0` to:
   * `left: 20, right: 20, bottom: 10`
4. Hot reload the app and see how the bottom bar transforms into a floating rounded pill inside the card!

---

## 🎯 Challenge

Build an **Instagram Story Profile Avatar with Add (+) Badge**:
* A `Stack` containing a 70x70 circular avatar (`CircleAvatar(radius: 35)`).
* Use `Positioned` to place a small blue circle with a white plus icon (`Icons.add`, size 16) at the bottom-right corner.
* Add a 2px white border around the plus circle so it visually separates from the avatar image.

---

## ❓ Practice Questions

### 1. Multiple Choice Question (MCQ)
Which widget must be the immediate parent of a `Positioned` widget?
* A) `Row`
* B) `Column`
* C) `Stack`
* D) `Container`

*(Correct Answer: C)*

---

### 2. True or False
**Statement:** You can specify `left`, `right`, and `width` on the same `Positioned` widget simultaneously.
*(Answer: False. Providing all three creates a mathematical conflict and causes a runtime assertion error).*

---

### 3. Output Prediction
What happens to a widget when wrapped in `Positioned(left: 0, right: 0, child: myWidget)` inside a `Stack`?
* A) It centers itself horizontally
* B) It is forced to stretch and match the full width of the `Stack`
* C) It disappears
* D) It crashes

*(Correct Answer: B)*

---

### 4. Theory
What is the function of the `Positioned.fill` constructor?
*(Answer: `Positioned.fill` sets `left: 0, top: 0, right: 0, bottom: 0` simultaneously, forcing the child to completely cover the entire area of the parent Stack).*

---

### 5. Multiple Choice Question (MCQ)
Can you provide negative coordinates (e.g. `top: -10`) to a `Positioned` widget?
* A) No, it throws an exception
* B) Yes, it allows the child to render outside the Stack bounds (if `clipBehavior` is `Clip.none`)
* C) Only on Android devices
* D) Only if the Stack has no other children

*(Correct Answer: B)*

---

### 6. Coding Exercise
Write the code to position a close button 16px from the top and 16px from the right inside a `Stack`.
*(Answer:*
```dart
Positioned(
  top: 16,
  right: 16,
  child: IconButton(
    icon: const Icon(Icons.close),
    onPressed: () {},
  ),
)
```
*)*

---

### 7. Theory
Why does `Positioned` not contribute to the natural size calculation of a `Stack`?
*(Answer: Because `Positioned` widgets are explicitly designed to align relative to the Stack's boundaries. The Stack's size must be resolved first by its non-positioned children before offsets like `right: 10` can be calculated).*

---

### 8. Output Prediction
Where will a child appear if you wrap it in `Positioned(bottom: 0, right: 0)`?
*(Answer: At the bottom-right corner of the parent `Stack`).*

---

### 9. Multiple Choice Question (MCQ)
Which constructor should you use if your app supports Right-to-Left (RTL) languages like Arabic or Hebrew?
* A) `Positioned.fill`
* B) `Positioned.directional`
* C) `Positioned.relative`
* D) `Positioned.adaptive`

*(Correct Answer: B)*

---

### 10. True or False
**Statement:** If both `top` and `height` are provided, the `bottom` property is calculated automatically.
*(Answer: True).*

---

## 📝 Summary

* **Core Purpose:** `Positioned` pins and sizes child widgets at explicit coordinates inside a `Stack`.
* **Immediate Parent:** Must always be placed directly inside a `Stack`.
* **The 2-of-3 Rule:** On each axis, you can supply at most two of the three properties (`left/right/width` or `top/bottom/height`).
* **Edge-to-Edge Stretch:** Specifying both `left: 0` and `right: 0` stretches the child horizontally; `top: 0` and `bottom: 0` stretches it vertically.
* **Shortcut Constructor:** `Positioned.fill` covers the entire `Stack` in one simple declaration.
* **Primary Real-World Uses:** Corner notification badges, floating action buttons over hero images, custom close controls, and transparent full-card touch overlays.
