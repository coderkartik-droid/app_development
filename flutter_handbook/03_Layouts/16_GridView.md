# GridView

## 📖 Introduction

While `ListView` excels at displaying items one after another in a single column or row, many of the most engaging interfaces in modern mobile apps require a 2D layout: **a grid of items**.

Think of:
* The Instagram profile page showing a 3-column grid of photo squares.
* The Google Photos gallery displaying pictures side by side in neat rows.
* The Amazon search page showing products in a 2-column comparison card view.
* An iPad dashboard displaying app icons in a 4x6 grid.

You could theoretically attempt to build this using nested `Row` and `Column` widgets. But managing 2D math, responsive columns, and synchronized vertical scrolling across multiple rows would require hundreds of lines of fragile boilerplate code.

Flutter solves this elegantly with the **`GridView`** widget.

The `GridView` widget displays its children in a scrollable, 2D array of boxes. It automatically manages column counts, cross-axis distribution, aspect ratios, and scrolling physics with minimal configuration.

---

## 🤔 Why Do We Need It?

Without `GridView`, building 2D scrollable grids presents severe layout challenges:

1. **Complex Sizing Math:** Calculating item widths based on screen width minus margins, dividers, and paddings requires error-prone math. `GridView` calculates all dimensions automatically based on simple rules (e.g. "I want 2 columns" or "Each item should be at most 180px wide").
2. **Unified Scroll Engine:** Trying to nest multiple `Row` widgets inside a `Column` or `SingleChildScrollView` does not provide smooth 2D recycling or grid alignment. `GridView` provides a native sliver-based grid engine.
3. **Aspect Ratio Enforcement:** In a grid, items must maintain harmonious proportions. `GridView` provides `childAspectRatio` (width-to-height ratio) to ensure cards don't distort on different screen sizes.

---

## 🌍 Real-Life Analogy

Think of **an ice cube tray**, **a waffle iron**, or **a wall of post office PO boxes**:

```text
 ┌──────────────────────┬──────────────────────┐
 │    PO Box #101       │    PO Box #102       │
 ├──────────────────────┼──────────────────────┤
 │    PO Box #103       │    PO Box #104       │
 ├──────────────────────┼──────────────────────┤
 │    PO Box #105       │    PO Box #106       │
 └──────────────────────┴──────────────────────┘
 ◄──────────────── 2 Columns ──────────────────►
```

* An ice cube tray has fixed rows and columns.
* Water poured into the tray is neatly divided into uniform, identical compartments.
* Regardless of whether the tray is short or long, every cube slot shares the exact same shape and spacing.
* That ice cube tray is the `GridView`.

---

## 🧠 Internal Working

How does Flutter’s layout engine execute a `GridView`?

```text
Parent (Incoming Bounds: Width = 360px, Height = 800px)
       │
       ▼
   [GridView] (gridDelegate: SliverGridDelegateWithFixedCrossAxisCount)
       │
       ├─► 1. Consults the GridDelegate:
       │      Delegate specifies: crossAxisCount = 2, crossAxisSpacing = 10, childAspectRatio = 1.0
       │
       ├─► 2. Calculates Child Cell Dimensions:
       │      Usable Width = 360px - 10px (spacing) = 350px
       │      Width per Cell = 350px / 2 = 175px
       │      Height per Cell = Width (175px) / childAspectRatio (1.0) = 175px
       │
       ├─► 3. Imposes Strict Constraints on Children:
       │      Each child receives: minWidth = maxWidth = 175px, minHeight = maxHeight = 175px
       │
       ├─► 4. Places Cells into Coordinates:
       │      Cell 0 -> (Row 0, Col 0) -> (X: 0,   Y: 0)
       │      Cell 1 -> (Row 0, Col 1) -> (X: 185, Y: 0)
       │      Cell 2 -> (Row 1, Col 0) -> (X: 0,   Y: 185)
       │      Cell 3 -> (Row 1, Col 1) -> (X: 185, Y: 185)
       │
       ▼
Viewport clips and renders visible grid cells cleanly onto screen
```

### The Concept of `gridDelegate`
Every `GridView` requires a **delegate** that dictates how the 2D geometry is calculated. Flutter provides two primary pre-built delegates:

