# CustomPaint and CustomPainter

## 📖 Introduction

So far, you've learned how to arrange widgets, clip them into shapes, and style them with containers. But what if you need to **draw something that doesn't exist as a widget**?

Imagine you want to:
* Draw a custom graph with lines and data points
* Create a waveform audio visualizer
* Paint a background with multiple overlapping waves
* Draw a signature pad where users can write with their finger
* Render a custom chart or diagram

Flutter's standard widgets (`Container`, `Row`, `Column`) are designed for layout and composition, not for raw drawing. When you need to paint custom graphics, you use **`CustomPaint`** and **`CustomPainter`**.

`CustomPaint` is Flutter's canvas for free-form drawing. It gives you direct access to the underlying `Canvas` object, allowing you to draw lines, circles, rectangles, paths, text, and images with pixel-level control.

---

## 🤔 Why Do We Need It?

Without `CustomPaint`, you are limited to composing existing widgets:
* You can use `Container` for rectangles and gradients
* You can use `ClipPath` to cut shapes
* You can use `Image` to display bitmap graphics

But you **cannot**:
* Draw a smooth sine wave across the screen
* Create a dynamic progress ring with animated arc segments
* Render a custom scatter plot with data points
* Draw a signature that follows the user's finger path
* Create a multi-layered wave animation

`CustomPaint` + `CustomPainter` unlocks the full power of Flutter's Skia rendering engine, allowing you to create any visual effect you can imagine.

---

## 🌍 Real-Life Analogy

Think of **a digital painting canvas**:

```text
         Widget Tree               CustomPaint
┌──────────────────────┐    ┌──────────────────────┐
│   Row of Widgets    │    │   Blank Canvas       │
│  [Text] [Image]     │    │                      │
└──────────────────────┘    │                      │
         (Layout)           │  🎨 You Paint Here   │
                            │                      │
                            │   ~~ Wave Pattern ~~ │
                            │      ⚡ Graph ⚡     │
                            └──────────────────────┘
```

* **Widget Tree** is like arranging pre-made stickers on a page.
* **CustomPaint** is like giving you a blank canvas and a paintbrush to draw whatever you want.
* **CustomPainter** is your set of instructions for what to paint (lines, curves, circles, etc.).

---

## 🧠 Internal Working

What happens inside Flutter when you use `CustomPaint`?

```text
Parent Widget
       │
       ├─► 1. CustomPaint calculates its size
       │      Uses size parameter or constraints from parent
       │
       ├─► 2. CustomPaint calls painter.paint(Canvas, Size)
       │      Your CustomPainter receives the canvas and size
       │
       ├─► 3. Your paint() method executes
       │      - You create Paint objects (color, stroke width, style)
       │      - You create Path objects (shapes, curves)
       │      - You call canvas methods: drawLine(), drawCircle(), drawPath()
       │
       ├─► 4. Child widget renders on top (if provided)
       │      The child overlays your painting
       │
       ▼
Final Rendered Widget (Painted Graphics + Child)
```

### The Difference Between Clipping and Painting

| Concept | `ClipPath` | `CustomPaint` |
|---|---|---|
| **Purpose** | Cuts an existing widget into a shape | Draws new graphics from scratch |
| **Input** | Takes a child widget and clips it | Takes no child (paints directly) |
| **Output** | Child widget is shaped | New graphics appear on screen |
| **Use Case** | Curved header, diagonal card | Wave background, graph, chart |

---

## ⚙ Syntax

### Basic CustomPaint Constructor

```dart
CustomPaint({
  Key? key,
  CustomPainter? painter,
  CustomPainter? foregroundPainter,
  Size? size,
  bool isComplex = false,
  bool willChange = false,
  Widget? child,
})
```

### CustomPainter Structure

```dart
class MyCustomPainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    // Define your drawing here
    final paint = Paint()
      ..color = Colors.blue
      ..style = PaintingStyle.fill;

    canvas.drawCircle(Offset(size.width / 2, size.height / 2), 50, paint);
  }

  @override
  bool shouldRepaint(CustomPainter oldDelegate) {
    // Return true if painting should recalculate
    return false;
  }
}
```

