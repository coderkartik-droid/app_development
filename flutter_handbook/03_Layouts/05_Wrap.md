# Wrap

## 📖 Introduction

In your journey through Flutter layouts so far, you have mastered `Row` and `Column`. 

However, `Row` has one major, uncompromising limitation: **it places all its children along a single horizontal line and refuses to break onto a new line.** If you have 10 filter chips, tags, or interest badges, placing them in a `Row` will immediately crash past the right screen boundary, triggering an overflow error.

What if you are building an e-commerce filter screen where users select clothing sizes (`XS`, `S`, `M`, `L`, `XL`, `XXL`), or a music app where users pick 15 favorite genres? You need elements to flow horizontally across the screen, and the moment an item hits the right edge, it should **automatically drop down to the next line**, just like words in a paragraph!

This is exactly what the **`Wrap`** widget does.

The `Wrap` widget displays its children in multiple runs (either horizontally or vertically). When it runs out of space along its main axis, it gracefully wraps to the next line.

---

## 🤔 Why Do We Need It?

Without `Wrap`, creating dynamic multi-line badge collections requires ugly, brittle workarounds:

1. **Unpredictable Data Lengths:** In modern apps, tags and chips come from backend APIs or user inputs (e.g. hashtags, product categories, skill tags). You never know beforehand how many words fit on one line of a user's phone.
2. **Device Width Variance:** Three tags might fit on a tablet in one line, but take three lines on a budget phone. `Wrap` adapts dynamically to any screen width.
3. **Eliminates Manual Math:** Without `Wrap`, you would have to calculate text lengths in Dart, partition lists into separate sub-lists, and manually nest multiple `Row` widgets inside a `Column`. `Wrap` handles all line-breaking math automatically.

---

## 🌍 Real-Life Analogy

Think of **typing text in a word processor** or **arranging pastries on a bakery tray**:

```text
Row (Rigid single-line table):
┌──────┬──────┬──────┬──────┐ 
│ Tag1 │ Tag2 │ Tag3 │ Tag4 │  Tag5 (Falls off the edge and crashes!)
└──────┴──────┴──────┴──────┘

Wrap (Flowing text or pastry tray):
┌──────┐ ┌──────┐ ┌──────┐
│ Tag1 │ │ Tag2 │ │ Tag3 │
└──────┘ └──────┘ └──────┘
┌──────┐ ┌──────┐
│ Tag4 │ │ Tag5 │ <--- Automatically flows to the next line!
└──────┘ └──────┘
```

* When you type words into a document, your cursor moves to the right. When a word is too long to fit before the right margin, the editor automatically drops that word down to the start of the next line.
* In the exact same way, `Wrap` places widgets side by side until the line is full, then starts a fresh row directly beneath it.

---

## 🧠 Internal Working

How does Flutter’s layout engine execute a `Wrap` widget?

```text
Parent (e.g., Container with max width: 350px)
       │
       ▼
     [Wrap] (direction: Axis.horizontal, spacing: 8px, runSpacing: 10px)
       │
       ├─► 1. Measures Child 1 (Width: 80px) -> Fits on Line 1 (Used: 80px)
       ├─► 2. Adds spacing (8px) + Child 2 (Width: 100px) -> Fits on Line 1 (Used: 188px)
       ├─► 3. Adds spacing (8px) + Child 3 (Width: 120px) -> Fits on Line 1 (Used: 316px)
       ├─► 4. Adds spacing (8px) + Child 4 (Width: 90px):
       │      316px + 8px + 90px = 414px > 350px (EXCEEDS MAX WIDTH!)
       │
       ├─► 5. Wrap terminates Line 1 ("Run 1")
       ├─► 6. Adds runSpacing (10px) vertically
       ├─► 7. Places Child 4 at the start of Line 2 ("Run 2")
       │
       ▼
Calculates total bounding box height and renders all lines cleanly
```

### Key Concept: "Runs"
* In a `Wrap`, each line of widgets is called a **"Run"**.
* **`spacing`** is the gap between items inside the **same run** (horizontal gap if horizontal).
* **`runSpacing`** is the gap between **adjacent runs** (vertical gap between lines).

