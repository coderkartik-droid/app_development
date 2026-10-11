# Custom Shapes Practice

## 📖 Introduction

You've learned the theory of `ClipPath`, `CustomClipper`, `CustomPaint`, and `CustomPainter`. Now it's time to put that knowledge into practice.

This chapter provides **5 hands-on exercises** ranging from beginner to intermediate difficulty. Each exercise includes:
* **Requirements:** What you need to build.
* **Expected Behavior:** How the final result should look and behave.
* **Hints:** Gentle nudges to guide you in the right direction without giving away the solution.

Solutions are provided in a separate section at the end, but try to solve each exercise on your own first. The real learning happens when you struggle through the problem and figure it out yourself.

---

## 🎯 Exercise 1: Rounded Profile Card (Beginner)

### Requirements
Create a profile card with:
* A circular profile picture (use a colored container with initials).
* A name and email below the profile picture.
* The entire card should have rounded corners (16px border radius).
* A subtle shadow beneath the card.

### Expected Behavior
* The profile picture should be a perfect circle.
* The card should have smooth rounded corners.
* The shadow should make the card appear elevated above the background.
* The layout should be responsive (work on different screen sizes).

### Hints
* Use `ClipOval` for the circular profile picture.
* Use `ClipRRect` or `BoxDecoration` with `borderRadius` for the card.
* Use `BoxShadow` in `BoxDecoration` for the shadow.
* Use `Column` to stack the profile picture, name, and email vertically.

---

## 🎯 Exercise 2: Diagonal Sale Banner (Beginner)

### Requirements
Create a sale banner with:
* A diagonal cut on the right side (slanted edge).
* A gradient background (e.g., red to orange).
* A "50% OFF" text in the center.
* The banner should span the full width of the screen.

### Expected Behavior
* The banner should have a diagonal edge on the right side.
* The gradient should be smooth and visually appealing.
* The text should be centered and clearly visible.
* The banner should be responsive to different screen widths.

### Hints
* Use `ClipPath` with a custom `CustomClipper<Path>`.
* In your clipper, use `lineTo()` to draw the diagonal line.
* Use `LinearGradient` in `BoxDecoration` for the background.
* The diagonal line should go from the top-right corner to a point on the right edge.

---

## 🎯 Exercise 3: Curved Profile Header (Intermediate)

### Requirements
Create a profile header with:
* A gradient background (purple to blue).
* A smooth curved wave at the bottom of the header.
* A profile picture that overlaps the wave (partially inside, partially outside).
* User name and bio text overlaying the gradient.

### Expected Behavior
* The header should have a smooth quadratic bezier curve at the bottom.
* The profile picture should be positioned such that it sits partly inside the gradient and partly outside.
* The wave should look natural and organic, not jagged.
* The layout should be responsive to different screen sizes.

### Hints
* Use `ClipPath` with a custom `CustomClipper<Path>`.
* Use `quadraticBezierTo()` to draw the wave curve.
* Use `Stack` to layer the profile picture on top of the clipped header.
* Position the profile picture using `Positioned` or `Align`.
* Ensure the control point of the bezier curve is below the bottom edge to create the wave depth.

---

## 🎯 Exercise 4: Wave Dashboard Header (Intermediate)

### Requirements
Create a dashboard header with:
* A gradient background (teal to green).
* **Two layered waves** at the bottom (one more transparent than the other).
* A title and subtitle overlaying the gradient.
* The waves should be offset to create depth.

### Expected Behavior
* The header should have two distinct wave layers.
* The front wave should be more opaque than the back wave.
* The waves should be offset horizontally to create a parallax-like effect.
* The gradient should be smooth and modern.

### Hints
* Use `CustomPaint` with a `CustomPainter` (not `ClipPath`).
* In your painter, draw two separate paths with different colors/opacities.
* Use `quadraticBezierTo()` for each wave.
* Offset the second wave by adjusting the control points.
* Use `withValues(alpha: 0.x)` to set transparency for the waves.

---

## 🎯 Exercise 5: Layered Wave Animation (Advanced)

### Requirements
Create an animated wave header with:
* Three layered waves that animate horizontally.
* Each wave should move at a different speed.
* The waves should loop seamlessly.
* A title overlaying the waves.

### Expected Behavior
* The waves should continuously move from right to left.
* The front wave should move faster than the back waves.
* The animation should be smooth (60fps).
* The waves should loop without visible jumps or glitches.

