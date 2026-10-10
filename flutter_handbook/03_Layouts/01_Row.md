# Row

## 📖 Introduction

In mobile application development, almost every screen you look at displays items placed side by side. Consider a chat app displaying a profile picture next to a user's name, or a shopping app showing a product price right beside a "Buy Now" button.

By default, computers think from top to bottom. If you place five widgets inside Flutter without specifying how they should sit, they do not automatically arrange themselves horizontally. 

This is where the **`Row`** widget comes in.

The `Row` widget is Flutter’s foundational layout tool for arranging multiple widgets in a single horizontal line across the screen. Before you learn how to customize colors, animate buttons, or fetch data from servers, you must understand how to place elements next to each other. The `Row` widget gives you full control over horizontal distribution, vertical alignment, and spacing between sibling components.

---

## 🤔 Why Do We Need It?

Imagine trying to build an interface where every button, text label, and icon has to be placed using manual X and Y pixel coordinates:

* On an iPhone SE, an element placed at `X = 320` might sit at the very edge of the screen.
* On a large Samsung Galaxy Ultra, that exact same coordinate leaves a massive empty gap on the right.
* If the user changes their device font size in accessibility settings, hard-coded coordinates cause text to collide and overlap.

We need `Row` because modern user interfaces must be **fluid and responsive**.

Instead of calculating pixel positions manually, `Row` allows you to declare relationships:
* "Distribute these three buttons evenly across whatever screen width is available."
* "Keep this icon glued to the left edge and push the timestamp to the far right."
* "Center this icon vertically against the text next to it."

`Row` solves the problem of horizontal multi-child arrangement across thousands of different screen sizes automatically.

---

## 🌍 Real-Life Analogy

Think of a **coat rack with hooks** or a **row of seats in a cinema theater**:

```text
 Seat 1      Seat 2      Seat 3      Seat 4
┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐
│ Person │  │ Person │  │ Person │  │ Person │
└────────┘  └────────┘  └────────┘  └────────┘
◄──────────────── Horizontal Axis ──────────────►
```

* When people sit in a single row of theater seats, they sit **shoulder to shoulder**.
* The **row** itself determines how wide the aisle is and how the chairs are spaced apart (packed together in the middle, spaced evenly, or pushed toward the aisles).
* Even if one person is tall and another is short, they all share that same horizontal line.
* If you attempt to squeeze 10 people into a row built for 4 seats, people spill out into the aisle. In Flutter, this exact situation causes the famous **Yellow-and-Black Striped Overflow Error**.

---

## 🧠 Internal Working

To master Flutter, you must understand what happens behind the scenes during Flutter's layout phase.

Flutter follows one golden rule for layouts:
> **"Constraints go down. Sizes go up. Parent sets position."**

Here is what Flutter does internally when a `Row` is built:

```text
Parent (e.g., Screen / Body)
       │ (Passes incoming constraints: Max Width = 400, Max Height = 800)
       ▼
     [Row]
       │
       ├─► 1. Non-Flexible Children measured first (e.g., Fixed Container, Icon, Text)
       │      Row asks Child 1: "How wide are you?" Child 1 answers: "60px".
       │      Row asks Child 2: "How wide are you?" Child 2 answers: "80px".
       │
       ├─► 2. Remaining space calculated: 400px - (60px + 80px) = 260px free space.
       │
       ├─► 3. MainAxisAlignment applied to distribute free space along Horizontal Axis.
       │
       ├─► 4. CrossAxisAlignment applied to align children along Vertical Axis.
       │
       ▼
Rendered on Screen (Positioned side by side)
```

### Understanding Axes

Every directional layout in Flutter has two axes:
1. **Main Axis:** The primary direction of the layout. For a `Row`, the main axis is **Horizontal** (left to right).
2. **Cross Axis:** The perpendicular direction. For a `Row`, the cross axis is **Vertical** (top to bottom).