```text
 ┌─────────┐   spacing   ┌─────────┐
 │ Item 1  │ ◄─────────► │ Item 2  │  <─── Run 1
 └─────────┘             └─────────┘
      ▲
      │ runSpacing (vertical gap between runs)
      ▼
 ┌─────────┐             ┌─────────┐
 │ Item 3  │             │ Item 4  │  <─── Run 2
 └─────────┘             └─────────┘
```

---

## ⚙ Syntax

The constructor for `Wrap`:

```dart
Wrap({
  Key? key,
  Axis direction = Axis.horizontal,
  WrapAlignment alignment = WrapAlignment.start,
  double spacing = 0.0,
  WrapAlignment runAlignment = WrapAlignment.start,
  double runSpacing = 0.0,
  WrapCrossAlignment crossAxisAlignment = WrapCrossAlignment.start,
  TextDirection? textDirection,
  VerticalDirection verticalDirection = VerticalDirection.down,
  Clip clipBehavior = Clip.none,
  List<Widget> children = const <Widget>[],
})
```

### Parameter Breakdown:
* **`direction`**: The direction in which children are placed (`Axis.horizontal` is default; `Axis.vertical` flows top-to-bottom and wraps into new columns).
* **`spacing`**: The empty space between consecutive children within the same run.
* **`runSpacing`**: The empty space between separate runs (lines).
* **`alignment`**: How children are aligned within each individual run along the main axis (`start`, `center`, `end`, `spaceBetween`, `spaceAround`, `spaceEvenly`).
* **`runAlignment`**: How the runs themselves are distributed along the cross axis when the `Wrap` has extra space.
* **`crossAxisAlignment`**: How children of varying heights within a single run are aligned relative to each other.

---

## 📋 Important Properties

| Property | Type | Default Value | Purpose |
|---|---|---|---|
| `children` | `List<Widget>` | `const []` | The widgets to be wrapped across lines. |
| `direction` | `Axis` | `Axis.horizontal` | The primary layout axis (`horizontal` wraps lines; `vertical` wraps columns). |
| `spacing` | `double` | `0.0` | Gap between items in the same line. |
| `runSpacing` | `double` | `0.0` | Gap between consecutive lines. |
| `alignment` | `WrapAlignment` | `WrapAlignment.start` | Main-axis alignment of items inside each line. |
| `runAlignment` | `WrapAlignment` | `WrapAlignment.start` | Cross-axis distribution of the runs. |
| `crossAxisAlignment` | `WrapCrossAlignment`| `WrapCrossAlignment.start` | Vertical alignment of items within the same line. |

---

## 💻 Complete Runnable Example

Below is a complete, standalone Flutter application showcasing `Wrap` for interactive filter chips, skills tags, and photo thumbnail badges.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const WrapDemoApp());
}

class WrapDemoApp extends StatelessWidget {
  const WrapDemoApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: WrapDemoScreen(),
    );
  }
}

class WrapDemoScreen extends StatefulWidget {
  const WrapDemoScreen({super.key});

  @override
  State<WrapDemoScreen> createState() => _WrapDemoScreenState();
}

class _WrapDemoScreenState extends State<WrapDemoScreen> {
  // Set to track selected filter chips
  final Set<String> _selectedFilters = {'Flutter', 'Dart'};

