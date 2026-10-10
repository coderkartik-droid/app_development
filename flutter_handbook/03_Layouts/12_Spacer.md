# Spacer

## 📖 Introduction

Have you ever looked at a mobile app bar or navigation row and wondered how developers push one element to the far left and another to the far right?

Consider a standard e-commerce cart row:
* On the left: The text `"Total Price"`.
* On the far right: The text `"$149.00"`.
* In between: A wide, flexible gap that smoothly expands whether the app runs on an iPhone SE, an Android tablet, or a desktop monitor.

You could try to configure `MainAxisAlignment.spaceBetween` on the parent `Row`. But what if you have *three* widgets, and you want two of them clustered on the left, and only the third pushed to the far right?

This is where the **`Spacer`** widget becomes your best friend.

The `Spacer` widget creates an adjustable, empty space that can be used to tune the spacing between widgets in a `Flex` container (like `Row` or `Column`). It acts like an invisible spring that pushes neighboring widgets as far apart as possible.

---

## 🤔 Why Do We Need It?

Without `Spacer`, creating asymmetric pushing gaps requires awkward code:

1. **Pushing Specific Siblings:** `MainAxisAlignment.spaceBetween` distributes free space between *all* children equally. If you have `[BackIcon, Title, Spacer, SettingsIcon]`, `Spacer` pushes only the `SettingsIcon` to the far right while keeping `BackIcon` and `Title` clustered tightly on the left.
2. **Readability:** Writing `const Spacer()` is clean, expressive, and immediately understandable. 
3. **Proportional Empty Space:** `Spacer` has a `flex` property. If you put `Spacer(flex: 1)` on the left and `Spacer(flex: 2)` on the right, the empty gap on the right will be exactly twice as wide as the gap on the left.

---

## 🌍 Real-Life Analogy

Think of **a mechanical coil spring placed between two books on a shelf**:

```text
┌──────────────┐   \/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\   ┌──────────────┐
│  Book Left   │   \            Mechanical Spring              /   │  Book Right  │
│ [Total: $40] │   \       (Pushes books to outer edges)       /   │ [Checkout >] │
└──────────────┘   \/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\   └──────────────┘
◄──────────────────────────────── Shelf Width ────────────────────────────────────►
```

* If you place two books on a wide bookshelf, they might naturally sit next to each other.
* If you insert a compressed spring between them and release it, the spring expands forcefully until Book 1 hits the far left bookend and Book 2 hits the far right bookend.
* That invisible expanding spring is the `Spacer` widget.

---

## 🧠 Internal Working

What is `Spacer` really doing behind the scenes?

Let's look at the secret under the hood:
```dart
class Spacer extends StatelessWidget {
  const Spacer({super.key, this.flex = 1}) : assert(flex > 0);

  final int flex;

  @override
  Widget build(BuildContext context) {
    return Expanded(
      flex: flex,
      child: const SizedBox.shrink(),
    );
  }
}
```

**`Spacer` is literally an `Expanded` widget wrapping an empty `SizedBox.shrink()`!**

Here is how Flutter processes it during layout:

```text
Row (Total Width = 360px)
 │
 ├─► 1. Measures rigid children first:
 │      Text("Items (3)"): 70px
 │      Icon(Icons.arrow_forward): 24px
 │      Total rigid width = 94px
 │
 ├─► 2. Calculates remaining free space:
 │      Free space = 360px - 94px = 266px
 │
 ├─► 3. Finds Spacer(flex: 1):
 │      Because Spacer is an Expanded widget, it consumes 100% of the 266px free space!
 │
 ├─► 4. Positions Children:
 │      Text("Items (3)") placed at X = 0
 │      Spacer occupies X = 70px to X = 336px (266px blank space)
 │      Icon placed at X = 336px
 │
 ▼
Clean, responsive push to opposite edges
```

---

## ⚙ Syntax

The constructor of `Spacer` is minimal:

```dart
const Spacer({
  Key? key,
  int flex = 1,
})
```

### Parameter Breakdown:
* **`flex`**: The flex factor to use for determining how much space this `Spacer` should take relative to other flexible widgets in the row or column. Defaults to `1`. Must be greater than `0`.

---

## 📋 Important Properties

| Property | Type | Default Value | Purpose |
|---|---|---|---|
| `flex` | `int` | `1` | Proportional weight determining how much of the available empty space this spacer consumes. |

### Visual Guide: `Spacer` vs `SizedBox`