### Parameter Breakdown:

* **`painter`**: Your `CustomPainter` that draws **behind** the child widget.
* **`foregroundPainter`**: Your `CustomPainter` that draws **on top of** the child widget.
* **`size`**: Explicit size for the canvas. If null, uses child's size or parent constraints.
* **`child`**: Optional widget that sits on top of the painted graphics.
* **`paint(Canvas canvas, Size size)`**: The core method where you draw. `canvas` is the drawing surface, `size` is the canvas dimensions.
* **`shouldRepaint()`**: Optimization. Return `true` only if your drawing changes (e.g., animation). Return `false` for static graphics.

---

## 📋 Important Properties

| Property | Type | Default Value | Purpose |
|---|---|---|---|
| `painter` | `CustomPainter?` | `null` | Painter that draws behind the child. |
| `foregroundPainter` | `CustomPainter?` | `null` | Painter that draws on top of the child. |
| `size` | `Size?` | `null` | Explicit canvas size. |
| `child` | `Widget?` | `null` | Widget that overlays the painting. |

### Paint Object Properties:

| Property | Type | Purpose |
|---|---|---|
| `color` | `Color` | The color of the stroke or fill. |
| `style` | `PaintingStyle` | `fill` (solid) or `stroke` (outline only). |
| `strokeWidth` | `double` | Thickness of the line (for stroke style). |
| `strokeCap` | `StrokeCap` | Shape of line ends: `round`, `butt`, `square`. |
| `strokeJoin` | `StrokeJoin` | Shape of line corners: `round`, `bevel`, `miter`. |

---

## 💻 Complete Runnable Example

Below is a complete Flutter app showcasing a colored wave header using `CustomPaint` and `CustomPainter`. This is the exact pattern used in modern dashboard apps.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const CustomPaintDemoApp());
}

class CustomPaintDemoApp extends StatelessWidget {
  const CustomPaintDemoApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: CustomPaintDemoScreen(),
    );
  }
}

class CustomPaintDemoScreen extends StatelessWidget {
  const CustomPaintDemoScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFFF8FAFC),
      body: Column(
        children: [
          // Wave Header using CustomPaint
          CustomPaint(
            size: const Size(double.infinity, 200),
            painter: WaveHeaderPainter(),
            child: SafeArea(
              child: Padding(
                padding: const EdgeInsets.all(20.0),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Row(
                      mainAxisAlignment: MainAxisAlignment.spaceBetween,
                      children: [
                        const Icon(Icons.menu, color: Colors.white, size: 28),
                        Container(
                          padding: const EdgeInsets.all(8),
                          decoration: BoxDecoration(
                            color: Colors.white.withValues(alpha: 0.2),
                            shape: BoxShape.circle,
                          ),
                          child: const Icon(Icons.search, color: Colors.white),
                        ),
                      ],
                    ),
                    const SizedBox(height: 50),
                    const Text(
                      'Dashboard',
                      style: TextStyle(
                        color: Colors.white,
                        fontSize: 32,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                    const SizedBox(height: 8),
                    const Text(
                      'Welcome to your analytics',
                      style: TextStyle(
                        color: Colors.white70,
                        fontSize: 14,
                      ),
                    ),
                  ],
                ),
              ),
            ),
          ),

          // Content below the wave header
          Expanded(
            child: Padding(
              padding: const EdgeInsets.all(20.0),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  const Text(
                    'Statistics',
                    style: TextStyle(
                      fontSize: 18,
                      fontWeight: FontWeight.bold,
                      color: Color(0xFF1E293B),
                    ),
                  ),
                  const SizedBox(height: 16),
                  Row(
                    children: [
                      Expanded(
                        child: _buildStatCard('Revenue', '\$12,450', Colors.green),
                      ),
                      const SizedBox(width: 12),
                      Expanded(
                        child: _buildStatCard('Users', '3,842', Colors.blue),
                      ),
                    ],
                  ),
                  const SizedBox(height: 12),
                  Row(
                    children: [
                      Expanded(
                        child: _buildStatCard('Orders', '1,205', Colors.orange),
                      ),
                      const SizedBox(width: 12),
                      Expanded(
                        child: _buildStatCard('Rating', '4.8', Colors.purple),
                      ),
                    ],
                  ),
                ],
              ),
            ),
          ),
        ],
      ),
    );
  }

  static Widget _buildStatCard(String title, String value, Color color) {
    return Container(
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: Colors.white,
        borderRadius: BorderRadius.circular(12),
        border: Border.all(color: Colors.grey.shade200),
      ),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Text(
            title,
            style: TextStyle(
              fontSize: 12,
              color: Colors.grey.shade600,
              fontWeight: FontWeight.w500,
            ),
          ),
          const SizedBox(height: 8),
          Text(
            value,
            style: TextStyle(
              fontSize: 20,
              fontWeight: FontWeight.bold,
              color: color,
            ),
          ),
        ],
      ),
    );
  }
}

class WaveHeaderPainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    // Paint object for the gradient background
    final backgroundPaint = Paint()
      ..shader = LinearGradient(
        colors: [const Color(0xFF6366F1), const Color(0xFF8B5CF6)],
        begin: Alignment.topLeft,
        end: Alignment.bottomRight,
      ).createShader(const Rect.fromLTWH(0, 0, 400, 200));

    // Draw the full rectangle background
    canvas.drawRect(
      Rect.fromLTWH(0, 0, size.width, size.height),
      backgroundPaint,
    );

    // Paint object for the wave (semi-transparent white)
    final wavePaint = Paint()
      ..color = Colors.white.withValues(alpha: 0.2)
      ..style = PaintingStyle.fill;

    // Draw the first wave
    final wavePath1 = Path();
    wavePath1.moveTo(0, size.height * 0.7);
    wavePath1.quadraticBezierTo(
      size.width * 0.25,
      size.height * 0.5,
      size.width * 0.5,
      size.height * 0.7,
    );
    wavePath1.quadraticBezierTo(
      size.width * 0.75,
      size.height * 0.9,
      size.width,
      size.height * 0.7,
    );
    wavePath1.lineTo(size.width, size.height);
    wavePath1.lineTo(0, size.height);
    wavePath1.close();
    canvas.drawPath(wavePath1, wavePaint);

    // Paint object for the second wave (more transparent)
    final wavePaint2 = Paint()
      ..color = Colors.white.withValues(alpha: 0.1)
      ..style = PaintingStyle.fill;

    // Draw the second wave (offset and lower)
    final wavePath2 = Path();
    wavePath2.moveTo(0, size.height * 0.8);
    wavePath2.quadraticBezierTo(
      size.width * 0.3,
      size.height * 0.6,
      size.width * 0.6,
      size.height * 0.8,
    );
    wavePath2.quadraticBezierTo(
      size.width * 0.8,
      size.height * 0.95,
      size.width,
      size.height * 0.8,
    );
    wavePath2.lineTo(size.width, size.height);
    wavePath2.lineTo(0, size.height);
    wavePath2.close();
    canvas.drawPath(wavePath2, wavePaint2);
  }

  @override
  bool shouldRepaint(CustomPainter oldDelegate) {
    // Return false because the waves are static (not animated)
    return false;
  }
}
```

---

## 🔍 Example Explanation

### 1. `CustomPaint(size: const Size(double.infinity, 200), painter: WaveHeaderPainter(), child: ...)`
* **What it does:** Creates a canvas that is 200 pixels tall and spans the full width of the screen. The `WaveHeaderPainter` draws a gradient background with two overlapping waves. The `child` (SafeArea with text) renders on top of the painting.
* **Why it exists:** Standard widgets cannot draw multi-layered waves with gradients. `CustomPaint` gives us a canvas to paint the waves directly.
* **What Flutter does internally:** Creates a 200px-tall canvas, calls `paint()` method, executes the drawing commands, then overlays the child widget.
* **What happens if removed:** The header becomes a plain gradient without the wave effects, losing the modern aesthetic.

### 2. `final backgroundPaint = Paint() ..shader = LinearGradient(...).createShader(...)`
* **What it does:** Creates a `Paint` object with a linear gradient shader instead of a solid color.
* **Why it exists:** Gradients look more professional than flat colors. The `shader` property allows any gradient or complex color pattern.
* **What Flutter does internally:** Generates a gradient shader object that the GPU uses to interpolate colors during rendering.

### 3. `canvas.drawRect(Rect.fromLTWH(0, 0, size.width, size.height), backgroundPaint)`
* **What it does:** Draws a rectangle covering the entire canvas area (from top-left 0,0 to bottom-right width,height) filled with the gradient.
* **Why it exists:** This is the base background layer before we draw the waves on top.
* **What Flutter does internally:** Issues a GPU draw command to render a textured rectangle with the gradient shader.

### 4. `wavePath1.quadraticBezierTo(size.width * 0.25, size.height * 0.5, size.width * 0.5, size.height * 0.7)`
* **What it does:** Draws a curved line segment for the first wave.
* **Parameter breakdown:**
  * `(size.width * 0.25, size.height * 0.5)`: Control point that pulls the curve upward.
  * `(size.width * 0.5, size.height * 0.7)`: End point of this curve segment.
* **Why it exists:** Multiple bezier curves chained together create smooth, natural-looking waves.
* **What Flutter does internally:** Calculates the quadratic bezier curve mathematically and adds it to the path.

### 5. `canvas.drawPath(wavePath1, wavePaint)`
* **What it does:** Takes the complete wave path and paints it onto the canvas using the specified paint (semi-transparent white).
* **Why it exists:** This is the command that actually renders the wave shape to the screen.
* **What Flutter does internally:** Sends the path geometry and paint properties to the GPU for rasterization.

### 6. `shouldRepaint() return false`
* **What it does:** Tells Flutter: "This painting never changes, so don't redraw it on every frame."
* **Why it exists:** Performance optimization. If you return `true`, Flutter repaints the entire canvas on every rebuild (expensive).
* **What Flutter does internally:** Caches the painted output and reuses it across multiple paint cycles.

---

## 📱 Phone Screen Preview

```text
+------------------------------------------+
|| ╔═════════════════════════════════════╗ ||
|| ║ ☰                      🔍           ║ ||
|| ║                                     ║ ||
|| ║ Dashboard                            ║ ||
|| ║ Welcome to your analytics            ║ ||
|| ╚═════════════════════════════════════╝ ||
||  ~~~ ~~~~~ ~~~~  (Purple gradient)     ||
||    ~~  ~~~~  ~~   (Layered waves)      ||
||       ~~~ ~~~                         ||
||                                         |
|| Statistics                              |
||                                         |
|| +------------------+ +------------------+|
|| | Revenue          | | Users            ||
|| | $12,450          | | 3,842            ||
|| +------------------+ +------------------+|
|| +------------------+ +------------------+|
|| | Orders           | | Rating           ||
|| | 1,205            | | 4.8              ||
|| +------------------+ +------------------+|
+------------------------------------------+
```

### What the user sees:
* A purple gradient header with two layered white waves at the bottom.
* "Dashboard" text overlaying the gradient.
* Four stat cards (Revenue, Users, Orders, Rating) below the header in a 2x2 grid.

---

## 🌳 Widget Tree

```text
CustomPaintDemoApp
└── MaterialApp
    └── CustomPaintDemoScreen
        └── Scaffold
            └── Column
                ├── CustomPaint
                │   ├── WaveHeaderPainter (CustomPainter)
                │   │   └── paint() method:
                │   │       ├── canvas.drawRect (Gradient Background)
                │   │       ├── canvas.drawPath (Wave 1)
                │   │       └── canvas.drawPath (Wave 2)
                │   └── SafeArea
                │       └── Padding
                │           └── Column
                │               ├── Row (Menu Icon + Search Icon)
                │               ├── Text ("Dashboard")
                │               └── Text ("Welcome to your analytics")
                └── Expanded
                    └── Padding
                        └── Column
                            ├── Text ("Statistics")
                            ├── Row (Revenue + Users)
                            └── Row (Orders + Rating)
