# Flutter Clipping Shapes

## 📖 Introduction

By now, you understand how to clip widgets into custom shapes using `ClipPath` and `CustomClipper`. But in everyday Flutter development, you don't always need to write custom clippers from scratch. Flutter provides a set of **built-in clipping widgets** that cover the most common design patterns.

Imagine you need to:
* Display a circular profile picture
* Create a card with rounded corners
* Cut a diagonal banner across the screen
* Make a wave-shaped header

Instead of writing a custom `CustomClipper` for every shape, you can use:
* `ClipRRect` for rounded rectangles
* `ClipOval` for circles and ellipses
* `ClipRect` for standard rectangles
* `ClipPath` for custom waves and diagonals

This lesson teaches you which widget to use for each common clipping scenario and when to choose a built-in clipper over a custom one.

---

## 🤔 Why Do We Need It?

Without built-in clippers, every non-rectangular shape would require:
1. Writing a custom `CustomClipper<Path>` class
2. Manually calculating bezier curves and coordinates
3. Handling responsive sizing with `size.width` and `size.height`
4. Managing `shouldReclip()` optimization

For simple shapes like circles or rounded corners, this is overkill. Built-in clippers give you:
* **Zero boilerplate:** One line of code instead of a 20-line custom class.
* **Better performance:** Optimized native implementations.
* **Clearer intent:** Reading `ClipOval` instantly tells you "this is a circle."

---

## 🌍 Real-Life Analogy

Think of **shape templates in a craft shop**:

```text
         Custom Scissors           Shape Templates
┌──────────────────────┐    ┌──────────────────────┐
│  Draw and Cut         │    │   Ready-Made Stencil │
│  Your Own Shape      │    │                      │
│                      │    │   ⚪ Circle          │
│  (CustomClipper)     │    │   ▢ Rounded Rect     │
└──────────────────────┘    │   ◇ Diagonal         │
                            └──────────────────────┘
         (Flexible)                 (Fast & Easy)
```

* **Custom Scissors (CustomClipper):** Perfect for unique, complex shapes like custom waves or star badges.
* **Shape Templates (Built-in Clippers):** Perfect for common shapes like circles and rounded corners.

---

## 🧠 Internal Working

All clipping widgets follow the same internal pattern:

```text
Parent Widget
       │
       ├─► 1. Child widget renders normally (as a full rectangle)
       │
       ├─► 2. Clipping widget measures child size
       │
       ├─► 3. Clipping widget creates a clipping boundary
       │      - ClipRRect: Rounded rectangle path
       │      - ClipOval: Ellipse/circle path
       │      - ClipRect: Rectangle path
       │      - ClipPath: Custom path from clipper
       │
       ├─► 4. Paint phase executes
       │      Only pixels INSIDE the boundary are rendered
       │
       ▼
Final Rendered Widget (Clipped Shape)
```

---

## ⚙ Syntax

### 1. ClipRRect (Rounded Rectangle)

```dart
ClipRRect({
  Key? key,
  BorderRadius borderRadius = BorderRadius.zero,
  Clip clipBehavior = Clip.antiAlias,
  Widget? child,
})
```

**Use Case:** Cards, buttons, and containers with rounded corners.

---

### 2. ClipOval (Circle/Ellipse)

```dart
ClipOval({
  Key? key,
  Clip clipBehavior = Clip.antiAlias,
  Widget? child,
})
```

**Use Case:** Profile pictures, circular badges, and avatars.

---

### 3. ClipRect (Rectangle)

```dart
ClipRect({
  Key? key,
  Clip clipBehavior = Clip.hardEdge,
  Widget? child,
})
```

**Use Case:** Standard rectangular clipping (rarely needed, since widgets are rectangles by default).

---

### 4. ClipPath (Custom Shape)

```dart
ClipPath({
  Key? key,
  CustomClipper<Path>? clipper,
  Clip clipBehavior = Clip.antiAlias,
  Widget? child,
})
```

**Use Case:** Waves, diagonals, and any custom shape not covered by built-in clippers.

---

## 📋 Important Properties