  final List<String> _availableSkills = [
    'Flutter',
    'Dart',
    'Firebase',
    'State Management',
    'REST APIs',
    'GraphQL',
    'Git & GitHub',
    'CI/CD Pipelines',
    'Clean Architecture',
    'Unit Testing',
    'SQLite',
    'UI/UX Design',
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFFF8FAFC),
      appBar: AppBar(
        title: const Text(
          'Wrap Widget Deep Dive',
          style: TextStyle(fontWeight: FontWeight.bold),
        ),
        backgroundColor: Colors.indigo.shade600,
        foregroundColor: Colors.white,
        elevation: 2,
      ),
      body: SafeArea(
        child: SingleChildScrollView(
          padding: const EdgeInsets.all(16.0),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              _buildSectionHeader('1. Interactive Filter Chips (Auto-Wrapping)'),
              const Text(
                'Tap chips to toggle. Notice how lines wrap dynamically on different widths:',
                style: TextStyle(fontSize: 13, color: Colors.grey),
              ),
              const SizedBox(height: 12),

              // Interactive Wrap Container
              Container(
                width: double.infinity,
                padding: const EdgeInsets.all(16),
                decoration: BoxDecoration(
                  color: Colors.white,
                  borderRadius: BorderRadius.circular(12),
                  border: Border.all(color: Colors.grey.shade300),
                ),
                child: Wrap(
                  spacing: 8.0, // Gap between chips on the same line
                  runSpacing: 10.0, // Gap between consecutive lines
                  children: _availableSkills.map((skill) {
                    final isSelected = _selectedFilters.contains(skill);
                    return FilterChip(
                      label: Text(skill),
                      selected: isSelected,
                      selectedColor: Colors.indigo.shade100,
                      checkmarkColor: Colors.indigo.shade700,
                      labelStyle: TextStyle(
                        color: isSelected ? Colors.indigo.shade800 : Colors.black87,
                        fontWeight: isSelected ? FontWeight.bold : FontWeight.normal,
                      ),
                      onSelected: (selected) {
                        setState(() {
                          if (selected) {
                            _selectedFilters.add(skill);
                          } else {
                            _selectedFilters.remove(skill);
                          }
                        });
                      },
                    );
                  }).toList(),
                ),
              ),

              const SizedBox(height: 28),

              _buildSectionHeader('2. Centered Social Action Badges'),
              Container(
                width: double.infinity,
                padding: const EdgeInsets.all(16),
                decoration: BoxDecoration(
                  color: Colors.white,
                  borderRadius: BorderRadius.circular(12),
                  border: Border.all(color: Colors.grey.shade300),
                ),
                child: Wrap(
                  alignment: WrapAlignment.center, // Centers chips in each run
                  spacing: 12.0,
                  runSpacing: 12.0,
                  children: [
                    _buildBadge(Icons.star, '4.9 Rating', Colors.amber),
                    _buildBadge(Icons.download, '10M+ Downloads', Colors.blue),
                    _buildBadge(Icons.security, 'Verified Safe', Colors.green),
                    _buildBadge(Icons.bolt, 'Fast Delivery', Colors.orange),
                    _buildBadge(Icons.support_agent, '24/7 Support', Colors.purple),
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
    return Text(
      title,
      style: const TextStyle(fontSize: 15, fontWeight: FontWeight.bold),
    );
  }

  static Widget _buildBadge(IconData icon, String text, Color color) {
    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 8),
      decoration: BoxDecoration(
        color: color.withValues(alpha: 0.12),
        borderRadius: BorderRadius.circular(20),
        border: Border.all(color: color.withValues(alpha: 0.5)),
      ),
      child: Row(
        mainAxisSize: MainAxisSize.min,
        children: [
          Icon(icon, size: 16, color: color),
          const SizedBox(width: 6),
          Text(
            text,
            style: TextStyle(
              color: color.shade800 ?? color,
              fontWeight: FontWeight.bold,
              fontSize: 12,
            ),
          ),
        ],
      ),
    );
  }
}

extension ColorShade on Color {
  Color? get shade800 => this is MaterialColor ? (this as MaterialColor)[800] : this;
}
```

---

## 🔍 Example Explanation

### 1. `Wrap(spacing: 8.0, runSpacing: 10.0, children: [...])`
* **What it does:** Lays out 12 `FilterChip` widgets in horizontal runs, automatically breaking onto a second, third, or fourth line whenever a chip cannot fit on the current row.
* **Why it exists:** A `Row` would immediately fail with an overflow error. `Wrap` gives each chip its natural size and flows seamlessly.
* **What Flutter does internally:** Measures the width of each chip. Accumulates width + spacing until reaching device width. When exceeded, increments vertical offset by `runSpacing` + tallest chip in that line, and resumes at `X = 0`.
* **What happens if removed:** If replaced with a `Row`, 8 of the 12 chips will disappear off screen accompanied by bright yellow/black warning stripes.

### 2. `alignment: WrapAlignment.center`
* **What it does:** Centers all items within each individual line of the Wrap.
* **Why it exists:** When a line contains only two badges (such as the last row), centering gives the badges an aesthetically balanced look rather than leaving them awkwardly stuck on the left edge.
* **What Flutter does internally:** Calculates empty space on that specific line and divides it equally on the left and right sides of that line's children.
* **What happens if removed:** Defaults to `WrapAlignment.start`, left-aligning every row.

### 3. `_availableSkills.map((skill) => FilterChip(...)).toList()`
* **What it does:** Dynamically transforms a Dart list of strings into a list of Flutter widgets.
* **Why it exists:** In production apps, tags are dynamic data fetched from servers; hardcoding widgets is impractical.
* **What Flutter does internally:** Builds a dynamic widget tree matching the collection length.
* **What happens if removed:** Static widgets would have to be manually typed.

---

## 📱 Phone Screen Preview

```text
+------------------------------------------+
| Wrap Widget Deep Dive                    |
+------------------------------------------+
| 1. Interactive Filter Chips              |
| +--------------------------------------+ |
| | [✔ Flutter] [✔ Dart] [Firebase]     | |
| | [State Management] [REST APIs]       | |
| | [GraphQL] [Git & GitHub]             | |
| | [CI/CD Pipelines] [Clean Arch]       | |
| | [Unit Testing] [SQLite] [UI/UX]      | |
| +--------------------------------------+ |
|                                          |
| 2. Centered Social Action Badges         |
| +--------------------------------------+ |
| |   [⭐ 4.9 Rating]  [⬇ 10M+ Downloads]  | |
| |   [🛡 Verified Safe] [⚡ Fast Delivery]| |
| |           [🎧 24/7 Support]           | |
| +--------------------------------------+ |
+------------------------------------------+
```

### What the user sees:
* A flowing grid of filter chips that wraps neatly across 4 to 5 lines depending on device width.
* Tapping a chip toggles its checkmark and indigo highlight.
* A bottom container where 5 badges wrap across 3 lines with each line gracefully centered.

---

## 🌳 Widget Tree

```text
WrapDemoApp
└── MaterialApp
    └── WrapDemoScreen (StatefulWidget)
        └── Scaffold
            ├── AppBar
            │   └── Text ("Wrap Widget Deep Dive")
            └── SafeArea
                └── SingleChildScrollView
                    └── Column
                        ├── Text ("1. Interactive Filter Chips...")
                        ├── Container
                        │   └── Wrap (spacing: 8.0, runSpacing: 10.0)
                        │       ├── FilterChip ("Flutter")
                        │       ├── FilterChip ("Dart")
                        │       ├── FilterChip ("Firebase")
                        │       └── ... (remaining FilterChips)
                        ├── SizedBox (height: 28)
                        ├── Text ("2. Centered Social Badges...")
                        └── Container
                            └── Wrap (alignment: center, spacing: 12, runSpacing: 12)
                                ├── Container (4.9 Rating)
                                ├── Container (10M+ Downloads)
                                ├── Container (Verified Safe)
                                ├── Container (Fast Delivery)
                                └── Container (24/7 Support)
```

---

## ⚙ Flutter Execution Flow

```text
1. Wrap receives max width constraint from parent Container (e.g. 358px)
   │
2. Wrap begins Run 1 at coordinate (0, 0)
   │
3. Chip 1 ("Flutter") measured: 75px wide -> Placed at X: 0
   │
4. Spacing (8px) added -> Next X: 83px
   │
5. Chip 2 ("Dart") measured: 65px wide -> Fits on Run 1 -> Placed at X: 83px
   │
6. Repeats until a chip exceeds 358px boundary
   │
7. Wrap starts Run 2 at Y: (height of tallest chip in Run 1) + runSpacing (10px)
   │
8. Resets X to 0 (or centers if alignment: WrapAlignment.center)
   │
9. Continues until all children in the list are placed across lines
   │
10. Wrap computes its total height and reports size back up to parent Column
```

---

## ⚠ Common Beginner Mistakes

### Mistake 1: Using `Row` instead of `Wrap` for Dynamic Lists of Chips

**Wrong Code:**
```dart
// ❌ CRASH: Overflows screen right edge on small devices!
Row(
  children: [
    Chip(label: Text('Flutter')),
    Chip(label: Text('Dart')),
    Chip(label: Text('State Management')),
    Chip(label: Text('Clean Architecture')),
  ],
)
```

**Why it is wrong:**
`Row` cannot wrap. As soon as the chips exceed the screen width, a yellow-and-black striped overflow error appears.

**Correct Code:**
```dart
// ✅ CORRECT: Wrap flows naturally across multiple lines
Wrap(
  spacing: 8,
  runSpacing: 8,
  children: const [
    Chip(label: Text('Flutter')),
    Chip(label: Text('Dart')),
    Chip(label: Text('State Management')),
    Chip(label: Text('Clean Architecture')),
  ],
)
```

---

### Mistake 2: Placing `Expanded` or `Spacer` inside a `Wrap`

**Wrong Code:**
```dart
// ❌ RUNTIME CRASH: Expanded cannot be a child of Wrap!
Wrap(
  children: [
    Chip(label: Text('Tag 1')),
    Expanded(child: Chip(label: Text('Tag 2'))),
  ],
)
```

**Why it is wrong:**
`Expanded` and `Spacer` only function inside a `Flex` widget (`Row`, `Column`, or `Flex`). A `Wrap` has dynamic line lengths and cannot calculate flex ratios across runs.

**Correct Code:**
```dart
// ✅ CORRECT: Let children inside Wrap have intrinsic sizes
Wrap(
  spacing: 8,
  children: const [
    Chip(label: Text('Tag 1')),
    Chip(label: Text('Tag 2')),
  ],
)
```

---

## 🚀 Professional Usage

`Wrap` is standard across leading consumer applications:

* **Amazon / E-commerce (Product Filter Drawer):**
  Size selectors (`S`, `M`, `L`, `XL`), color swatches, and department filters are rendered using `Wrap` so they adjust naturally across all phone and tablet widths.
* **YouTube / Spotify (Category Topic Pills):**
  The top horizontal filter tags (`"All"`, `"Podcasts"`, `"Gaming"`, `"Live"`, `"Dart"`, `"Computer Science"`) often use `Wrap` inside bottom sheet filter dialogs.
* **Instagram / Social Apps (Hashtag Collections):**
  Displays user hashtags (`#flutter`, `#mobiledev`, `#google`, `#coding`) beneath a post caption, wrapping cleanly into a paragraph-like block.
* **School ERP / Job Portal (Skills & Hobbies):**
  Candidate skill tags on resumes (`"Flutter"`, `"Python"`, `"Machine Learning"`, `"SQL"`, `"AWS"`).
* **Food Delivery Apps (Dietary Preference Tags):**
  Tags like `"Vegan"`, `"Gluten-Free"`, `"Halal"`, `"Nut-Free"`, `"Organic"`.

---

## 💡 Best Practices

1. **Always Set Both `spacing` and `runSpacing`:** Always specify explicit values for both properties (e.g., `spacing: 8, runSpacing: 8`). By default both are `0.0`, which causes chips to touch each other and look cramped.
2. **Combine with `FilterChip` or `ActionChip`:** `Wrap` is the natural home for Material design chips.
3. **Use inside `SingleChildScrollView`:** If your `Wrap` can contain dozens of tags from a backend database, always ensure the enclosing page or modal is scrollable.
4. **Avoid Massive Lists:** If you have 2,000 items, do not render them in a `Wrap` all at once because `Wrap` does not have a lazy-loading `.builder` constructor. For hundreds of items, use a `GridView.builder`.

---

## 🧪 Try It Yourself

1. Copy the runnable example above into your Flutter project.
2. In Section 2 (`Social Action Badges`), change `alignment: WrapAlignment.center` to:
   * `WrapAlignment.spaceBetween`
   * `WrapAlignment.end`
3. Hot reload the app and observe how the last line of badges re-aligns itself.

---

## 🎯 Challenge

Build an **E-Commerce Clothing Size Selector**:
* Available sizes: `["XXS", "XS", "S", "M", "L", "XL", "XXL", "3XL", "Custom Size"]`.
* Use a `Wrap` with `spacing: 10` and `runSpacing: 10`.
* Each size should be a rounded square box (`width: 50, height: 50`) with the size text centered.
* Allow the user to tap to select a size (highlight the selected box with a black border and dark background).
* Verify that on narrow screens it wraps onto 3 lines and on wider screens it condenses onto 2 lines.

---

## ❓ Practice Questions

### 1. Multiple Choice Question (MCQ)
What is the primary difference between `Row` and `Wrap`?
* A) `Wrap` only works with text
* B) `Row` wraps children to the next line, while `Wrap` does not
* C) `Wrap` automatically breaks children onto a new line when horizontal space runs out, while `Row` overflows
* D) `Wrap` can only have 3 children