### Hints
* Use `CustomPaint` with a `CustomPainter`.
* Add an `AnimationController` to your widget.
* Pass the animation value to your painter (make the painter stateful or use a parameter).
* In the painter, offset the wave paths using the animation value.
* Use `modulo` or `remainder` to ensure the waves loop seamlessly.
* Each wave should have a different multiplier for the animation speed.

---

## 💡 Solutions

### Solution 1: Rounded Profile Card

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const RoundedProfileCardApp());
}

class RoundedProfileCardApp extends StatelessWidget {
  const RoundedProfileCardApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: RoundedProfileCardScreen(),
    );
  }
}

class RoundedProfileCardScreen extends StatelessWidget {
  const RoundedProfileCardScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFFF8FAFC),
      body: Center(
        child: Container(
          width: 300,
          padding: const EdgeInsets.all(24),
          decoration: BoxDecoration(
            color: Colors.white,
            borderRadius: BorderRadius.circular(16),
            boxShadow: [
              BoxShadow(
                color: Colors.black.withValues(alpha: 0.1),
                blurRadius: 20,
                offset: const Offset(0, 10),
              ),
            ],
          ),
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              ClipOval(
                child: Container(
                  width: 80,
                  height: 80,
                  color: Colors.indigo.withValues(alpha: 0.2),
                  child: const Center(
                    child: Text(
                      'AJ',
                      style: TextStyle(
                        fontSize: 32,
                        fontWeight: FontWeight.bold,
                        color: Colors.indigo,
                      ),
                    ),
                  ),
                ),
              ),
              const SizedBox(height: 16),
              const Text(
                'Alex Johnson',
                style: TextStyle(
                  fontSize: 20,
                  fontWeight: FontWeight.bold,
                ),
              ),
              const SizedBox(height: 4),
              Text(
                'alex.johnson@email.com',
                style: TextStyle(
                  fontSize: 14,
                  color: Colors.grey.shade600,
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

### Solution 2: Diagonal Sale Banner

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const DiagonalBannerApp());
}

class DiagonalBannerApp extends StatelessWidget {
  const DiagonalBannerApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: DiagonalBannerScreen(),
    );
  }
}

class DiagonalBannerScreen extends StatelessWidget {
  const DiagonalBannerScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFFF8FAFC),
      body: Center(
        child: Padding(
          padding: const EdgeInsets.all(20.0),
          child: ClipPath(
            clipper: DiagonalClipper(),
            child: Container(
              width: double.infinity,
              height: 100,
              decoration: BoxDecoration(
                gradient: LinearGradient(
                  colors: [Colors.red.shade400, Colors.orange.shade400],
                  begin: Alignment.topLeft,
                  end: Alignment.bottomRight,
                ),
              ),
              child: const Center(
                child: Text(
                  '50% OFF SALE',
                  style: TextStyle(
                    color: Colors.white,
                    fontSize: 28,
                    fontWeight: FontWeight.bold,
                  ),
                ),
              ),
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
    path.lineTo(size.width - 50, size.height);
    path.lineTo(0, size.height);
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

### Solution 3: Curved Profile Header

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const CurvedProfileHeaderApp());
}

class CurvedProfileHeaderApp extends StatelessWidget {
  const CurvedProfileHeaderApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: CurvedProfileHeaderScreen(),
    );
  }
}