1. **`SliverGridDelegateWithFixedCrossAxisCount`:**
   * You specify the exact number of columns (e.g. `crossAxisCount: 2` or `3`).
   * Column width changes dynamically as the screen gets wider.
2. **`SliverGridDelegateWithMaxCrossAxisExtent`:**
   * You specify the maximum width an item can have (e.g. `maxCrossAxisExtent: 150`).
   * On a phone, Flutter might fit 2 columns; on a wide tablet, it automatically fits 5 columns!

---

## ⚙ Syntax

The constructor of `GridView`:

```dart
GridView({
  Key? key,
  Axis scrollDirection = Axis.vertical,
  bool reverse = false,
  ScrollController? controller,
  bool? primary,
  ScrollPhysics? physics,
  bool shrinkWrap = false,
  EdgeInsetsGeometry? padding,
  required SliverGridDelegate gridDelegate,
  bool addAutomaticKeepAlives = true,
  bool addRepaintBoundaries = true,
  bool addSemanticIndexes = true,
  double? cacheExtent,
  List<Widget> children = const <Widget>[],
  int? semanticChildCount,
  DragStartBehavior dragStartBehavior = DragStartBehavior.start,
  Clip clipBehavior = Clip.hardEdge,
  ScrollViewKeyboardDismissBehavior keyboardDismissBehavior = ScrollViewKeyboardDismissBehavior.manual,
  String? restorationId,
})
```

### Convenience Named Constructors:
* **`GridView.count(...)`**: Creates a grid with a fixed number of columns without manually writing the delegate (e.g. `GridView.count(crossAxisCount: 3, children: [...])`).
* **`GridView.extent(...)`**: Creates a grid with items of a maximum width (e.g. `GridView.extent(maxCrossAxisExtent: 160, children: [...])`).

---

## 📋 Important Properties

| Property | Type | Default Value | Purpose |
|---|---|---|---|
| `gridDelegate` | `SliverGridDelegate` | *Required* | Mathematical algorithm controlling grid cell sizing and column counts. |
| `children` | `List<Widget>` | `const []` | The list of widgets placed inside the grid cells. |
| `padding` | `EdgeInsetsGeometry?` | `null` | Outer padding around the entire grid. |
| `physics` | `ScrollPhysics?` | Platform default | Touch scroll behavior (bounce, clamp). |
| `shrinkWrap` | `bool` | `false` | Whether the grid should wrap its contents' height. |
| `scrollDirection`| `Axis` | `Axis.vertical` | Scroll axis (`vertical` or `horizontal`). |

### Key Properties inside `SliverGridDelegateWithFixedCrossAxisCount`:
* **`crossAxisCount`**: Number of columns.
* **`mainAxisSpacing`**: Vertical gap between rows.
* **`crossAxisSpacing`**: Horizontal gap between columns.
* **`childAspectRatio`**: The ratio of the cross-axis to the main-axis extent (Width ÷ Height). Defaults to `1.0` (square).

---

## 💻 Complete Runnable Example

Below is a complete, runnable Flutter app showcasing `GridView` in two realistic patterns: a 2-column e-commerce product catalog and a 4-column quick-action banking menu.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GridViewDemoApp());
}

class GridViewDemoApp extends StatelessWidget {
  const GridViewDemoApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: GridViewDemoScreen(),
    );
  }
}