| Widget | Key Property | Type | Purpose |
|---|---|---|---|
| **`ClipRRect`** | `borderRadius` | `BorderRadius` | Defines corner radius for rounded rectangle. |
| **`ClipOval`** | (None) | — | Automatically clips to ellipse based on child's aspect ratio. |
| **`ClipRect`** | (None) | — | Clips to rectangle (same as child's bounds). |
| **`ClipPath`** | `clipper` | `CustomClipper<Path>?` | Custom clipping boundary. |

### ClipBehavior Options (All Clippers):

| Value | Behavior |
|---|---|
| `Clip.none` | No clipping. |
| `Clip.hardEdge` | Fast, pixelated edges. |
| `Clip.antiAlias` | Smoothed edges (recommended). |
| `Clip.antiAliasWithSaveLayer` | Highest quality but slowest. |

---

## 💻 Complete Runnable Example

Below is a complete Flutter app showcasing all common clipping patterns: circular profile pictures, rounded cards, diagonal banners, and wave headers.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ClippingShapesDemoApp());
}

class ClippingShapesDemoApp extends StatelessWidget {
  const ClippingShapesDemoApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: ClippingShapesDemoScreen(),
    );
  }
}

class ClippingShapesDemoScreen extends StatelessWidget {
  const ClippingShapesDemoScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFFF8FAFC),
      appBar: AppBar(
        title: const Text(
          'Flutter Clipping Shapes',
          style: TextStyle(fontWeight: FontWeight.bold),
        ),
        backgroundColor: Colors.indigo.shade900,
        foregroundColor: Colors.white,
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            _buildSectionHeader('1. ClipOval - Circular Profile'),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceAround,
              children: [
                _buildCircularProfile('Alex', Colors.blue),
                _buildCircularProfile('Sarah', Colors.purple),
                _buildCircularProfile('Mike', Colors.green),
              ],
            ),
            const SizedBox(height: 24),

            _buildSectionHeader('2. ClipRRect - Rounded Cards'),
            Row(
              children: [
                Expanded(
                  child: _buildRoundedCard(
                    'Premium',
                    '₹ 299/mo',
                    Colors.orange,
                  ),
                ),
                const SizedBox(width: 12),
                Expanded(
                  child: _buildRoundedCard(
                    'Basic',
                    '₹ 99/mo',
                    Colors.blue,
                  ),
                ),
              ],
            ),
            const SizedBox(height: 24),

            _buildSectionHeader('3. ClipPath - Diagonal Banner'),
            _buildDiagonalBanner(),
            const SizedBox(height: 24),

            _buildSectionHeader('4. ClipPath - Wave Header'),
            _buildWaveHeader(),
          ],
        ),
      ),
    );
  }

  static Widget _buildSectionHeader(String title) {
    return Padding(
      padding: const EdgeInsets.only(bottom: 12.0),
      child: Text(
        title,
        style: const TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
      ),
    );
  }

  static Widget _buildCircularProfile(String name, Color color) {
    return Column(
      children: [
        ClipOval(
          child: Container(
            width: 80,
            height: 80,
            color: color.withValues(alpha: 0.2),
            child: Center(
              child: Text(
                name[0],
                style: TextStyle(
                  fontSize: 32,
                  fontWeight: FontWeight.bold,
                  color: color,
                ),
              ),
            ),
          ),
        ),
        const SizedBox(height: 8),
        Text(
          name,
          style: const TextStyle(fontWeight: FontWeight.w600),
        ),
      ],
    );
  }

  static Widget _buildRoundedCard(String title, String price, Color color) {
    return ClipRRect(
      borderRadius: BorderRadius.circular(16),
      child: Container(
        padding: const EdgeInsets.all(16),
        decoration: BoxDecoration(
          gradient: LinearGradient(
            colors: [color, color.withValues(alpha: 0.7)],
          ),
        ),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              title,
              style: const TextStyle(
                color: Colors.white,
                fontSize: 14,
                fontWeight: FontWeight.w500,
              ),
            ),
            const SizedBox(height: 8),
            Text(
              price,
              style: const TextStyle(
                color: Colors.white,
                fontSize: 20,
                fontWeight: FontWeight.bold,
              ),
            ),
          ],
        ),
      ),
    );
  }

  static Widget _buildDiagonalBanner() {
    return ClipPath(
      clipper: DiagonalClipper(),
      child: Container(
        width: double.infinity,
        height: 100,
        decoration: BoxDecoration(
          gradient: LinearGradient(
            colors: [Colors.red.shade400, Colors.red.shade600],
          ),
        ),
        child: const Center(
          child: Text(
            '50% OFF SALE',
            style: TextStyle(
              color: Colors.white,
              fontSize: 24,
              fontWeight: FontWeight.bold,
            ),
          ),
        ),
      ),
    );
  }

  static Widget _buildWaveHeader() {
    return ClipPath(
      clipper: WaveClipper(),
      child: Container(
        width: double.infinity,
        height: 120,
        decoration: BoxDecoration(
          gradient: LinearGradient(
            colors: [Colors.teal.shade400, Colors.teal.shade600],
          ),
        ),
        child: const Center(
          child: Text(
            'Wave Header',
            style: TextStyle(
              color: Colors.white,
              fontSize: 24,
              fontWeight: FontWeight.bold,
            ),
          ),
        ),
      ),
    );
  }
}