```

---

## ⚙ Flutter Execution Flow

```text
1. Scaffold passes screen constraints to Column
   │
2. Column lays out first child: CustomPaint
   │
3. CustomPaint sets its size: 200px height, full width
   │
4. CustomPaint calls painter.paint(canvas, Size(390, 200))
   │
5. WaveHeaderPainter.paint() executes:
   - Creates gradient backgroundPaint
   - canvas.drawRect() paints gradient background
   - Creates wavePaint1 (semi-transparent white)
   - Builds wavePath1 with quadratic bezier curves
   - canvas.drawPath() paints first wave
   - Creates wavePaint2 (more transparent)
   - Builds wavePath2 with offset curves
   - canvas.drawPath() paints second wave
   │
6. Child widget (SafeArea with text) renders on top
   │
7. Stat cards render below the header
```

---

## ⚠ Common Beginner Mistakes

### Mistake 1: Forgetting to Set Paint Style

**Wrong Code:**
```dart
// ❌ MISSING paint.style - defaults to fill, but confusing
final paint = Paint()
  ..color = Colors.blue
  ..strokeWidth = 5;

canvas.drawCircle(Offset(100, 100), 50, paint);
```

**Why it is wrong:**
The default `PaintingStyle.fill` ignores `strokeWidth`. If you want an outline, you must explicitly set `style`.

**Correct Code:**
```dart
// ✅ EXPLICIT style for outlines
final paint = Paint()
  ..color = Colors.blue
  ..style = PaintingStyle.stroke
  ..strokeWidth = 5;