class CurvedProfileHeaderScreen extends StatelessWidget {
  const CurvedProfileHeaderScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFFF8FAFC),
      body: Column(
        children: [
          // Curved Header
          ClipPath(
            clipper: CurvedHeaderClipper(),
            child: Container(
              width: double.infinity,
              height: 250,
              decoration: const BoxDecoration(
                gradient: LinearGradient(
                  colors: [Color(0xFF6366F1), Color(0xFF3B82F6)],
                  begin: Alignment.topLeft,
                  end: Alignment.bottomRight,
                ),
              ),
              child: SafeArea(
                child: Padding(
                  padding: const EdgeInsets.all(20.0),
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      const Icon(Icons.menu, color: Colors.white, size: 28),
                      const SizedBox(height: 60),
                      const Text(
                        'Alex Johnson',
                        style: TextStyle(
                          color: Colors.white,
                          fontSize: 28,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                      const SizedBox(height: 8),
                      Text(
                        'Flutter Developer',
                        style: TextStyle(
                          color: Colors.white.withValues(alpha: 0.8),
                          fontSize: 14,
                        ),
                      ),
                    ],
                  ),
                ),
              ),
            ),
          ),

          // Profile Picture Overlapping the Wave
          Transform.translate(
            offset: const Offset(0, -50),
            child: Center(
              child: ClipOval(
                child: Container(
                  width: 100,
                  height: 100,
                  color: Colors.white,
                  padding: const EdgeInsets.all(4),
                  child: ClipOval(
                    child: Container(
                      color: Colors.indigo.withValues(alpha: 0.2),
                      child: const Center(
                        child: Text(
                          'AJ',
                          style: TextStyle(
                            fontSize: 40,
                            fontWeight: FontWeight.bold,
                            color: Colors.indigo,
                          ),
                        ),
                      ),
                    ),
                  ),
                ),
              ),
            ),
          ),

          const SizedBox(height: 40),

          // Additional Content
          const Padding(
            padding: EdgeInsets.symmetric(horizontal: 20.0),
            child: Text(
              'About Me',
              style: TextStyle(
                fontSize: 18,
                fontWeight: FontWeight.bold,
              ),
            ),
          ),
          const SizedBox(height: 8),
          const Padding(
            padding: EdgeInsets.symmetric(horizontal: 20.0),
            child: Text(
              'I am a passionate Flutter developer with 5 years of experience building beautiful mobile applications.',
              style: TextStyle(
                fontSize: 14,
                color: Colors.grey,
              ),
            ),
          ),
        ],
      ),
    );
  }
}

class CurvedHeaderClipper extends CustomClipper<Path> {
  @override
  Path getClip(Size size) {
    final path = Path();
    path.moveTo(0, 0);
    path.lineTo(size.width, 0);
    path.lineTo(size.width, size.height - 60);
    path.quadraticBezierTo(
      size.width / 2,
      size.height + 40,
      0,
      size.height - 60,
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

### Solution 4: Wave Dashboard Header

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const WaveDashboardHeaderApp());
}

class WaveDashboardHeaderApp extends StatelessWidget {
  const WaveDashboardHeaderApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: WaveDashboardHeaderScreen(),
    );
  }
}