class DiagonalClipper extends CustomClipper<Path> {
  @override
  Path getClip(Size size) {
    final path = Path();
    path.moveTo(0, 0);
    path.lineTo(size.width, 0);
    path.lineTo(size.width - 40, size.height);
    path.lineTo(0, size.height);
    path.close();
    return path;
  }

  @override
  bool shouldReclip(CustomClipper<Path> oldClipper) {
    return false;
  }
}

class WaveClipper extends CustomClipper<Path> {
  @override
  Path getClip(Size size) {
    final path = Path();
    path.moveTo(0, 0);
    path.lineTo(size.width, 0);
    path.lineTo(size.width, size.height - 30);
    path.quadraticBezierTo(
      size.width / 2,
      size.height + 20,
      0,
      size.height - 30,
    );
    path.close();
    return path;
  }

  @override
  bool shouldReclip(CustomClipper<Path> oldClipper) {
    return false;
  }
}
```

---

## 🔍 Example Explanation

### 1. `ClipOval(child: Container(...))`
* **What it does:** Clips the child container into a perfect circle (since width and height are equal).
* **Why it exists:** Profile pictures, avatars, and circular badges are extremely common. `ClipOval` is the simplest way to achieve this.
* **What Flutter does internally:** Creates an elliptical clipping path based on the child's aspect ratio and clips to that shape.
* **What happens if removed:** The profile becomes a square instead of a circle.

### 2. `ClipRRect(borderRadius: BorderRadius.circular(16), child: Container(...))`
* **What it does:** Clips the child container into a rectangle with 16px rounded corners.
* **Why it exists:** Modern UI design uses rounded corners extensively for cards, buttons, and containers.
* **What Flutter does internally:** Creates a rounded rectangle path with the specified corner radius and clips to that shape.
* **What happens if removed:** The card becomes a sharp rectangle with 90-degree corners.

### 3. `ClipPath(clipper: DiagonalClipper(), child: Container(...))`
* **What it does:** Clips the banner container with a diagonal cut on the right side.
* **Why it exists:** Diagonal banners are common for sales, promotions, and alerts. Built-in clippers cannot do this.
* **What Flutter does internally:** Calls `DiagonalClipper.getClip()` to get the custom path, then clips to that shape.
* **What happens if removed:** The banner becomes a plain rectangle without the diagonal edge.

### 4. `ClipPath(clipper: WaveClipper(), child: Container(...))`
* **What it does:** Clips the header container with a smooth wave curve at the bottom.
* **Why it exists:** Wave headers are popular in modern apps for a friendly, organic aesthetic.
* **What Flutter does internally:** Calls `WaveClipper.getClip()` to get the bezier curve path, then clips to that shape.
* **What happens if removed:** The header becomes a plain rectangle without the wave.

---

## 📱 Phone Screen Preview

```text
+------------------------------------------+
|| Flutter Clipping Shapes                 |
+------------------------------------------+
||                                         |
|| 1. ClipOval - Circular Profile          |
||                                         |
||   ( A )       ( S )       ( M )         |
||  Alex        Sarah        Mike          |
||                                         |
|| 2. ClipRRect - Rounded Cards            |
|| +------------------+ +----------------+|
|| | Premium          | | Basic          ||
|| | ₹ 299/mo         | | ₹ 99/mo        ||
|| +------------------+ +----------------+|
||                                         |
|| 3. ClipPath - Diagonal Banner           |
|| +------------------------------------+  |
|| | / 50% OFF SALE                     |  |
|| |/                                    |  |
|| +------------------------------------+  |
||                                         |
|| 4. ClipPath - Wave Header               |
|| +------------------------------------+  |
|| |   Wave Header                      |  |
|| |   ~~~~~~~~~~~~~~~                   |  |
|| +------------------------------------+  |
+------------------------------------------+
```

### What the user sees:
* Three circular profile avatars with initials.
* Two rounded cards with gradient backgrounds (Premium and Basic pricing).
* A diagonal sale banner with a 50% OFF message.
* A wave header with a smooth curved bottom edge.

---

## 🌳 Widget Tree

```text
ClippingShapesDemoApp
└── MaterialApp
    └── ClippingShapesDemoScreen
        └── Scaffold
            ├── AppBar
            └── SingleChildScrollView
                └── Padding
                    └── Column
                        ├── Section 1: ClipOval
                        │   └── Row
                        │       ├── ClipOval -> Container (Circle: "A")
                        │       ├── ClipOval -> Container (Circle: "S")
                        │       └── ClipOval -> Container (Circle: "M")
                        ├── Section 2: ClipRRect
                        │   └── Row
                        │       ├── ClipRRect -> Container (Rounded Card: Premium)
                        │       └── ClipRRect -> Container (Rounded Card: Basic)
                        ├── Section 3: ClipPath (Diagonal)
                        │   └── ClipPath
                        │       └── DiagonalClipper (CustomClipper<Path>)
                        │           └── Container (Diagonal Banner)
                        └── Section 4: ClipPath (Wave)
                            └── ClipPath
                                └── WaveClipper (CustomClipper<Path>)
                                    └── Container (Wave Header)