*(Correct Answer: C)*

---

### 2. True or False
**Statement:** Placing an `Expanded` widget inside a `Wrap` will make that child stretch to fill the rest of the current line.
*(Answer: False. `Expanded` cannot be used inside a `Wrap`; doing so causes a runtime exception).*

---

### 3. Theory
Explain the difference between `spacing` and `runSpacing` in a `Wrap` widget.
*(Answer: `spacing` is the gap between adjacent children along the main axis within the same line/run. `runSpacing` is the gap between consecutive lines/runs along the cross axis).*

---

### 4. Output Prediction
What happens if you have a `Wrap` with `direction: Axis.vertical`?
*(Answer: The `Wrap` will lay out children from top to bottom in a vertical column, and when it runs out of vertical space, it will wrap into a new column to the right).*

---

### 5. Multiple Choice Question (MCQ)
Which property controls how children are aligned within each individual run?
* A) `runAlignment`
* B) `crossAxisAlignment`
* C) `alignment`
* D) `mainAxisSize`

*(Correct Answer: C)*

---

### 6. Coding Exercise
Write the code for a `Wrap` widget that displays three text chips with 8px horizontal spacing and 12px vertical line spacing.
*(Answer:*
```dart
Wrap(
  spacing: 8.0,
  runSpacing: 12.0,
  children: const [
    Chip(label: Text('Chip 1')),
    Chip(label: Text('Chip 2')),
    Chip(label: Text('Chip 3')),
  ],
)
```
*)*

