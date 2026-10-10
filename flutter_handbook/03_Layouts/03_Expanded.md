# Expanded

## 📖 Introduction

When you start building Flutter apps with `Row` and `Column`, you will quickly encounter a very common layout dilemma:

*"How do I make a widget take up all the remaining empty space on the screen?"*

Suppose you have a search bar at the top of an app with a fixed search icon on the left (40px) and a fixed microphone icon on the right (40px). The text field in the middle should automatically stretch to fill whatever space is left between those two icons—whether running on a compact 4.7-inch phone or a massive 12.9-inch tablet.

In Flutter, you do not calculate screen widths using math formulas like `screenWidth - 80`. 

Instead, you use the **`Expanded`** widget.

The `Expanded` widget is a layout control widget designed specifically to sit inside a `Row`, `Column`, or `Flex`. It instructs its child widget to expand aggressively and claim all remaining unused space along the parent’s main axis.

---

## 🤔 Why Do We Need It?

Without `Expanded`, building responsive interfaces that gracefully adapt to thousands of different mobile screens is virtually impossible:

1. **Eliminating Hardcoded Math:** Without `Expanded`, developers would have to query `MediaQuery.of(context).size.width`, subtract paddings and margins manually, and pass fixed dimensions. If padding or device orientation changes, the math breaks instantly.
2. **Preventing Overflow Errors:** In a `Row`, if you put a `Text` widget with a long sentence, the text refuses to wrap and instead crashes past the right edge with a Yellow-and-Black striped overflow error. Wrapping that `Text` inside an `Expanded` provides it with a strict, bounded width, forcing the text to wrap cleanly onto multiple lines.
3. **Proportional Layouts (Ratios):** What if you want one container to take 70% of the screen width and another to take 30%? `Expanded` provides a `flex` property that divides available space proportionally with surgical precision.

---

## 🌍 Real-Life Analogy

Think of **passengers on a bus bench** or **elastic rubber bands**:

```text
┌──────────────┬──────────────────────────────────────────┬──────────────┐
│  Passenger 1 │            Elastic Rubber Band           │  Passenger 2 │
│ (Fixed 50px) │          (Expands to fill all gap)       │ (Fixed 50px) │
└──────────────┴──────────────────────────────────────────┴──────────────┘
◄─────────────────────────── Bus Bench ──────────────────────────────────►
```

* Two passengers sit at opposite ends of a bench.
* Between them is an elastic balloon or spring.
* No matter how wide the bench is, the balloon stretches out until every millimeter of empty wood between the passengers is completely occupied.
* If you have two elastic balloons side by side—one with strength 2 and one with strength 1—the first balloon claims twice as much space as the second. That is the `flex` factor in action.

---

## 🧠 Internal Working

How does Flutter calculate the size of an `Expanded` widget during the layout phase?

```text
Parent (e.g., Row with total available width = 360px)
       │
       ├─► Step 1: Measure Non-Flexible Children First
       │   Icon 1: Fixed 40px width
       │   Icon 2: Fixed 40px width
       │   Total Fixed Space Used = 80px
       │
       ├─► Step 2: Calculate Remaining Free Space
       │   Free Space = 360px - 80px = 280px
       │
       ├─► Step 3: Check Sum of All Flex Factors
       │   Expanded A (flex: 1) + Expanded B (flex: 3) = Total Flex: 4
       │
       ├─► Step 4: Allocate Bounded Constraints to Flexible Children
       │   Expanded A gets: (1 / 4) * 280px = 70px
       │   Expanded B gets: (3 / 4) * 280px = 210px
       │
       ▼
All children positioned and rendered without any overflow or wasted space
```

### Critical Rule: The Main Axis Only!
An `Expanded` widget only stretches its child along the **Main Axis** of its parent:
* Inside a **`Row`**: `Expanded` stretches horizontally (width).
* Inside a **`Column`**: `Expanded` stretches vertically (height).

Along the **Cross Axis**, `Expanded` simply passes the parent's cross-axis constraints down to its child.

---

## ⚙ Syntax

The constructor of `Expanded` is simple and focused:

```dart
const Expanded({
  Key? key,
  int flex = 1,
  required Widget child,
})
```

### Parameter Breakdown:
* **`child`**: The widget you want to expand to fill remaining space (e.g., a `Container`, `Text`, `ListView`, or `Image`).
* **`flex`**: An integer representing the proportional share of free space this widget should receive relative to other flexible siblings. The default value is `1`.

---

## 📋 Important Properties

| Property | Type | Default Value | Purpose |
|---|---|---|---|
| `child` | `Widget` | *Required* | The widget to be stretched along the main axis. |
| `flex` | `int` | `1` | Proportional weight used to distribute available free space among multiple `Expanded` or `Flexible` widgets. |

### Visual Guide: Understanding the `flex` Ratio

Suppose a `Row` has 300px of free horizontal space remaining after fixed widgets are laid out:

| Configuration | Space Allocation Calculation | Resulting Width |
|---|---|---|
| `Expanded(flex: 1)` + `Expanded(flex: 1)` | 300px ÷ 2 equal parts | 150px each (50% / 50%) |
| `Expanded(flex: 2)` + `Expanded(flex: 1)` | 300px ÷ 3 parts (2 parts to 1st, 1 part to 2nd) | 200px (66.6%) / 100px (33.3%) |
| `Expanded(flex: 3)` + `Expanded(flex: 1)` | 300px ÷ 4 parts (3 parts to 1st, 1 part to 2nd) | 225px (75%) / 75px (25%) |

---

## 💻 Complete Runnable Example

Below is a complete, standalone Flutter application demonstrating `Expanded` with real-world patterns: search bars, proportional split cards, and responsive vertical columns.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ExpandedDemoApp());
}

class ExpandedDemoApp extends StatelessWidget {
  const ExpandedDemoApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: ExpandedDemoScreen(),
    );
  }
}