```

---

## ⚙ Flutter Execution Flow

```text
1. Scaffold passes constraints to SingleChildScrollView
   │
2. Column lays out each section vertically
   │
3. ClipOval Section:
   - Three containers render as 80x80 squares
   - ClipOval clips each to a circle
   - Text initials centered inside
   │
4. ClipRRect Section:
   - Two containers render with gradients
   - ClipRRect clips each to rounded rectangle (16px radius)
   - Text overlays the gradient
   │
5. ClipPath Diagonal Section:
   - Container renders as full-width rectangle
   - DiagonalClipper.getClip() returns diagonal path
   - ClipPath clips to diagonal shape
   │
6. ClipPath Wave Section:
   - Container renders as full-width rectangle
   - WaveClipper.getClip() returns wave path
   - ClipPath clips to wave shape
```

---

## ⚠ Common Beginner Mistakes

### Mistake 1: Using ClipPath for Simple Circles

**Wrong Code:**
```dart
// ❌ OVERKILL: Custom clipper for a simple circle
ClipPath(
  clipper: CircleClipper(),
  child: Container(width: 80, height: 80),
)
```

**Why it is wrong:**
You're writing 20 lines of custom code for a shape that `ClipOval` does in one line.

**Correct Code:**
```dart
// ✅ SIMPLE: Built-in clipper
ClipOval(
  child: Container(width: 80, height: 80),
)
```

---

### Mistake 2: Using ClipRRect When BoxDecoration Has borderRadius

**Wrong Code:**
```dart
// ❌ REDUNDANT: Both ClipRRect and BoxDecoration borderRadius
ClipRRect(
  borderRadius: BorderRadius.circular(16),
  child: Container(
    decoration: BoxDecoration(
      borderRadius: BorderRadius.circular(16), // Duplicate!
      color: Colors.blue,
    ),
  ),
)
```

**Why it is redundant:**
`BoxDecoration` with `borderRadius` already clips the container's content. Wrapping it in `ClipRRect` is unnecessary.

**Correct Code:**
```dart
// ✅ CLEAN: Just BoxDecoration
Container(
  decoration: BoxDecoration(
    borderRadius: BorderRadius.circular(16),
    color: Colors.blue,
  ),
)
```

---

### Mistake 3: Forgetting ClipBehavior for Pixelated Edges

**Wrong Code:**
```dart
// ❌ PIXELATED: Default may be hardEdge on some clippers
ClipOval(
  clipBehavior: Clip.hardEdge, // Jagged edges
  child: Image.asset('avatar.png'),
)
```

**Why it looks bad:**
`Clip.hardEdge` doesn't smooth edges, resulting in pixelated, jagged circles.

**Correct Code:**
```dart
// ✅ SMOOTH: Anti-aliasing
ClipOval(
  clipBehavior: Clip.antiAlias, // Smooth edges
  child: Image.asset('avatar.png'),
)
```

---

## 🚀 Professional Usage

Choosing the right clipper is a daily decision in Flutter development:

* **Social Media Apps:** `ClipOval` for profile pictures, `ClipRRect` for post cards.
* **E-commerce:** `ClipRRect` for product cards, `ClipPath` for sale banners.
* **Banking:** `ClipRRect` for credit cards, `ClipPath` for promotional headers.
* **Onboarding:** `ClipPath` for wave-shaped splash screens and masks.
* **Gaming:** `ClipPath` for custom game UI shapes and badges.

---

## 💡 Best Practices

1. **Prefer Built-in Clippers:** Use `ClipOval`, `ClipRRect`, and `ClipRect` before writing custom clippers.
2. **Use BoxDecoration for Rounded Corners:** If you only need rounded corners on a `Container`, use `BoxDecoration` instead of `ClipRRect`.
3. **Default to `Clip.antiAlias`:** Always use anti-aliasing for smooth edges unless performance testing requires `hardEdge`.
4. **Extract Custom Clippers:** When you do write custom clippers, extract them into reusable classes.

---

## 🧪 Try It Yourself

1. Copy the runnable example above into your Flutter project.
2. Run the app and observe all four clipping patterns.
3. Change the `ClipRRect` border radius from `16` to `0` to see the sharp rectangle.
4. Modify the diagonal clipper to make the cut steeper (change `40` to `80`).

---

## 🎯 Challenge

Build a **Ticket Stub with Jagged Edges**:
* Create a `CustomClipper<Path>` that draws a ticket shape with semicircular cutouts on the left and right edges.
* Use `arcTo()` or `addArc()` to draw the semicircles.
* Place the ticket inside a list to verify it works with dynamic content.
* Add a price, event name, and date inside the ticket.

---

## ❓ Practice Questions

### 1. Multiple Choice Question (MCQ)
Which built-in clipper is best for creating a circular profile picture?
* A) `ClipRRect`
* B) `ClipOval`
* C) `ClipRect`
* D) `ClipPath`

*(Correct Answer: B)*

---

### 2. True or False
**Statement:** `BoxDecoration` with `borderRadius` is functionally equivalent to wrapping a `Container` in `ClipRRect`.
*(Answer: True. Both achieve rounded corners, but `BoxDecoration` is simpler since it's already part of the container.)*

---

### 3. Output Prediction
What happens if you use `ClipOval` on a container with `width: 100` and `height: 50`?
* A) It crashes
* B) It clips to a perfect circle
* C) It clips to an ellipse (oval shape)
* D) Nothing changes

*(Correct Answer: C)*

---

### 4. Theory
When should you use `ClipPath` instead of built-in clippers like `ClipRRect` or `ClipOval`?
*(Answer: When you need a custom shape that built-in clippers cannot achieve, such as waves, diagonals, stars, or complex geometric patterns.)*

---

### 5. Multiple Choice Question (MCQ)
Which `clipBehavior` value provides the smoothest edges but may impact performance?
* A) `Clip.none`
* B) `Clip.hardEdge`
* C) `Clip.antiAlias`
* D) `Clip.antiAliasWithSaveLayer`

*(Correct Answer: D)*

---

### 6. Coding Exercise
Write the code to create a rounded card with a 12px border radius and a blue background using `ClipRRect`:
*(Answer:*
```dart
ClipRRect(
  borderRadius: BorderRadius.circular(12),
  child: Container(
    color: Colors.blue,
    height: 100,
    width: double.infinity,
  ),
)
```
*)*

---

## 📝 Summary

* **Built-in Clippers First:** Use `ClipOval`, `ClipRRect`, and `ClipRect` for common shapes before writing custom clippers.
* **ClipOval for Circles:** Perfect for profile pictures, avatars, and circular badges.
* **ClipRRect for Rounded Corners:** Ideal for cards, buttons, and containers with border radius.
* **ClipPath for Custom Shapes:** Use when built-in clippers cannot achieve your design (waves, diagonals, stars).
* **BoxDecoration Alternative:** For simple rounded corners on containers, use `BoxDecoration` instead of `ClipRRect`.
* **ClipBehavior Matters:** Default to `Clip.antiAlias` for smooth edges; use `Clip.hardEdge` only for performance testing.
* **Responsive Design:** Always use `size.width` and `size.height` in custom clippers to support different screen sizes.