| Feature | `Spacer` | `SizedBox(width: ...)` |
|---|---|---|
| **Space Type** | **Dynamic / Flexible** | **Fixed / Rigid** |
| **Takes Remaining Space** | Yes (acts like `Expanded`) | No (fixed number of pixels) |
| **Parent Requirement** | Must be in `Row`, `Column`, or `Flex` | Can be used anywhere |
| **Adaptable to Screen Size** | Yes, automatically expands/contracts | No, stays exact pixel size |
| **Best Used For** | Pushing items to opposite edges | Adding a predictable 8px or 16px gap |

---

## 💻 Complete Runnable Example

Below is a complete, runnable Flutter app showcasing `Spacer` in three practical enterprise layouts: an app bar action header, an e-commerce order checkout row, and an onboarding screen with top text and a bottom-docked button.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const SpacerDemoApp());
}

class SpacerDemoApp extends StatelessWidget {
  const SpacerDemoApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: SpacerDemoScreen(),
    );
  }
}

class SpacerDemoScreen extends StatelessWidget {
  const SpacerDemoScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFFF8FAFC),
      appBar: AppBar(
        title: const Text(
          'Spacer Deep Dive',
          style: TextStyle(fontWeight: FontWeight.bold),
        ),
        backgroundColor: Colors.purple.shade800,
        foregroundColor: Colors.white,
        elevation: 2,
      ),
      body: SafeArea(
        child: SingleChildScrollView(
          padding: const EdgeInsets.all(16.0),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              // 1. Asymmetric Row Pushing
              _buildSectionHeader('1. Asymmetric Row Pushing (Two Left, One Right)'),
              Container(
                padding: const EdgeInsets.all(14),
                decoration: BoxDecoration(
                  color: Colors.white,
                  borderRadius: BorderRadius.circular(12),
                  border: Border.all(color: Colors.grey.shade300),
                ),
                child: Row(
                  children: [
                    const Icon(Icons.shopping_bag_outlined, color: Colors.purple),
                    const SizedBox(width: 8),
                    const Text(
                      'Cart Summary',
                      style: TextStyle(fontWeight: FontWeight.bold, fontSize: 16),
                    ),

                    // The Spacer pushes the edit button to the far right edge!
                    const Spacer(),

                    TextButton(
                      style: TextButton.styleFrom(foregroundColor: Colors.purple),
                      onPressed: () {},
                      child: const Text('Edit Items'),
                    ),
                  ],
                ),
              ),

              const SizedBox(height: 28),

              // 2. Proportional Multi-Spacer Balancing
              _buildSectionHeader('2. Multi-Spacer Ratio (flex: 1 vs flex: 2)'),
              const Text(
                'First gap is flex: 1. Second gap is flex: 2 (twice as wide):',
                style: TextStyle(fontSize: 13, color: Colors.grey),
              ),
              const SizedBox(height: 12),
              Container(
                padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 12),
                decoration: BoxDecoration(
                  color: Colors.white,
                  borderRadius: BorderRadius.circular(12),
                  border: Border.all(color: Colors.grey.shade300),
                ),
                child: Row(
                  children: [
                    _buildPill('Start', Colors.blue),

                    // Spacer with flex: 1
                    const Spacer(flex: 1),

                    _buildPill('Middle', Colors.orange),

                    // Spacer with flex: 2 (gets 2x the remaining space)
                    const Spacer(flex: 2),

                    _buildPill('End', Colors.green),
                  ],
                ),
              ),

              const SizedBox(height: 28),

              // 3. Spacer in Column (Pushing CTA to Bottom)
              _buildSectionHeader('3. Spacer in Column (Pushing CTA to Bottom)'),
              Container(
                height: 220,
                width: double.infinity,
                padding: const EdgeInsets.all(16),
                decoration: BoxDecoration(
                  color: Colors.purple.shade50,
                  borderRadius: BorderRadius.circular(14),
                  border: Border.all(color: Colors.purple.shade200),
                ),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    const Text(
                      'Ready to launch your project?',
                      style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
                    ),
                    const SizedBox(height: 6),
                    Text(
                      'All your files and settings have been configured successfully.',
                      style: TextStyle(color: Colors.purple.shade900, fontSize: 13),
                    ),

                    // Spacer claims all remaining vertical height in the 220px box
                    const Spacer(),

                    SizedBox(
                      width: double.infinity,
                      child: ElevatedButton(
                        style: ElevatedButton.styleFrom(
                          backgroundColor: Colors.purple.shade800,
                          foregroundColor: Colors.white,
                          padding: const EdgeInsets.symmetric(vertical: 12),
                        ),
                        onPressed: () {},
                        child: const Text('Deploy Application Now'),
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

  static Widget _buildPill(String text, Color color) {
    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 14, vertical: 6),
      decoration: BoxDecoration(
        color: color.withValues(alpha: 0.15),
        borderRadius: BorderRadius.circular(20),
        border: Border.all(color: color),
      ),
      child: Text(
        text,
        style: TextStyle(color: color, fontWeight: FontWeight.bold, fontSize: 12),
      ),
    );
  }
}
```

---

## 🔍 Example Explanation

### 1. `Row(children: [Icon, SizedBox, Text, Spacer(), TextButton])`
* **What it does:** Keeps the icon and "Cart Summary" text grouped together on the left, while pushing the "Edit Items" button to the far right border.
* **Why it exists:** `MainAxisAlignment.spaceBetween` would insert space between the icon and the title, which looks fractured. `Spacer` lets you choose *exactly* which gap expands.
* **What Flutter does internally:** Measures the icon, gap, text, and button. It hands all remaining row width to the `Spacer`.
* **What happens if removed:** The "Edit Items" button sits immediately next to the text on the left.

### 2. `const Spacer(flex: 1)` and `const Spacer(flex: 2)`
* **What it does:** Creates two dynamic gaps between three pills, making the second gap exactly twice as wide as the first.
* **Why it exists:** Demonstrates proportional whitespace allocation.
* **What Flutter does internally:** Sums the flex factors `(1 + 2 = 3)`. Assigns 1/3 of the free space to the first spacer, and 2/3 to the second spacer.
* **What happens if removed:** All three pills touch each other.

### 3. `Spacer()` inside a fixed-height Column
* **What it does:** Pushes the `"Deploy Application Now"` button flush against the bottom of the 220px card.
* **Why it exists:** Keeps the button anchored at the bottom regardless of whether the title is 1 line or 3 lines tall.
* **What Flutter does internally:** Calculates the height used by the title and description, and expands the `Spacer` to consume the remaining vertical height.
* **What happens if removed:** The button sits immediately below the description, leaving empty purple space at the bottom of the card.

---

## 📱 Phone Screen Preview

```text
+------------------------------------------+
| Spacer Deep Dive                         |
+------------------------------------------+
|                                          |
| 1. Asymmetric Row Pushing                |
| +--------------------------------------+ |
| | (🛍) Cart Summary       [Edit Items] | | <--- Spacer pushed button to right!
| +--------------------------------------+ |
|                                          |
| 2. Multi-Spacer Ratio                    |
| +--------------------------------------+ |
| | [Start]  ---  [Middle]  ======  [End]| |
| |        (1x gap)        (2x gap)      | |
| +--------------------------------------+ |
|                                          |
| 3. Spacer in Column (Pushing to Bottom)  |
| +--------------------------------------+ |
| | Ready to launch your project?        | |
| | All files have been configured...    | |
| |                                      | | <--- Vertical Spacer
| | [      Deploy Application Now      ] | | <--- Anchored at bottom
| +--------------------------------------+ |
+------------------------------------------+
```

### What the user sees:
* In the cart card, the icon and text are paired on the left, while the edit link is pinned to the far right.
* In the pill row, the middle pill sits closer to the start pill than the end pill due to the `flex: 1` vs `flex: 2` ratio.
* In the launch card, the deploy button is anchored to the bottom edge.

---

## 🌳 Widget Tree

```text
SpacerDemoApp
└── MaterialApp
    └── SpacerDemoScreen
        └── Scaffold
            ├── AppBar
            │   └── Text ("Spacer Deep Dive")
            └── SafeArea
                └── SingleChildScrollView
                    └── Column
                        ├── Container
                        │   └── Row
                        │       ├── Icon (shopping_bag)
                        │       ├── SizedBox (width: 8)
                        │       ├── Text ("Cart Summary")
                        │       ├── Spacer (flex: 1)
                        │       └── TextButton ("Edit Items")
                        ├── SizedBox (height: 28)
                        ├── Container
                        │   └── Row
                        │       ├── Container (Pill "Start")
                        │       ├── Spacer (flex: 1)
                        │       ├── Container (Pill "Middle")
                        │       ├── Spacer (flex: 2)
                        │       └── Container (Pill "End")
                        ├── SizedBox (height: 28)
                        └── Container (height: 220)
                            └── Column
                                ├── Text ("Ready to launch...")
                                ├── SizedBox (height: 6)
                                ├── Text ("All your files...")
                                ├── Spacer (flex: 1)
                                └── SizedBox (width: double.infinity)
                                    └── ElevatedButton ("Deploy...")
```

---

## ⚙ Flutter Execution Flow

```text
1. Row measures non-flexible children:
   - Icon + SizedBox: 32px
   - Text ("Cart Summary"): 110px
   - TextButton ("Edit Items"): 80px
   - Total fixed width = 222px
   │
2. Row determines available screen width = 358px
   │
3. Calculates free space:
   358px - 222px = 136px
   │
4. Evaluates Spacer(flex: 1):
   Spacer wraps SizedBox.shrink inside Expanded(flex: 1)
   Spacer width = exactly 136px
   │
5. Coordinates assigned:
   - Icon placed at X = 0
   - Text placed at X = 32
   - Spacer occupies X = 142 to X = 278
   - TextButton placed at X = 278
   │
6. RenderObject rasterizes pixels onto canvas
```

---

## ⚠ Common Beginner Mistakes

### Mistake 1: Placing `Spacer` inside a `Container` or `Stack`

**Wrong Code:**
```dart
// ❌ RUNTIME CRASH: Spacer must be inside Row, Column, or Flex!
Stack(
  children: const [
    Text('Top'),
    Spacer(),
    Text('Bottom'),
  ],
)
```

**Why it is wrong:**
`Spacer` uses `Expanded` internally. `Expanded` communicates exclusively with a `Flex` parent (`Row`, `Column`, or `Flex`). Placing it in a `Stack` or `Container` triggers a fatal exception:
`Spacer widgets must be placed inside Flex widgets such as Row, Column, or Flex.`

**Correct Code:**
```dart
// ✅ CORRECT: Use Spacer only inside Row or Column
Column(
  children: const [
    Text('Top'),
    Spacer(),
    Text('Bottom'),
  ],
)
```

---

### Mistake 2: Placing `Spacer` inside an unconstrained scrollable Column

**Wrong Code:**
```dart
// ❌ RUNTIME CRASH: Infinite height inside infinite height!
SingleChildScrollView(
  child: Column(
    children: const [
      Text('Header'),
      Spacer(), // CRASH!
      Text('Footer'),
    ],
  ),
)
```

**Why it is wrong:**
A `SingleChildScrollView` gives its child `Column` unbounded (infinite) vertical height. A `Spacer` tries to expand to fill all remaining height. An expanding spacer inside infinite height causes an immediate crash.

**Correct Code:**
```dart
// ✅ CORRECT: In a scrollable column, use a fixed SizedBox instead
SingleChildScrollView(
  child: Column(
    children: const [
      Text('Header'),
      SizedBox(height: 100), // Predictable fixed gap
      Text('Footer'),
    ],
  ),
)
```

---

## 🚀 Professional Usage

`Spacer` is used extensively in high-level app architectures:

* **WhatsApp (Custom Header Bars):**
  A top bar `Row`: `[Back Button] -> [SizedBox(8)] -> [Avatar] -> [SizedBox(8)] -> [User Name] -> [Spacer()] -> [Video Call Icon] -> [Phone Call Icon]`.
* **Amazon / E-commerce (Checkout Order Summary):**
  Each line item: `[Text("Subtotal")] -> [Spacer()] -> [Text("$49.99")]` and `[Text("Shipping Fee")] -> [Spacer()] -> [Text("FREE")]`.
* **Onboarding Screen (Welcome Flow):**
  A full-screen `Column`: `[Illustration Image] -> [Welcome Title] -> [Subtitle] -> [Spacer()] -> [Get Started Full-Width Button]`.
* **Music Players (Spotify / Apple Music):**
  Scrubber time row: `[Text("01:24")] -> [Spacer()] -> [Text("-02:45")]`.

---

## 💡 Best Practices

1. **Use `const Spacer()`:** Always prefix `Spacer` with `const` to optimize rebuild performance.
2. **Prefer `Spacer()` over `Expanded(child: SizedBox())`:** While functionally identical, `Spacer()` is vastly cleaner and expresses clear design intent.
3. **Never Use in ScrollViews:** Do not put `Spacer` inside a `SingleChildScrollView` or `ListView` unless the container has a strictly bounded height.
4. **Use for Complex Toolbars:** When you have a mixture of left-aligned and right-aligned action buttons, place a `Spacer` between the two groups.

---

## 🧪 Try It Yourself

1. Copy the runnable example above into your Flutter project.
2. In Section 1 (`Cart Summary`), remove the `const Spacer()`.
3. Hot reload the app and observe how the "Edit Items" button snaps back to the left, touching the text.
4. Put the `Spacer()` back, and change Section 2's `flex` values to `flex: 1` and `flex: 1`. Observe how the pills become equidistant!

---

## 🎯 Challenge

Build a **Music Player Scrub Bar Header**:
* A `Row` with 16px outer padding.
* **Left:** A shuffle icon (`Icons.shuffle`) followed by a 16px gap, followed by a repeat icon (`Icons.repeat`).
* **Center / Push:** Use a `Spacer()` to bridge the center gap.
* **Right:** A favorite heart icon (`Icons.favorite_border`) followed by a 16px gap, followed by a playlist queue icon (`Icons.queue_music`).
* Verify that on both narrow and wide screens, the two button groups stay neatly pinned to opposite sides of the player.

---

## ❓ Practice Questions

### 1. Multiple Choice Question (MCQ)
What does the `Spacer` widget actually render under the hood?
* A) An empty `Container`
* B) An `Expanded` widget wrapping a `SizedBox.shrink()`
* C) A `Padding` widget
* D) An `Align` widget

*(Correct Answer: B)*

---

### 2. True or False
**Statement:** `Spacer` can be used inside a `SingleChildScrollView` with a vertical `Column`.
*(Answer: False. A `SingleChildScrollView` provides unbounded vertical height, causing `Spacer` to throw an unbounded height layout exception).*

---

### 3. Output Prediction
What will happen if you place two `Spacer(flex: 1)` widgets between three widgets in a `Row`?
* A) Both gaps will be of equal width, distributing the empty space 50/50
* B) The second spacer will take all the space
* C) It will crash
* D) The row will overflow

*(Correct Answer: A)*

---

### 4. Theory
When should you use `Spacer` instead of `MainAxisAlignment.spaceBetween` in a `Row`?
*(Answer: When you want only *some* children pushed apart while keeping other children clustered together (e.g., two items on the left and one on the right)).*

---

### 5. Multiple Choice Question (MCQ)
What is the default value of `flex` in a `Spacer` widget?
* A) `0`
* B) `1`
* C) `10`
* D) `null`

*(Correct Answer: B)*

---

### 6. Coding Exercise
Write the code for a `Row` that places a title on the left and a close icon on the far right using `Spacer`.
*(Answer:*
```dart
Row(
  children: const [
    Text('Modal Title'),
    Spacer(),
    Icon(Icons.close),
  ],
)
```
*)*

---

### 7. Theory
Why must the `flex` property of a `Spacer` be an integer greater than zero?
*(Answer: Because `Spacer` uses `Expanded(flex: flex)`. In Flutter, a flex factor represents a non-zero proportional share of space; a flex of zero or negative value fails an internal assertion).*

---

### 8. Output Prediction
If a `Row` of width 400px contains two buttons of width 50px each and `Spacer(flex: 2)` and `Spacer(flex: 1)`, what will be the width of each spacer?
*(Answer: Remaining space = 400 - 100 = 300px. Total flex = 2 + 1 = 3. First spacer = (2/3) * 300 = 200px. Second spacer = (1/3) * 300 = 100px).*

---

### 9. Multiple Choice Question (MCQ)
Which of the following widgets is NOT a valid parent for a `Spacer`?
* A) `Row`
* B) `Column`
* C) `Flex`
* D) `Stack`

*(Correct Answer: D)*

---

### 10. True or False
**Statement:** `Spacer` can be used in a `Column` to push a footer button to the bottom of a bounded screen.
*(Answer: True).*

---

## 📝 Summary

* **Core Purpose:** `Spacer` creates an adjustable empty space that pushes adjacent siblings apart in a `Row`, `Column`, or `Flex`.
* **Under the Hood:** It is an `Expanded` widget wrapping `const SizedBox.shrink()`.
* **Asymmetric Pushing:** Enables clustering some widgets together while pushing others to the screen borders.
* **Proportional Ratios:** The `flex` property allocates empty space proportionally (e.g., `flex: 1` vs `flex: 2`).
* **No Unbounded Scrollables:** Never place a `Spacer` directly inside a vertically scrolling `SingleChildScrollView` or `ListView`.
* **Top Real-World Use Cases:** App bar headers, e-commerce total/checkout rows, onboarding screens, and music player audio bars.