class ExpandedDemoScreen extends StatelessWidget {
  const ExpandedDemoScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFFF8F9FA),
      appBar: AppBar(
        title: const Text(
          'Expanded Deep Dive',
          style: TextStyle(fontWeight: FontWeight.bold),
        ),
        backgroundColor: Colors.teal.shade700,
        foregroundColor: Colors.white,
        elevation: 2,
      ),
      body: SafeArea(
        child: Padding(
          padding: const EdgeInsets.all(16.0),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              // Example 1: Search Bar (Fixed Icons + Expanded Text Field)
              _buildHeading('1. Search Bar (Fixed + Expanded + Fixed)'),
              Container(
                padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 4),
                decoration: BoxDecoration(
                  color: Colors.white,
                  borderRadius: BorderRadius.circular(30),
                  border: Border.all(color: Colors.grey.shade300),
                ),
                child: Row(
                  children: [
                    const Icon(Icons.search, color: Colors.teal),
                    const SizedBox(width: 8),
                    // Expanded gives the TextField bounded horizontal width
                    Expanded(
                      child: TextField(
                        decoration: const InputDecoration(
                          hintText: 'Search products, brands, stores...',
                          border: InputBorder.none,
                          isDense: true,
                        ),
                      ),
                    ),
                    const SizedBox(width: 8),
                    Container(
                      padding: const EdgeInsets.all(6),
                      decoration: BoxDecoration(
                        color: Colors.teal.shade50,
                        shape: BoxShape.circle,
                      ),
                      child: Icon(Icons.mic, color: Colors.teal.shade700, size: 20),
                    ),
                  ],
                ),
              ),

              const SizedBox(height: 24),

              // Example 2: Proportional Flex Ratio (70% vs 30% split)
              _buildHeading('2. Proportional Ratio (flex: 2 vs flex: 1)'),
              Row(
                children: [
                  Expanded(
                    flex: 2, // Takes 2/3 (66.6%) of space
                    child: Container(
                      height: 70,
                      decoration: BoxDecoration(
                        color: Colors.teal.shade600,
                        borderRadius: BorderRadius.circular(8),
                      ),
                      alignment: Alignment.center,
                      child: const Text(
                        'flex: 2 (66.6%)\nPrimary Content',
                        textAlign: TextAlign.center,
                        style: TextStyle(color: Colors.white, fontWeight: FontWeight.bold),
                      ),
                    ),
                  ),
                  const SizedBox(width: 12),
                  Expanded(
                    flex: 1, // Takes 1/3 (33.3%) of space
                    child: Container(
                      height: 70,
                      decoration: BoxDecoration(
                        color: Colors.teal.shade200,
                        borderRadius: BorderRadius.circular(8),
                      ),
                      alignment: Alignment.center,
                      child: Text(
                        'flex: 1\n(33.3%)',
                        textAlign: TextAlign.center,
                        style: TextStyle(
                          color: Colors.teal.shade900,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                    ),
                  ),
                ],
              ),

              const SizedBox(height: 24),

              // Example 3: Expanded inside Column (Claiming Remaining Height)
              _buildHeading('3. Expanded in Column (List takes remaining height)'),
              Expanded(
                child: Container(
                  decoration: BoxDecoration(
                    color: Colors.white,
                    borderRadius: BorderRadius.circular(12),
                    border: Border.all(color: Colors.grey.shade300),
                  ),
                  child: ListView.separated(
                    padding: const EdgeInsets.all(8),
                    itemCount: 6,
                    separatorBuilder: (_, __) => const Divider(height: 1),
                    itemBuilder: (context, index) {
                      return ListTile(
                        leading: CircleAvatar(
                          backgroundColor: Colors.teal.shade100,
                          child: Text('${index + 1}'),
                        ),
                        title: Text('Dynamic Transaction #${1000 + index}'),
                        subtitle: const Text('Successfully processed via UPI'),
                        trailing: const Icon(Icons.chevron_right),
                      );
                    },
                  ),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }

  static Widget _buildHeading(String text) {
    return Padding(
      padding: const EdgeInsets.only(bottom: 8.0),
      child: Text(
        text,
        style: const TextStyle(fontSize: 14, fontWeight: FontWeight.bold),
      ),
    );
  }
}
```

---

## 🔍 Example Explanation

### 1. `Expanded(child: TextField(...))` in Search Bar
* **What it does:** Forces the `TextField` to fill all space between the search icon on the left and the microphone icon on the right.
* **Why it exists:** `TextField` requires bounded horizontal constraints. Without `Expanded`, a `TextField` inside a `Row` attempts to take infinite width, causing an immediate crash.
* **What Flutter does internally:** Subtracts the width of the search icon, mic icon, and spacing gaps from the container's width, and passes the exact remaining width constraint to the `TextField`.
* **What happens if removed:** If you remove `Expanded` and place `TextField` directly into the `Row`, Flutter throws an assertion error: `Horizontal viewport was given unbounded width.`

### 2. `Expanded(flex: 2, ...)` and `Expanded(flex: 1, ...)`
* **What it does:** Divides the row into a 2:1 ratio.
* **Why it exists:** It creates responsive split layouts without calculating pixels.
* **What Flutter does internally:** Sums the flex values `(2 + 1 = 3)`. It assigns `(2/3)` of the available free space to the first child and `(1/3)` to the second child.
* **What happens if removed:** Without `Expanded`, containers with no width either shrink to zero or require hardcoded widths that break on different screen sizes.

### 3. `Expanded(child: Container(child: ListView.separated(...)))` in Column
* **What it does:** Allows the transaction list to expand vertically and occupy all remaining vertical screen real estate down to the bottom padding.
* **Why it exists:** `ListView` has infinite natural height. Inside a `Column`, it must be bounded by `Expanded` so Flutter knows how tall the scrollable viewport should be.
* **What Flutter does internally:** Calculates the height consumed by the search bar, the heading, and the flex boxes, and gives the remaining height to the `Container` and `ListView`.
* **What happens if removed:** Placing a `ListView` directly in a `Column` without `Expanded` crashes the app with `Vertical viewport was given unbounded height.`

---

## 📱 Phone Screen Preview

```text
+------------------------------------------+
| Expanded Deep Dive                       |
+------------------------------------------+
| 1. Search Bar (Fixed + Expanded + Fixed) |
| +--------------------------------------+ |
| | (🔍) Search products...         (🎤) | |
| +--------------------------------------+ |
|                                          |
| 2. Proportional Ratio (flex: 2 vs flex: 1|
| +-------------------------+ +----------+ |
| | flex: 2 (66.6%)         | | flex: 1  | |
| | Primary Content         | | (33.3%)  | |
| +-------------------------+ +----------+ |
|                                          |
| 3. Expanded in Column (Scrollable List)  |
| +--------------------------------------+ |
| | (1) Dynamic Transaction #1000      > | |
| |--------------------------------------| |
| | (2) Dynamic Transaction #1001      > | |
| |--------------------------------------| |
| | (3) Dynamic Transaction #1002      > | |
| | (Rest of list fills screen bottom)   | |
| +--------------------------------------+ |
+------------------------------------------+
```

### What the user sees:
* A responsive pill search bar where the text input expands to fit any screen width.
* Two side-by-side tiles divided cleanly into 2/3 and 1/3 proportions.
* A clean transaction list that expands to use all available remaining vertical space without overflowing.

---

## 🌳 Widget Tree

```text
ExpandedDemoApp
└── MaterialApp
    └── ExpandedDemoScreen
        └── Scaffold
            ├── AppBar
            │   └── Text ("Expanded Deep Dive")
            └── SafeArea
                └── Padding
                    └── Column
                        ├── Text ("1. Search Bar...")
                        ├── Container
                        │   └── Row
                        │       ├── Icon (search)
                        │       ├── SizedBox (width: 8)
                        │       ├── Expanded (flex: 1)
                        │       │   └── TextField
                        │       ├── SizedBox (width: 8)
                        │       └── Container (mic icon)
                        ├── SizedBox (height: 24)
                        ├── Text ("2. Proportional Ratio...")
                        ├── Row
                        │   ├── Expanded (flex: 2)
                        │   │   └── Container (Primary Content)
                        │   ├── SizedBox (width: 12)
                        │   └── Expanded (flex: 1)
                        │       └── Container (Secondary)
                        ├── SizedBox (height: 24)
                        ├── Text ("3. Expanded in Column...")
                        └── Expanded (flex: 1)
                            └── Container
                                └── ListView.separated
```

---

## ⚙ Flutter Execution Flow

```text
1. MaterialApp and Scaffold provide physical bounds (e.g. 390 x 844 px)
   │
2. Parent Column lays out rigid children first:
   ├── Search bar container (height: ~48px)
   ├── Section headings and SizedBoxes (height: ~120px)
   └── Proportional flex Row (height: ~70px)
   │
3. Total fixed vertical space used = ~238px
   │
4. Column finds Expanded(child: ListView) at the bottom:
   Remaining Height = 844px - 238px = 606px
   │
5. Column imposes a strict tight constraint on Expanded:
   height = exactly 606px
   │
6. ListView receives bounded 606px viewport and initializes scroll physics
   │
7. Rasterizer paints pixels smoothly onto screen
```

---

## ⚠ Common Beginner Mistakes

### Mistake 1: Placing `Expanded` outside a `Row`, `Column`, or `Flex`

**Wrong Code:**
```dart
// ❌ CRASH: Expanded must be an immediate descendant of Row, Column, or Flex!
Container(
  child: Expanded(
    child: Text('Hello'),
  ),
)
```

**Why it is wrong:**
`Expanded` communicates layout requests to a parent `Flex` render object (`RenderFlex`). A regular `Container` or `Stack` does not understand `flex` properties, causing an immediate runtime error:
`Expanded widgets must be placed inside such Flex widgets as Row, Column, or Flex.`

**Correct Code:**
```dart
// ✅ CORRECT: Expanded must be an immediate child of Row, Column, or Flex
Row(
  children: const [
    Expanded(
      child: Text('Hello'),
    ),
  ],
)
```

---

### Mistake 2: Forgetting to wrap long Text inside a Row

**Wrong Code:**
```dart
// ❌ OVERFLOW: Sentence runs off the screen edge!
Row(
  children: const [
    Icon(Icons.person),
    Text('This is a very long user bio that needs to be displayed inside a profile card.'),
  ],
)
```

**Why it is wrong:**
`Row` gives unbounded width to children. `Text` tries to stay on a single line unless bounded.

**Correct Code:**
```dart
// ✅ CORRECT: Expanded forces the Text widget to wrap within available space
Row(
  children: const [
    Icon(Icons.person),
    SizedBox(width: 8),
    Expanded(
      child: Text('This is a very long user bio that will wrap cleanly onto multiple lines.'),
    ),
  ],
)
```

---

### Mistake 3: Setting a fixed width/height on the child of an Expanded

**Wrong Code:**
```dart
// ❌ IGNORED PROPERTY: Width 100 is completely overridden by Expanded!
Row(
  children: [
    Expanded(
      child: Container(
        width: 100, // This width is completely ignored!
        color: Colors.red,
      ),
    ),
  ],
)
```

**Why it is wrong:**
`Expanded` enforces a **tight constraint** on its child along the main axis. Whatever fixed width or height you specify on the child is overridden by the space calculated by `Expanded`.

**Correct Understanding:**
If you want the child to have a fixed size or shrink if it's smaller, use **`Flexible(fit: FlexFit.loose)`** instead of `Expanded`.

---

## 🚀 Professional Usage

`Expanded` is an indispensable tool in top production mobile apps:

* **WhatsApp (Chat Input Bar):**
  A horizontal `Row` with `[Emoji Icon] -> [Expanded(TextField)] -> [Attach File Icon] -> [Camera Icon] -> [Voice Record Button]`. The `Expanded` ensures the input box stretches smoothly on any phone width.
* **Instagram (Profile Statistics Header):**
  A `Row` with three equal `Expanded(flex: 1)` widgets displaying `[Posts Count]`, `[Followers Count]`, and `[Following Count]`, ensuring all three columns have identical widths.
* **Amazon / E-commerce (Product Listing Card):**
  A `Row` containing a fixed square image (100x100), followed by an `Expanded` column for the product title, rating stars, price, and delivery badge.
* **Google Pay / Banking (Keypad / Action Rows):**
  A bottom action bar with `Expanded(flex: 1)` on a secondary button ("Decline") and `Expanded(flex: 2)` on the primary button ("Pay Now").
* **Food Delivery Apps (Order Summary):**
  A `Row` with an `Expanded` item name on the left and a fixed price on the right, ensuring long item names (e.g. *"Double Truffle Mushroom Swiss Burger with Fries"*) wrap without pushing the price off screen.

---

## 💡 Best Practices

1. **Always Use `Expanded` with Text in Rows:** Whenever a `Row` contains text whose length you cannot predict (e.g., user input, usernames, API descriptions), always wrap that `Text` in an `Expanded`.
2. **Combine with `flex` for Proportional Split Grids:** Use `flex: 1`, `flex: 2`, etc., instead of calculating screen percentages with `MediaQuery`. It makes your layouts cleaner and immune to screen size edge-cases.
3. **Keep `Expanded` as a Direct Child:** Do not place intermediate widgets like `Padding` or `GestureDetector` between the `Row` and the `Expanded`. Put the `Padding` or `GestureDetector` *inside* the `Expanded` child.
4. **Use `Expanded` to Bound Scrollables in Columns:** Always wrap `ListView`, `GridView`, or `PageView` inside `Expanded` when they live inside a `Column`.

---

## 🧪 Try It Yourself

1. Copy the runnable example above into your Flutter project.
2. In Example 2, change the flex values:
   * First `Expanded` flex: `3`
   * Second `Expanded` flex: `2`
3. Notice how Flutter instantly redistributes the horizontal space into a 60% / 40% ratio without you changing any pixel values.

---

## 🎯 Challenge

Build a **Chat Message Composer Bar**:
* Wrap the composer in a bottom-docked `Row` with 12px outer padding.
* **Left:** An `IconButton` with an attachment icon (`Icons.attach_file`).
* **Middle:** An `Expanded` widget containing a rounded `TextField` with hint `"Type a message..."`.
* **Right:** A circular send button (`Icons.send`) inside a `FloatingActionButton.small` or circular `Container`.
* Verify that on both narrow and wide screens, the text field resizes fluidly while the icons maintain their fixed dimensions.

---

## ❓ Practice Questions

### 1. Multiple Choice Question (MCQ)
What is the default value of the `flex` property in an `Expanded` widget?
* A) `0`
* B) `1`
* C) `100`
* D) `null`

*(Correct Answer: B)*

---

### 2. True or False
**Statement:** An `Expanded` widget can be placed directly as a child of a `Container`.
*(Answer: False. `Expanded` must be an immediate child of a `Row`, `Column`, or `Flex`).*

---

### 3. Output Prediction
If a `Row` has an available width of 400px and contains:
`Expanded(flex: 1, child: WidgetA())` and `Expanded(flex: 3, child: WidgetB())`,
what will be the exact width of `WidgetA` and `WidgetB`?
*(Answer: Total flex = 1 + 3 = 4. WidgetA gets (1/4)*400 = 100px. WidgetB gets (3/4)*400 = 300px).*

---

### 4. Theory
What is the primary difference between how `Expanded` behaves inside a `Row` versus inside a `Column`?
*(Answer: Inside a `Row`, `Expanded` stretches its child horizontally along the X-axis (width). Inside a `Column`, `Expanded` stretches its child vertically along the Y-axis (height)).*

---

### 5. Multiple Choice Question (MCQ)
What error occurs if you put a `ListView` directly inside a `Column` without wrapping it in `Expanded` or `SizedBox`?
* A) `RenderFlex overflowed by XXX pixels`
* B) `Vertical viewport was given unbounded height`
* C) `Null check operator used on a null value`
* D) `Invalid constant value`

*(Correct Answer: B)*

---

### 6. Coding Exercise
Fix the following broken code snippet so the long text wraps safely without an overflow error:
```dart
// Broken code:
Row(
  children: const [
    Icon(Icons.info),
    Text('Very long explanatory text that exceeds device width...'),
  ],
)
```
*(Answer:*
```dart
Row(
  children: const [
    Icon(Icons.info),
    Expanded(
      child: Text('Very long explanatory text that exceeds device width...'),
    ),
  ],
)
```
*)*

---

### 7. Theory
Why does specifying `width: 200` on a `Container` have no effect when that `Container` is wrapped in an `Expanded` inside a `Row`?
*(Answer: Because `Expanded` enforces a strict, tight constraint on its child along the main axis. The width calculated by the `Expanded` widget overrides any explicit width defined on the child).*

---

### 8. Output Prediction
What happens if you have a `Row` with three `Expanded` widgets, all having `flex: 1`?
*(Answer: The available horizontal space will be divided equally into three parts (33.33% each), and each of the three children will receive identical width).*

---

### 9. Multiple Choice Question (MCQ)
Which of the following is equivalent to `Expanded(child: myWidget)`?
* A) `Flexible(fit: FlexFit.tight, child: myWidget)`
* B) `Flexible(fit: FlexFit.loose, child: myWidget)`
* C) `Align(alignment: Alignment.center, child: myWidget)`
* D) `SizedBox.expand(child: myWidget)`

*(Correct Answer: A)*

---

### 10. True or False
**Statement:** Wrapping every single child of a `Row` in `Expanded(flex: 1)` will make them share the screen width equally, regardless of their content size.
*(Answer: True. All children will be forced to take equal portions of the row's total width).*

---

## 📝 Summary

* **Core Purpose:** `Expanded` forces a child widget to fill all remaining available space along the main axis of a `Row`, `Column`, or `Flex`.
* **Directional Behavior:** Stretches **horizontally** in a `Row`; stretches **vertically** in a `Column`.
* **Immediate Parent Requirement:** Must be placed directly inside a `Row`, `Column`, or `Flex`; placing it elsewhere causes a runtime exception.
* **Proportional Splitting:** The `flex` property distributes available space proportionally (e.g., `flex: 2` vs `flex: 1`).
* **Text Overflow Guard:** Essential for wrapping unpredictable or long text inside horizontal rows.
* **Scrollable Bounding:** Required when placing scrollable widgets like `ListView` or `GridView` inside a `Column`.