```text
▲ Cross Axis (Vertical: Top to Bottom)
│
│   ┌────────┐  ┌────────┐  ┌────────┐
│   │ Item 1 │  │ Item 2 │  │ Item 3 │
│   └────────┘  └────────┘  └────────┘
┼────────────────────────────────────────► Main Axis (Horizontal: Left to Right)
```

### Unbounded Width Warning
Internally, a `Row` gives its children **unbounded (infinite) width constraints**. It tells each child: *"You can be as wide as you want."* 

If a child (like a long paragraph of text) takes more width than the physical phone screen provides, the `Row` cannot automatically wrap to a second line. Instead, it overflows the viewport boundary, triggering an overflow diagnostic error.

---

## ⚙ Syntax

The basic constructor syntax of a `Row` widget is:

```dart
Row({
  Key? key,
  MainAxisAlignment mainAxisAlignment = MainAxisAlignment.start,
  MainAxisSize mainAxisSize = MainAxisSize.max,
  CrossAxisAlignment crossAxisAlignment = CrossAxisAlignment.center,
  TextDirection? textDirection,
  VerticalDirection verticalDirection = VerticalDirection.down,
  TextBaseline? textBaseline,
  List<Widget> children = const <Widget>[],
})
```

### Parameter Breakdown:
* **`children`**: A list of widgets (`<Widget>[]`) that you want to display side by side horizontally.
* **`mainAxisAlignment`**: Determines how the free horizontal space is distributed between and around the children.
* **`crossAxisAlignment`**: Determines how children are aligned vertically relative to each other within the Row's height.
* **`mainAxisSize`**: Determines how much horizontal space the `Row` itself should occupy (`MainAxisSize.max` takes all available screen width; `MainAxisSize.min` shrinks to wrap only its children).
* **`textDirection`**: Explicitly sets reading order (`TextDirection.ltr` for left-to-right or `TextDirection.rtl` for right-to-left).
* **`verticalDirection`**: Dictates whether vertical cross-axis alignment begins from top-to-bottom (`VerticalDirection.down`) or bottom-to-top (`VerticalDirection.up`).

---

## 📋 Important Properties

| Property | Type | Default Value | Purpose |
|---|---|---|---|
| `children` | `List<Widget>` | `const []` | The list of widgets placed horizontally. |
| `mainAxisAlignment` | `MainAxisAlignment` | `start` | How children are distributed along the horizontal axis. |
| `crossAxisAlignment` | `CrossAxisAlignment` | `center` | How children are aligned along the vertical axis. |
| `mainAxisSize` | `MainAxisSize` | `max` | Whether the Row expands to fill maximum width or shrinks to fit children. |
| `textBaseline` | `TextBaseline?` | `null` | Required if using `CrossAxisAlignment.baseline` to align text fonts cleanly. |

### Visual Guide: `MainAxisAlignment` Options

| Value | Behavior Along the Horizontal Axis |
|---|---|
| `MainAxisAlignment.start` | Children packed together at the start (left edge). |
| `MainAxisAlignment.center` | Children packed together tightly in the center. |
| `MainAxisAlignment.end` | Children packed together at the end (right edge). |
| `MainAxisAlignment.spaceBetween` | Free space divided evenly **only between** children; first and last touch edges. |
| `MainAxisAlignment.spaceAround` | Free space divided evenly, with **half-sized space** before first and after last child. |
| `MainAxisAlignment.spaceEvenly` | Free space divided evenly, with **equal-sized space** between and outside all children. |

---

## 💻 Complete Runnable Example

The following code is a complete, self-contained, and production-grade Flutter application. You can paste it directly into your `lib/main.dart` file and run it.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const RowDemoApp());
}

class RowDemoApp extends StatelessWidget {
  const RowDemoApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: RowDemoScreen(),
    );
  }
}