---

### 7. Theory
Why should you NOT use `Wrap` to render a list of 5,000 items?
*(Answer: Because `Wrap` does not lazily build or recycle off-screen items. It measures and builds every single child simultaneously in memory, causing severe performance lag and memory spikes. Use `GridView.builder` instead).*

---

### 8. Output Prediction
If a `Wrap` has 4 children each 100px wide, with `spacing: 10px`, inside a container of width 350px, how many runs (lines) will be created?
*(Answer: Child 1 (100) + gap (10) + Child 2 (100) + gap (10) + Child 3 (100) = 320px <= 350px. Child 4 cannot fit (320 + 10 + 100 = 430px > 350px). Therefore, Child 4 wraps to a second line. Exactly **2 runs**).*

---

### 9. Multiple Choice Question (MCQ)
What is the default value of the `direction` property in a `Wrap` widget?
* A) `Axis.vertical`
* B) `Axis.horizontal`
* C) `Axis.diagonal`
* D) `Axis.both`

*(Correct Answer: B)*

---

### 10. True or False
**Statement:** `Wrap` supports `alignment: WrapAlignment.spaceBetween`.
*(Answer: True. Just like `Row`, `Wrap` can distribute remaining space on each line between the items).*

---

## 📝 Summary

* **Core Purpose:** `Wrap` arranges children in multiple runs (lines or columns), automatically wrapping to the next line when space runs out.
* **Solves the Row Overflow Problem:** Eliminates `RenderFlex overflowed` errors for dynamic content like chips, tags, and category badges.
* **Two Key Spacing Controls:** `spacing` (space between items on the same line) and `runSpacing` (space between lines).
* **Alignment Options:** Supports `WrapAlignment.start`, `center`, `end`, `spaceBetween`, `spaceAround`, and `spaceEvenly` per line.
* **No `Expanded` Allowed:** Children must have intrinsic sizes; `Expanded` and `Spacer` are not valid children inside `Wrap`.
* **Memory Notice:** Instantiates all children upfront; for thousands of items, prefer a virtualized `GridView.builder`.