class GridViewDemoScreen extends StatelessWidget {
  const GridViewDemoScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFFF1F5F9),
      appBar: AppBar(
        title: const Text(
          'GridView Deep Dive',
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
              _buildSectionHeader('1. Quick Banking Actions (4 Columns)'),
              // 4-column GridView using GridView.count
              GridView.count(
                shrinkWrap: true, // Required inside SingleChildScrollView
                physics: const NeverScrollableScrollPhysics(), // Disables inner scroll
                crossAxisCount: 4,
                crossAxisSpacing: 10,
                mainAxisSpacing: 10,
                childAspectRatio: 0.85, // Slightly taller than wide
                children: [
                  _buildActionTile(Icons.send_rounded, 'Send', Colors.blue),
                  _buildActionTile(Icons.call_received_rounded, 'Receive', Colors.green),
                  _buildActionTile(Icons.account_balance, 'Bank', Colors.amber),
                  _buildActionTile(Icons.receipt_long, 'Bills', Colors.purple),
                  _buildActionTile(Icons.phone_android, 'Mobile', Colors.deepOrange),
                  _buildActionTile(Icons.sports_esports, 'Gaming', Colors.indigo),
                  _buildActionTile(Icons.local_gas_station, 'Gas', Colors.teal),
                  _buildActionTile(Icons.more_horiz, 'More', Colors.blueGrey),
                ],
              ),

              const SizedBox(height: 28),

              _buildSectionHeader('2. E-Commerce Product Grid (2 Columns)'),
              // 2-column GridView using standard constructor
              GridView(
                shrinkWrap: true,
                physics: const NeverScrollableScrollPhysics(),
                gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
                  crossAxisCount: 2,
                  crossAxisSpacing: 14,
                  mainAxisSpacing: 14,
                  childAspectRatio: 0.72, // Taller to fit image + title + price
                ),
                children: [
                  _buildProductCard('Wireless Headphones', '99.99', Icons.headphones, Colors.indigo),
                  _buildProductCard('Smart Watch V2', '149.99', Icons.watch, Colors.pink),
                  _buildProductCard('Mechanical Keyboard', '129.99', Icons.keyboard, Colors.deepPurple),
                  _buildProductCard('Ultra Gaming Mouse', '59.99', Icons.mouse, Colors.teal),
                ],
              ),
            ],
          ),
        ),
      ),
    );
  }

  static Widget _buildSectionHeader(String title) {
    return Padding(
      padding: const EdgeInsets.only(bottom: 12.0),
      child: Text(
        title,
        style: const TextStyle(fontSize: 15, fontWeight: FontWeight.bold),
      ),
    );
  }

  static Widget _buildActionTile(IconData icon, String label, Color color) {
    return Container(
      decoration: BoxDecoration(
        color: Colors.white,
        borderRadius: BorderRadius.circular(12),
        border: Border.all(color: Colors.grey.shade300),
      ),
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          CircleAvatar(
            backgroundColor: color.withValues(alpha: 0.15),
            radius: 20,
            child: Icon(icon, color: color, size: 20),
          ),
          const SizedBox(height: 6),
          Text(
            label,
            style: const TextStyle(fontSize: 11, fontWeight: FontWeight.bold),
          ),
        ],
      ),
    );
  }

  static Widget _buildProductCard(String name, String price, IconData icon, Color color) {
    return Container(
      decoration: BoxDecoration(
        color: Colors.white,
        borderRadius: BorderRadius.circular(14),
        border: Border.all(color: Colors.grey.shade300),
      ),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          // Mock Image Area
          Expanded(
            child: Container(
              decoration: BoxDecoration(
                color: color.withValues(alpha: 0.1),
                borderRadius: const BorderRadius.only(
                  topLeft: Radius.circular(14),
                  topRight: Radius.circular(14),
                ),
              ),
              child: Center(
                child: Icon(icon, size: 54, color: color),
              ),
            ),
          ),
          // Product Info
          Padding(
            padding: const EdgeInsets.all(10.0),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(
                  name,
                  maxLines: 1,
                  overflow: TextOverflow.ellipsis,
                  style: const TextStyle(fontWeight: FontWeight.bold, fontSize: 13),
                ),
                const SizedBox(height: 4),
                Text(
                  '\$$price',
                  style: TextStyle(
                    fontWeight: FontWeight.bold,
                    fontSize: 15,
                    color: Colors.teal.shade800,
                  ),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}
```

---

## 🔍 Example Explanation

### 1. `GridView.count(crossAxisCount: 4, ...)`
* **What it does:** Organizes 8 quick-action banking buttons into a clean 4-column by 2-row grid.
* **Why it exists:** Provides an instant shortcut constructor when you have a static list of items and a fixed column count.
* **What Flutter does internally:** Creates a `SliverGridDelegateWithFixedCrossAxisCount` with `crossAxisCount = 4`, divides the available width by 4, and aligns the children into 4 columns.
* **What happens if removed:** If built with nested `Row` and `Column`, you would have to manually partition the 8 items into two separate lists and build two separate rows.

### 2. `childAspectRatio: 0.72` on the Product Grid
* **What it does:** Sets the width-to-height ratio of each product card.
* **Why it exists:** A default ratio of `1.0` creates square cards, which leaves no vertical room for product image, title, and price. Setting `0.72` makes each cell taller than it is wide.
* **What Flutter does internally:** Computes cell height as `CellWidth ÷ 0.72`. For example, if cell width is 160px, cell height is set to `160 / 0.72 = 222px`.
* **What happens if removed:** Defaults to `1.0` (square); text and price overflow outside the card bottom.

### 3. `shrinkWrap: true` and `physics: NeverScrollableScrollPhysics()`
* **What it does:** Allows the two GridViews to sit inside a single outer `SingleChildScrollView`.
* **Why it exists:** By default, `GridView` tries to expand to infinite height and scroll independently. Disabling inner scrolling lets the entire page scroll as one cohesive document.
* **What Flutter does internally:** Forces the grid to compute its exact collective height and pass it up to the parent `SingleChildScrollView`.
* **What happens if removed:** Flutter crashes with `Vertical viewport was given unbounded height.`

---

## 📱 Phone Screen Preview

```text
+------------------------------------------+
| GridView Deep Dive                       |
+------------------------------------------+
|                                          |
| 1. Quick Banking Actions (4 Columns)     |
| +----+ +----+ +----+ +----+              |
| |(✈) | |(↓) | |(🏛) | |(📄) |              |
| |Send| |Recv| |Bank| |Bill|              |
| +----+ +----+ +----+ +----+              |
| +----+ +----+ +----+ +----+              |
| |(📱) | |(🎮) | |(⛽) | |(••) |              |
| |Mob | |Game| |Gas | |More|              |
| +----+ +----+ +----+ +----+              |
|                                          |
| 2. E-Commerce Product Grid (2 Columns)   |
| +---------------+   +---------------+    |
| |    ( 🎧 )     |   |    ( ⌚ )     |    |
| |               |   |               |    |
| | Wireless Head |   | Smart Watch V2|    |
| | $99.99        |   | $149.99       |    |
| +---------------+   +---------------+    |
| +---------------+   +---------------+    |
| |    ( ⌨ )      |   |    ( 🖱 )     |    |
| | Mech Keyboard |   | Ultra Gaming  |    |
| | $129.99       |   | $59.99        |    |
| +---------------+   +---------------+    |
+------------------------------------------+
```

### What the user sees:
* A 4-column quick action grid with circular icons.
* A 2-column e-commerce catalog featuring product image thumbnails, titles, and green prices.
* Flawless vertical scrolling across the entire composite screen.

---

## 🌳 Widget Tree

```text
GridViewDemoApp
└── MaterialApp
    └── GridViewDemoScreen
        └── Scaffold
            ├── AppBar
            │   └── Text ("GridView Deep Dive")
            └── SafeArea
                └── SingleChildScrollView
                    └── Column
                        ├── Text ("1. Quick Banking Actions...")
                        ├── GridView.count (crossAxisCount: 4)
                        │   ├── Container (Send)
                        │   ├── Container (Receive)
                        │   └── ... (remaining 6 actions)
                        ├── SizedBox (height: 28)
                        ├── Text ("2. E-Commerce Product Grid...")
                        └── GridView (crossAxisCount: 2)
                            ├── Container (Wireless Headphones)
                            ├── Container (Smart Watch)
                            ├── Container (Mechanical Keyboard)
                            └── Container (Ultra Gaming Mouse)
```

---

## ⚙ Flutter Execution Flow

```text
1. SingleChildScrollView passes bounded width (358px) and unbounded height
   │
2. First GridView (crossAxisCount: 4) computes:
   - 3 gaps of 10px = 30px
   - Usable width = 358px - 30px = 328px
   - Column width = 328 / 4 = 82px
   - Cell height = 82 / 0.85 = 96px
   - Total height for 2 rows = (96 * 2) + 10 = 202px
   │
3. Second GridView (crossAxisCount: 2) computes:
   - 1 gap of 14px
   - Usable width = 358px - 14px = 344px
   - Column width = 344 / 2 = 172px
   - Cell height = 172 / 0.72 = 238px
   - Total height for 2 rows = (238 * 2) + 14 = 490px
   │
4. Entire page assembled and rasterized
```

---

## ⚠ Common Beginner Mistakes

### Mistake 1: Not adjusting `childAspectRatio` when content overflows

**Wrong Code:**
```dart
// ❌ OVERFLOW: Product card content overflows the bottom edge!
GridView.count(
  crossAxisCount: 2,
  // childAspectRatio defaults to 1.0 (square)!
  children: [
    ProductCardWithLongText(), // Square cell is too short!
  ],
)
```

**Why it is wrong:**
By default, `childAspectRatio` is `1.0`, meaning every cell is a perfect square. If your product card contains an image (120px) plus two lines of text (40px) plus a price and button (50px), the total required height is 210px. If the phone width produces a cell width of only 160px, the cell height is also 160px, causing a 50px bottom overflow!

**Correct Code:**
```dart
// ✅ CORRECT: Lower the ratio to give cells more vertical height
GridView.count(
  crossAxisCount: 2,
  childAspectRatio: 0.7, // Width / Height -> Height = Width / 0.7
  children: [
    ProductCardWithLongText(),
  ],
)
```

---

### Mistake 2: Placing `GridView` in a `Column` without `Expanded` or `shrinkWrap`

**Wrong Code:**
```dart
// ❌ RUNTIME CRASH: Vertical viewport was given unbounded height!
Column(
  children: [
    Text('My Grid'),
    GridView.count(
      crossAxisCount: 2,
      children: [...],
    ),
  ],
)
```

**Why it is wrong:**
Like `ListView`, `GridView` expands to infinite height along its scroll axis. Placing it in an unconstrained `Column` crashes the app.

**Correct Code:**
```dart
// ✅ CORRECT: Wrap in Expanded (if it takes full remaining height)
// OR use shrinkWrap: true + physics: NeverScrollableScrollPhysics() (if in a scroll view)
Expanded(
  child: GridView.count(
    crossAxisCount: 2,
    children: [...],
  ),
)
```

---

## 🚀 Professional Usage

`GridView` is used in nearly every commercial application:

* **Instagram (User Profile Photos):**
  A 3-column square grid (`crossAxisCount: 3, childAspectRatio: 1.0`) displaying thumbnails of uploaded user posts.
* **Amazon / Flipkart (Category Browsing):**
  A 2-column product listing with `childAspectRatio: 0.7` displaying image, discount badge, title, rating stars, and price.
* **Google Photos / Gallery App:**
  A grid displaying photos with `SliverGridDelegateWithMaxCrossAxisExtent(maxCrossAxisExtent: 120)` that adapts seamlessly when turning the phone to landscape mode.
* **Food Delivery (Cuisine Selector):**
  Square cards representing food categories (`"Pizza"`, `"Burgers"`, `"Biryani"`).

---

## 💡 Best Practices

1. **Calculate `childAspectRatio` Carefully:** Always test your grid on both small phones (e.g. iPhone SE) and large phones to ensure your `childAspectRatio` leaves enough room for text without clipping.
2. **Use `GridView.count` for Static 2D Menus:** When displaying a fixed set of options (like dashboard tools or settings tiles), `GridView.count` is cleaner and more readable than standard `GridView`.
3. **Use `GridView.builder` for Collections Over 20 Items:** Just like `ListView`, the default `GridView` builds all items upfront. For photo galleries or large product catalogs, always use `GridView.builder`.
4. **Use `MaxCrossAxisExtent` for Responsive Tablets:** If your app runs on both phones and tablets/desktops, use `SliverGridDelegateWithMaxCrossAxisExtent` so the app automatically adds more columns on wider screens.

---

## 🧪 Try It Yourself

1. Copy the runnable example above into your Flutter project.
2. In Section 2 (`Product Grid`), change `childAspectRatio: 0.72` to:
   * `childAspectRatio: 1.0` (Watch the bottom overflow stripes appear!)
   * `childAspectRatio: 0.6` (Watch the cards become taller and more spacious).
3. In Section 1, change `crossAxisCount: 4` to `crossAxisCount: 3` and observe how the tiles resize instantly.

---

## 🎯 Challenge

Build a **Responsive Photo Gallery Grid**:
* Use `GridView` with `SliverGridDelegateWithMaxCrossAxisExtent(maxCrossAxisExtent: 150, crossAxisSpacing: 8, mainAxisSpacing: 8)`.
* Populate it with 12 mock photo cards (rounded corners, subtle shadow).
* Run the app in portrait mode, and then rotate your simulator/emulator to landscape mode.
* Verify that in portrait mode Flutter renders ~2 to 3 columns, and in landscape mode it automatically expands to 5 to 6 columns without changing a single line of code!

---

## ❓ Practice Questions

### 1. Multiple Choice Question (MCQ)
What does `childAspectRatio` represent in a `SliverGridDelegateWithFixedCrossAxisCount`?
* A) Height divided by Width
* B) Width divided by Height (Width ÷ Height)
* C) Number of columns
* D) Total screen percentage

*(Correct Answer: B)*

---

### 2. True or False
**Statement:** A `childAspectRatio` of `1.0` creates perfect square grid cells.
*(Answer: True).*

---

### 3. Output Prediction
If a grid cell has a width of 150px and `childAspectRatio` is set to `0.5`, what will be the height of the cell?
* A) 75px
* B) 150px
* C) 300px (150 / 0.5 = 300px)
* D) 50px

*(Correct Answer: C)*

---

### 4. Theory
Explain the difference between `SliverGridDelegateWithFixedCrossAxisCount` and `SliverGridDelegateWithMaxCrossAxisExtent`.
*(Answer: `FixedCrossAxisCount` creates a fixed, explicit number of columns regardless of screen width. `MaxCrossAxisExtent` defines the maximum allowable width for each cell, allowing Flutter to automatically add more columns as the device screen gets wider).*

---

### 5. Multiple Choice Question (MCQ)
Which constructor is a convenient shortcut for creating a grid with a fixed column count without typing the delegate?
* A) `GridView.builder`
* B) `GridView.extent`
* C) `GridView.count`
* D) `GridView.fixed`

*(Correct Answer: C)*

---

### 6. Coding Exercise
Write the code for a 3-column square grid displaying 6 colored containers.
*(Answer:*
```dart
GridView.count(
  crossAxisCount: 3,
  childAspectRatio: 1.0,
  children: List.generate(
    6,
    (index) => Container(color: Colors.primaries[index % Colors.primaries.length]),
  ),
)
```
*)*

---

### 7. Theory
Why does decreasing the `childAspectRatio` (e.g. from 1.0 to 0.7) solve vertical overflow errors in grid cards?
*(Answer: Because `childAspectRatio = Width / Height`. Decreasing the ratio increases the calculated cell height for a given width, providing more vertical space for text and buttons).*

---

### 8. Output Prediction
What happens if you place a `GridView` directly inside a `Column` without specifying `shrinkWrap: true` or wrapping it in `Expanded`?
*(Answer: Flutter throws a runtime layout exception: `Vertical viewport was given unbounded height`).*

---

### 9. Multiple Choice Question (MCQ)
Which property controls the horizontal gap between adjacent columns in a `GridView`?
* A) `mainAxisSpacing`
* B) `crossAxisSpacing`
* C) `itemSpacing`
* D) `runSpacing`

*(Correct Answer: B)*

---

### 10. True or False
**Statement:** The standard `GridView(children: [...])` constructor lazily builds off-screen cells.
*(Answer: False. The default constructor builds all children upfront in memory. For lazy loading of large datasets, you must use `GridView.builder`).*

---

## 📝 Summary

* **Core Purpose:** `GridView` arranges children in a scrollable, 2D array of cells along horizontal and vertical axes.
* **Two Key Delegates:** `FixedCrossAxisCount` (fixed column count) and `MaxCrossAxisExtent` (responsive column count based on max cell width).
* **The Aspect Ratio Key:** `childAspectRatio = Width ÷ Height`. Lower ratios create taller cells; higher ratios create wider cells; `1.0` creates square cells.
* **Shortcut Constructors:** `GridView.count` and `GridView.extent` provide clean, readable syntax without verbose delegate classes.
* **Unbounded Warning:** Inside a `Column`, always wrap `GridView` in `Expanded` or use `shrinkWrap: true` with `NeverScrollableScrollPhysics`.