class RowDemoScreen extends StatelessWidget {
  const RowDemoScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFFF4F6F9),
      appBar: AppBar(
        title: const Text(
          'Row Widget Deep Dive',
          style: TextStyle(fontWeight: FontWeight.bold),
        ),
        backgroundColor: Colors.indigo,
        foregroundColor: Colors.white,
        elevation: 2,
      ),
      body: SafeArea(
        child: SingleChildScrollView(
          padding: const EdgeInsets.all(16.0),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              const Text(
                '1. Space Between (Edge to Edge)',
                style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
              ),
              const SizedBox(height: 8),
              Container(
                padding: const EdgeInsets.symmetric(vertical: 12, horizontal: 8),
                decoration: BoxDecoration(
                  color: Colors.white,
                  borderRadius: BorderRadius.circular(8),
                  border: Border.all(color: Colors.grey.shade300),
                ),
                child: Row(
                  mainAxisAlignment: MainAxisAlignment.spaceBetween,
                  children: [
                    _buildTag('Left', Colors.blue),
                    _buildTag('Center', Colors.teal),
                    _buildTag('Right', Colors.orange),
                  ],
                ),
              ),
              const SizedBox(height: 24),
              const Text(
                '2. Space Evenly (Equal Gaps)',
                style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
              ),
              const SizedBox(height: 8),
              Container(
                padding: const EdgeInsets.symmetric(vertical: 12),
                decoration: BoxDecoration(
                  color: Colors.white,
                  borderRadius: BorderRadius.circular(8),
                  border: Border.all(color: Colors.grey.shade300),
                ),
                child: Row(
                  mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                  children: [
                    _buildIconBadge(Icons.call, 'Call', Colors.green),
                    _buildIconBadge(Icons.message, 'Message', Colors.blue),
                    _buildIconBadge(Icons.email, 'Email', Colors.purple),
                  ],
                ),
              ),
              const SizedBox(height: 24),
              const Text(
                '3. CrossAxisAlignment (Top vs Center vs Bottom)',
                style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
              ),
              const SizedBox(height: 8),
              Container(
                height: 100,
                padding: const EdgeInsets.all(8),
                decoration: BoxDecoration(
                  color: Colors.white,
                  borderRadius: BorderRadius.circular(8),
                  border: Border.all(color: Colors.grey.shade300),
                ),
                child: Row(
                  mainAxisAlignment: MainAxisAlignment.spaceAround,
                  crossAxisAlignment: CrossAxisAlignment.center,
                  children: [
                    Container(
                      width: 50,
                      height: 40,
                      color: Colors.indigo.shade200,
                      alignment: Alignment.center,
                      child: const Text('40px'),
                    ),
                    Container(
                      width: 50,
                      height: 80,
                      color: Colors.indigo.shade500,
                      alignment: Alignment.center,
                      child: const Text(
                        '80px',
                        style: TextStyle(color: Colors.white),
                      ),
                    ),
                    Container(
                      width: 50,
                      height: 60,
                      color: Colors.indigo.shade300,
                      alignment: Alignment.center,
                      child: const Text('60px'),
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

  static Widget _buildTag(String text, Color color) {
    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
      decoration: BoxDecoration(
        color: color.withValues(alpha: 0.15),
        borderRadius: BorderRadius.circular(6),
        border: Border.all(color: color, width: 1.5),
      ),
      child: Text(
        text,
        style: TextStyle(color: color, fontWeight: FontWeight.bold),
      ),
    );
  }

  static Widget _buildIconBadge(IconData icon, String label, Color color) {
    return Column(
      mainAxisSize: MainAxisSize.min,
      children: [
        CircleAvatar(
          backgroundColor: color.withValues(alpha: 0.15),
          radius: 22,
          child: Icon(icon, color: color),
        ),
        const SizedBox(height: 6),
        Text(
          label,
          style: TextStyle(
            fontSize: 12,
            fontWeight: FontWeight.w600,
            color: Colors.grey.shade800,
          ),
        ),
      ],
    );
  }
}
```

---

## 🔍 Example Explanation

Let's break down each essential component of this code step by step:

### 1. `void main() => runApp(...)`
* **What it does:** Serves as the initial entry point of execution and inflates `RowDemoApp` to fill the entire device screen.
* **Why it exists:** Dart requires a `main()` function to run any program.
* **What Flutter does internally:** Creates the root binding between the Dart runtime environment and the phone's rendering surface.
* **What happens if removed:** The app cannot compile or execute.

### 2. `MaterialApp` and `Scaffold`
* **What it does:** `MaterialApp` provides the Material Design configuration, and `Scaffold` gives the visual structure (AppBar, Body, background color).
* **Why it exists:** Without `Scaffold`, there are no standard layout anchors, defaults, or background colors.
* **What Flutter does internally:** Establishes the top-level MediaQuery, Directionality, and Theme widgets.
* **What happens if removed:** Text rendering appears distorted with yellow double-underlines, and no AppBar or page structure is rendered.

### 3. `Row(mainAxisAlignment: MainAxisAlignment.spaceBetween, children: [...])`
* **What it does:** Arranges the three tag containers in a single horizontal line.
* **Why it exists:** We want the "Left" tag to stick to the left edge, the "Right" tag to touch the right edge, and the "Center" tag placed precisely at the midpoint.
* **What Flutter does internally:** Calculates the combined width of all three tags, subtracts that from the total container width, and distributes the remaining pixels equally between tag 1 and 2, and between tag 2 and 3.
* **What happens if removed:** Without `Row`, multiple widgets cannot be laid out horizontally side by side; you would have to manually calculate offsets or stack them clumsily.

### 4. `Row(mainAxisAlignment: MainAxisAlignment.spaceEvenly, children: [...])`
* **What it does:** Displays three action buttons (Call, Message, Email) where the spacing before, between, and after every item is identical.
* **Why it exists:** Action bar icons look harmonious when the surrounding white space is distributed with uniform symmetry.
* **What Flutter does internally:** Takes the total width, subtracts the width of the 3 items, divides the remaining space into 4 equal segments `(N + 1)`, and places one segment between every item and screen border.
* **What happens if removed:** The buttons would clump together on the left edge (`MainAxisAlignment.start`), looking unbalanced and untidy.

### 5. `crossAxisAlignment: CrossAxisAlignment.center`
* **What it does:** Aligns three boxes of different heights (40px, 80px, 60px) along their vertical centers inside a 100px-high container.
* **Why it exists:** When child widgets have varying heights, aligning them to their center prevents jarring step-down stairs.
* **What Flutter does internally:** Takes the height of the tallest item (80px) or parent constraint (100px), computes the midpoint, and positions each child so its vertical center matches that midpoint.
* **What happens if removed:** It defaults to `center`. If changed to `start`, all boxes align along the top rim.

---

## 📱 Phone Screen Preview

```text
+------------------------------------------+
| Row Widget Deep Dive                     |
+------------------------------------------+
|                                          |
| 1. Space Between (Edge to Edge)          |
| +--------------------------------------+ |
| | [ Left ]       [ Center ]    [ Right ]| |
| +--------------------------------------+ |
|                                          |
| 2. Space Evenly (Equal Gaps)             |
| +--------------------------------------+ |
| |    (📞)          (💬)          (✉)   | |
| |    Call        Message        Email  | |
| +--------------------------------------+ |
|                                          |
| 3. CrossAxisAlignment (Vertical Centers) |
| +--------------------------------------+ |
| |           +----+                     | |
| |   +----+  |    |  +----+             | |
| |   |40px|  |80px|  |60px|             | |
| |   +----+  |    |  +----+             | |
| |           +----+                     | |
| +--------------------------------------+ |
|                                          |
+------------------------------------------+
```

### What the user sees:
1. **Top Section:** Three colored badge tags spaced out across the whole container width, touching the outer padding edges.
2. **Middle Section:** Three circular icon buttons spaced evenly with identical gaps on the left, right, and between each pair.
3. **Bottom Section:** Three purple boxes of varying heights aligned symmetrically along their horizontal middle line.

---

## 🌳 Widget Tree

```text
RowDemoApp
└── MaterialApp
    └── RowDemoScreen
        └── Scaffold
            ├── AppBar
            │   └── Text ("Row Widget Deep Dive")
            └── SafeArea
                └── SingleChildScrollView
                    └── Column
                        ├── Text ("1. Space Between...")
                        ├── Container
                        │   └── Row (mainAxisAlignment: spaceBetween)
                        │       ├── Container (Tag: "Left")
                        │       ├── Container (Tag: "Center")
                        │       └── Container (Tag: "Right")
                        ├── Text ("2. Space Evenly...")
                        ├── Container
                        │   └── Row (mainAxisAlignment: spaceEvenly)
                        │       ├── Column (Icon + Text: "Call")
                        │       ├── Column (Icon + Text: "Message")
                        │       └── Column (Icon + Text: "Email")
                        ├── Text ("3. CrossAxisAlignment...")
                        └── Container
                            └── Row (crossAxisAlignment: center)
                                ├── Container (40px)
                                ├── Container (80px)
                                └── Container (60px)
```

---

## ⚙ Flutter Execution Flow

Here is how Flutter executes and renders this screen step by step:

```text
1. main() invokes runApp(RowDemoApp)
   │
2. BuildContext passes constraints from MaterialApp down to Scaffold
   │
3. Scaffold calculates physical screen constraints (e.g., width: 390px, height: 844px)
   │
4. AppBar claims top 56px; SafeArea removes notch and home indicator insets
   │
5. Column lays out its children vertically from top to bottom
   │
6. First Row receives finite width (390px - 32px padding = 358px)
   │
7. Row measures each child's intrinsic width without constraints
   │
8. Row applies MainAxisAlignment.spaceBetween to distribute remaining empty space
   │
9. Row checks tallest child, determines Row height, applies CrossAxisAlignment.center
   │
10. RenderObject calculates final pixel coordinates and rasterizes pixels on the screen
```

---

## ⚠ Common Beginner Mistakes

### Mistake 1: The Yellow-and-Black Striped Overflow Error

**Wrong Code:**
```dart
// ❌ CRASH: Text will overflow on smaller phone screens!
Row(
  children: [
    Icon(Icons.warning),
    Text(
      'This is an extremely long sentence that will definitely exceed the phone screen boundary and cause an overflow error.',
    ),
  ],
)
```

**Why it is wrong:**
A `Row` gives its children **unbounded horizontal width**. The `Text` widget tries to render all words on one infinite single line. When the total width exceeds the screen width, Flutter throws:
`A RenderFlex overflowed by XXX pixels on the right.`

**Correct Code:**
```dart
// ✅ CORRECT: Wrap the flexible text in an Expanded or Flexible widget
Row(
  children: [
    const Icon(Icons.warning),
    const SizedBox(width: 8),
    Expanded(
      child: Text(
        'This is an extremely long sentence that will safely wrap onto multiple lines when needed.',
      ),
    ),
  ],
)
```

---

### Mistake 2: Nesting a Row inside a horizontally scrolling ListView without width

**Wrong Code:**
```dart
// ❌ RUNTIME ERROR: Both ListView and Row have unbounded horizontal width!
ListView(
  scrollDirection: Axis.horizontal,
  children: [
    Row(
      children: [
        Text('Hello'),
        Text('World'),
      ],
    ),
  ],
)
```

**Why it is wrong:**
A horizontal `ListView` provides infinite width. A `Row` also wants to expand to maximum width by default (`MainAxisSize.max`). Infinite width inside infinite width causes Flutter's layout engine to fail.

**Correct Code:**
```dart
// ✅ CORRECT: Tell the Row to only occupy the space needed by its children
ListView(
  scrollDirection: Axis.horizontal,
  children: [
    Row(
      mainAxisSize: MainAxisSize.min, // Shrink-wrap children
      children: const [
        Text('Hello'),
        SizedBox(width: 8),
        Text('World'),
      ],
    ),
  ],
)
```

---

### Mistake 3: Setting `CrossAxisAlignment.baseline` without `TextBaseline`

**Wrong Code:**
```dart
// ❌ RUNTIME ASSERTION: Crash due to missing textBaseline!
Row(
  crossAxisAlignment: CrossAxisAlignment.baseline,
  children: const [
    Text('Big', style: TextStyle(fontSize: 32)),
    Text('Small', style: TextStyle(fontSize: 14)),
  ],
)
```

**Why it is wrong:**
Flutter does not know which baseline standard to align fonts against (alphabetic vs ideographic).

**Correct Code:**
```dart
// ✅ CORRECT: Always supply textBaseline when aligning by baseline
Row(
  crossAxisAlignment: CrossAxisAlignment.baseline,
  textBaseline: TextBaseline.alphabetic,
  children: const [
    Text('Big', style: TextStyle(fontSize: 32)),
    Text('Small', style: TextStyle(fontSize: 14)),
  ],
)
```

---

## 🚀 Professional Usage

The `Row` widget is used extensively in high-profile, production-grade applications:

* **WhatsApp (Chat List Item):**
  A horizontal `Row` containing:
  `[Profile Avatar] -> [Expanded Column (Name + Message snippet)] -> [Column (Timestamp + Unread badge)]`.
* **Instagram (Post Action Bar):**
  A `Row` with `MainAxisAlignment.spaceBetween`:
  On the left, a nested `Row` holding `[Heart Icon, Comment Icon, Share Icon]`, and on the far right, `[Bookmark Icon]`.
* **Amazon (Product Card Pricing):**
  A `Row` displaying `[Current Price text]`, `[Crossed-out Original Price]`, and `[Discount percentage Badge]`.
* **Google Pay (Quick Payment Action Bar):**
  A `Row` with `MainAxisAlignment.spaceEvenly` hosting quick action buttons: `[Scan QR]`, `[Pay Contacts]`, `[Bank Transfer]`.
* **Food Delivery Apps (Delivery ETA Bar):**
  A `Row` displaying an animated delivery scooter icon, dynamic arrival time (`25 mins`), and a "Track Order" link.

---

## 💡 Best Practices

1. **Use `SizedBox(width: ...)` for Spacing:** Avoid creating empty `Container()` widgets or using arbitrary padding to create gaps between elements. A simple `SizedBox(width: 12)` is cleaner, faster, and more readable.
2. **Combine with `Expanded` for Long Content:** Whenever a `Row` contains dynamic content (such as usernames, API data, or descriptions), wrap that child in an `Expanded` widget to prevent overflow exceptions.
3. **Use `mainAxisSize: MainAxisSize.min` Inside Dialogs and Badges:** By default, `Row` expands to take the full width of its parent (`MainAxisSize.max`). When building small chips, tags, or dialog actions, set `mainAxisSize: MainAxisSize.min` so it hugs its children.
4. **Prefer `const` Constructors:** Mark immutable children and spacers with `const` to save memory and avoid redundant repainting during state rebuilds.

---

## 🧪 Try It Yourself

1. Copy the runnable example above into your editor.
2. Locate the first `Row` inside the example.
3. Change its `mainAxisAlignment` from `MainAxisAlignment.spaceBetween` to:
   * `MainAxisAlignment.center`
   * `MainAxisAlignment.spaceAround`
4. Hot reload the app after each change and observe how the spacing around the three tags rearranges instantly.

---

## 🎯 Challenge

Build a **Social Media Post Header** widget using a single main `Row`:
* **Left:** A circular profile image avatar (radius 20) followed by a 12px gap.
* **Middle:** A vertical `Column` containing the user's name in bold text (`"flutter_master"`) and their location in smaller grey text (`"San Francisco, CA"`). Ensure this middle section expands so long names don't crash the screen.
* **Right:** A vertical three-dots options `IconButton` (`Icons.more_vert`).

---

## ❓ Practice Questions

### 1. Multiple Choice Question (MCQ)
What is the default `mainAxisAlignment` of a `Row`?
* A) `MainAxisAlignment.center`
* B) `MainAxisAlignment.spaceEvenly`
* C) `MainAxisAlignment.start`
* D) `MainAxisAlignment.spaceBetween`

*(Correct Answer: C)*

---

### 2. True or False
**Statement:** A `Row` widget automatically wraps its children onto the next line if the total width exceeds the screen size.
*(Answer: False. A `Row` never wraps children automatically; it overflows. To wrap items onto the next line, you must use a `Wrap` widget).*

---

### 3. Output Prediction
What happens visually if you set `mainAxisSize: MainAxisSize.min` on a `Row` that has two children of width 50px each inside a screen of width 400px?
*(Answer: The `Row` will have a total width of exactly 100px and will tightly wrap around its two children, rather than expanding across the full 400px width).*

---

### 4. Theory
Explain the difference between `MainAxisAlignment.spaceAround` and `MainAxisAlignment.spaceEvenly`.
*(Answer: In `spaceEvenly`, the space between every child and the outer edges is identical. In `spaceAround`, the space between children is equal, but the outer margins at the far left and right edges are exactly half the size of the space between children).*

---

### 5. Multiple Choice Question (MCQ)
Which axis represents the **cross axis** for a `Row` widget?
* A) Horizontal Axis
* B) Vertical Axis
* C) Z-Axis (Depth)
* D) Diagonal Axis

*(Correct Answer: B)*

---

### 6. Coding Exercise
Write the code for a `Row` that places an icon on the far left edge and a text label on the far right edge of the screen using `MainAxisAlignment`.

*(Answer:*
```dart
Row(
  mainAxisAlignment: MainAxisAlignment.spaceBetween,
  children: const [
    Icon(Icons.arrow_back),
    Text('Settings'),
  ],
)
```
*)*

---

### 7. Theory
Why does an unconstrained `Text` widget cause a yellow-and-black striped error when placed inside a `Row` with long text?
*(Answer: Because a `Row` provides its children with unbounded (infinite) horizontal width constraints. The `Text` widget tries to render on a single infinite line instead of wrapping, exceeding the viewport width and triggering a RenderFlex overflow error).*

---

### 8. Output Prediction
What error occurs if you set `crossAxisAlignment: CrossAxisAlignment.baseline` without supplying `textBaseline`?
*(Answer: Flutter throws an assertion error at runtime because it requires `textBaseline: TextBaseline.alphabetic` or `TextBaseline.ideographic` to calculate the vertical font alignment).*

---

### 9. Multiple Choice Question (MCQ)
Which widget should you wrap around a child of a `Row` to force it to take up all remaining available horizontal space?
* A) `Center`
* B) `Container`
* C) `Expanded`
* D) `Align`

*(Correct Answer: C)*

---

### 10. True or False
**Statement:** `Row` can have an infinite number of children inside its `children` property without degrading performance.
*(Answer: False. `Row` renders all of its children at once regardless of whether they are visible on screen. For long, scrollable horizontal lists, a `ListView.builder` with `scrollDirection: Axis.horizontal` should be used instead).*

---

## 📝 Summary

* **Primary Purpose:** `Row` arranges multiple child widgets in a single horizontal line across the screen.
* **Main vs Cross Axis:** Main Axis is **Horizontal** (left-to-right); Cross Axis is **Vertical** (top-to-bottom).
* **MainAxisAlignment:** Controls horizontal spacing (`start`, `center`, `end`, `spaceBetween`, `spaceAround`, `spaceEvenly`).
* **CrossAxisAlignment:** Controls vertical alignment (`start`, `center`, `end`, `stretch`, `baseline`).
* **MainAxisSize:** Controls whether the Row fills all available horizontal width (`max`) or shrink-wraps its contents (`min`).
* **Overflow Protection:** Children with dynamic or long widths must be wrapped in `Expanded` or `Flexible` to prevent yellow-and-black striped overflow errors.
* **Spacing Standard:** Use `SizedBox(width: ...)` between items for clean, readable horizontal margins.