class WaveDashboardHeaderScreen extends StatelessWidget {
  const WaveDashboardHeaderScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFFF8FAFC),
      body: Column(
        children: [
          // Wave Header using CustomPaint
          CustomPaint(
            size: const Size(double.infinity, 200),
            painter: WaveDashboardPainter(),
            child: SafeArea(
              child: Padding(
                padding: const EdgeInsets.all(20.0),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    const Icon(Icons.menu, color: Colors.white, size: 28),
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
                    Text(
                      'Welcome back, User',
                      style: TextStyle(
                        color: Colors.white.withValues(alpha: 0.8),
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
                    'Quick Stats',
                    style: TextStyle(
                      fontSize: 18,
                      fontWeight: FontWeight.bold,
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

class WaveDashboardPainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    // Gradient Background
    final backgroundPaint = Paint()
      ..shader = LinearGradient(
        colors: [const Color(0xFF14B8A6), const Color(0xFF22C55E)],
        begin: Alignment.topLeft,
        end: Alignment.bottomRight,
      ).createShader(const Rect.fromLTWH(0, 0, 400, 200));

    canvas.drawRect(
      Rect.fromLTWH(0, 0, size.width, size.height),
      backgroundPaint,
    );

    // First Wave (Back, more transparent)
    final wavePaint1 = Paint()
      ..color = Colors.white.withValues(alpha: 0.15)
      ..style = PaintingStyle.fill;

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
    canvas.drawPath(wavePath1, wavePaint1);

    // Second Wave (Front, less transparent, offset)
    final wavePaint2 = Paint()
      ..color = Colors.white.withValues(alpha: 0.25)
      ..style = PaintingStyle.fill;

    final wavePath2 = Path();
    wavePath2.moveTo(0, size.height * 0.75);
    wavePath2.quadraticBezierTo(
      size.width * 0.3,
      size.height * 0.55,
      size.width * 0.6,
      size.height * 0.75,
    );
    wavePath2.quadraticBezierTo(
      size.width * 0.8,
      size.height * 0.95,
      size.width,
      size.height * 0.75,
    );
    wavePath2.lineTo(size.width, size.height);
    wavePath2.lineTo(0, size.height);
    wavePath2.close();
    canvas.drawPath(wavePath2, wavePaint2);
  }

  @override
  bool shouldRepaint(CustomPainter oldDelegate) {
    return false;
  }
}
```

---

### Solution 5: Layered Wave Animation

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const AnimatedWaveApp());
}

class AnimatedWaveApp extends StatelessWidget {
  const AnimatedWaveApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: AnimatedWaveScreen(),
    );
  }
}

class AnimatedWaveScreen extends StatefulWidget {
  const AnimatedWaveScreen({super.key});

  @override
  State<AnimatedWaveScreen> createState() => _AnimatedWaveScreenState();
}

class _AnimatedWaveScreenState extends State<AnimatedWaveScreen>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 3),
    )..repeat();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFFF8FAFC),
      body: Column(
        children: [
          // Animated Wave Header
          AnimatedBuilder(
            animation: _controller,
            builder: (context, child) {
              return CustomPaint(
                size: const Size(double.infinity, 200),
                painter: AnimatedWavePainter(_controller.value),
                child: SafeArea(
                  child: Padding(
                    padding: const EdgeInsets.all(20.0),
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        const Icon(Icons.menu, color: Colors.white, size: 28),
                        const SizedBox(height: 50),
                        const Text(
                          'Animated Waves',
                          style: TextStyle(
                            color: Colors.white,
                            fontSize: 32,
                            fontWeight: FontWeight.bold,
                          ),
                        ),
                        const SizedBox(height: 8),
                        Text(
                          'Smooth 60fps animation',
                          style: TextStyle(
                            color: Colors.white.withValues(alpha: 0.8),
                            fontSize: 14,
                          ),
                        ),
                      ],
                    ),
                  ),
                ),
              );
            },
          ),

          // Content below
          Expanded(
            child: Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  const Icon(
                    Icons.waves,
                    size: 64,
                    color: Colors.blue,
                  ),
                  const SizedBox(height: 16),
                  const Text(
                    'Watch the waves move!',
                    style: TextStyle(
                      fontSize: 18,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                ],
              ),
            ),
          ),
        ],
      ),
    );
  }
}

class AnimatedWavePainter extends CustomPainter {
  final double animationValue;

  AnimatedWavePainter(this.animationValue);

  @override
  void paint(Canvas canvas, Size size) {
    // Gradient Background
    final backgroundPaint = Paint()
      ..shader = LinearGradient(
        colors: [const Color(0xFF3B82F6), const Color(0xFF8B5CF6)],
        begin: Alignment.topLeft,
        end: Alignment.bottomRight,
      ).createShader(const Rect.fromLTWH(0, 0, 400, 200));

    canvas.drawRect(
      Rect.fromLTWH(0, 0, size.width, size.height),
      backgroundPaint,
    );

    // Wave 1 (Back, slowest)
    _drawWave(
      canvas,
      size,
      animationValue * 0.5,
      0.7,
      Colors.white.withValues(alpha: 0.1),
    );

    // Wave 2 (Middle, medium speed)
    _drawWave(
      canvas,
      size,
      animationValue * 0.75,
      0.75,
      Colors.white.withValues(alpha: 0.15),
    );

    // Wave 3 (Front, fastest)
    _drawWave(
      canvas,
      size,
      animationValue,
      0.8,
      Colors.white.withValues(alpha: 0.2),
    );
  }

  void _drawWave(Canvas canvas, Size size, double offset, double heightRatio, Color color) {
    final paint = Paint()
      ..color = color
      ..style = PaintingStyle.fill;

    final path = Path();
    final waveWidth = size.width * 1.5;
    final startX = -size.width * 0.25 + (offset * waveWidth % waveWidth);

    path.moveTo(startX, size.height * heightRatio);

    for (double x = startX; x < startX + waveWidth; x += waveWidth / 2) {
      path.quadraticBezierTo(
        x + waveWidth / 4,
        size.height * (heightRatio - 0.1),
        x + waveWidth / 2,
        size.height * heightRatio,
      );
    }

    path.lineTo(startX + waveWidth, size.height);
    path.lineTo(startX, size.height);
    path.close();

    canvas.drawPath(path, paint);
  }

  @override
  bool shouldRepaint(CustomPainter oldDelegate) {
    return true;
  }
}
```

---

## 🎓 Next Steps

Congratulations on completing these exercises! You now have hands-on experience with:
* Creating circular and rounded shapes using built-in clippers.
* Writing custom clippers for diagonal edges and curved waves.
* Using `CustomPaint` and `CustomPainter` for layered wave effects.
* Animating custom graphics with `AnimationController`.

Continue practicing by:
1. Combining multiple clipping techniques in a single screen.
2. Experimenting with different bezier curve control points.
3. Creating your own unique shapes and animations.
4. Building a complete app screen that uses custom shapes throughout.

Happy coding! 🚀