canvas.drawCircle(Offset(100, 100), 50, paint);
```

---

### Mistake 2: Using Fixed Sizes Instead of `size` Parameter

**Wrong Code:**
```dart
// ❌ HARDCODED 400px - breaks on different screens
void paint(Canvas canvas, Size size) {
  canvas.drawRect(
    Rect.fromLTWH(0, 0, 400, 200), // Won't work on small phones
    paint,
  );
}
```

**Why it is wrong:**
Your drawing will be cut off on small screens or look tiny on large screens.

**Correct Code:**
```dart
// ✅ RESPONSIVE: Use the size parameter
void paint(Canvas canvas, Size size) {
  canvas.drawRect(
    Rect.fromLTWH(0, 0, size.width, size.height), // Adapts to any screen
    paint,
  );
}
```

---

### Mistake 3: Returning `true` in `shouldRepaint()` for Static Graphics

**Wrong Code:**
```dart
// ❌ UNNECESSARY REPAINTING
@override
bool shouldRepaint(CustomPainter oldDelegate) {
  return true; // Repaints on EVERY frame
}
```

**Why it is inefficient:**
Flutter repaints the entire canvas on every rebuild, wasting GPU resources for graphics that never change.

**Correct Code:**
```dart
// ✅ OPTIMIZED: Only return true if graphics change dynamically
@override
bool shouldRepaint(CustomPainter oldDelegate) {
  return false; // Cache the painting permanently
}
```

---

## 🚀 Professional Usage

`CustomPaint` is essential for specialized UI components:

* **Data Visualization:** Line charts, bar graphs, scatter plots, and pie charts.
* **Audio Visualizers:** Waveforms that animate in real-time with music.
* **Signature Pads:** Drawing areas where users can sign with their finger.
* **Progress Indicators:** Custom circular progress rings with animated arcs.
* **Game Graphics:** Drawing game sprites, power-ups, and special effects.
* **Decorative Backgrounds:** Multi-layered waves, gradients, and patterns.

---

## 💡 Best Practices

1. **Always Use the `size` Parameter:** Never hardcode canvas dimensions. Your painting must work on phones, tablets, and desktops.
2. **Default `shouldRepaint()` to `false`:** Only return `true` if you're animating the painting (e.g., a moving wave or growing chart).
3. **Reuse Paint Objects:** Create `Paint` objects once and reuse them for multiple draw calls to avoid allocations.
4. **Use `foregroundPainter` for Overlays:** If you need to draw on top of a child (e.g., selection highlights), use `foregroundPainter` instead of nesting another `CustomPaint`.

---

## 🧪 Try It Yourself

1. Copy the runnable example above into your Flutter project.
2. Run the app and observe the layered wave header.
3. Modify the wave opacity from `Colors.white.withValues(alpha: 0.2)` to `Colors.white.withValues(alpha: 0.5)` to make the waves more visible.
4. Change the wave control points to create steeper or gentler curves.

---

## 🎯 Challenge

Build a **Circular Progress Ring**:
* Create a `CustomPainter` that draws a gray background circle.
* Draw a colored arc representing progress (e.g., 75% complete).
* Use `canvas.drawArc()` with `startAngle` and `sweepAngle`.
* Ensure the ring is responsive using `size.width` and `size.height`.
* Add the ring to a screen with a slider that updates the progress value.

---

## ❓ Practice Questions

### 1. Multiple Choice Question (MCQ)
Which method in `CustomPainter` is responsible for the actual drawing?
* A) `shouldRepaint()`
* B) `paint(Canvas canvas, Size size)`
* C) `getClip(Size size)`
* D) `build(BuildContext)`

*(Correct Answer: B)*

---

### 2. True or False
**Statement:** `CustomPaint` can have both a `painter` and a `foregroundPainter` at the same time.
*(Answer: True. `painter` draws behind the child, and `foregroundPainter` draws on top of the child.)*

---

### 3. Output Prediction
What happens if you set `PaintingStyle.stroke` but forget to set `strokeWidth`?
* A) The app crashes
* B) Nothing is drawn
* C) A 1-pixel-wide line is drawn (default stroke width)
* D) A filled shape is drawn

*(Correct Answer: C)*

---

### 4. Theory
Explain the difference between `ClipPath` and `CustomPaint`.
*(Answer: `ClipPath` cuts an existing widget into a shape, while `CustomPaint` draws new graphics from scratch on a canvas. `ClipPath` modifies existing content; `CustomPaint` creates new content.)*

---

### 5. Multiple Choice Question (MCQ)
When should you use `foregroundPainter` instead of `painter`?
* A) Always
* B) Never
* C) When you need to draw on top of the child widget
* D) When you need to draw behind the child widget

*(Correct Answer: C)*

---

### 6. Coding Exercise
Write the code to draw a simple circle centered on the canvas with a radius of 50 pixels and a blue fill:
*(Answer:*
```dart
void paint(Canvas canvas, Size size) {
  final paint = Paint()
    ..color = Colors.blue
    ..style = PaintingStyle.fill;

  canvas.drawCircle(
    Offset(size.width / 2, size.height / 2),
    50,
    paint,
  );
}
```
*)*

---

## 📝 Summary

* **CustomPaint is the Canvas:** It provides a drawing surface where you can paint custom graphics.
* **CustomPainter Defines the Art:** Override `paint()` to draw lines, circles, paths, and shapes using the `Canvas` object.
* **Paint Object Controls Style:** Use `Paint` to set color, stroke width, fill vs stroke, and gradients.
* **Canvas Methods Draw the Graphics:** `drawRect()`, `drawCircle()`, `drawLine()`, `drawPath()`, and `drawArc()` are your brush strokes.
* **Responsive Design:** Always use the `size` parameter instead of hardcoded pixel values.
* **Performance:** Return `false` in `shouldRepaint()` for static graphics to avoid unnecessary redraws.
* **Common Use Cases:** Wave backgrounds, charts, graphs, progress rings, signature pads, and game graphics.
