# Practice Questions: Flutter Layouts

## 📖 Introduction

You have arrived at the final self-assessment chapter of the Layouts module.

In software engineering, reading code and understanding theory is only the first step. True mastery is demonstrated when you can:
1. Diagnose a layout bug without running the debugger.
2. Predict the exact pixel behavior of constraints before hot reloading.
3. Choose the optimal layout widget for any design specification instantly.
4. Write clean, responsive, production-ready Flutter code from scratch.

This chapter contains a curated suite of exam-grade practice questions, output prediction puzzles, architectural theory challenges, and hands-on coding exercises covering the entire `03_Layouts` curriculum.

---

## 🤔 Why Do We Need It?

Without rigorous testing and problem-solving practice, knowledge remains superficial:

1. **Debugging Muscle Memory:** Encountering layout exceptions (`RenderFlex overflowed`, `unbounded height`, `unbounded width`) in practice questions prepares you to fix them effortlessly in high-stakes job interviews and production environments.
2. **Interview Readiness:** Technical Flutter interviewers at top tech companies (Google, ByteDance, BMW, Grab) love grilling candidates on Flutter’s layout protocol, sliver mechanics, and flex ratios.
3. **Reinforcing Mental Models:** Explaining *why* code fails cements the "Constraints go down, Sizes go up, Parent sets position" principle in your subconscious mind.

---

## 🌍 Real-Life Analogy

Think of **a flight simulator for an airplane pilot**:

```text
 ┌────────────────────────────────────────────────────────┐
 │               Layout Flight Simulator                  │
 │                                                        │
 │  🚨 Warning: Unbounded Height Alarm in Engine 2!      │
 │  ⚡ CrossAxis Contention Detected in Sector 4!        │
 │                                                        │
 │  [ Pilot Action: Apply Expanded & Bounded Viewport ]   │
 │                                                        │
 │  ✅ Flight Status: Smooth 120fps Cruising Altitude     │
 └────────────────────────────────────────────────────────┘
```

* A pilot doesn't wait for an engine failure with 300 passengers on board to learn how to handle an emergency; they practice in a safe simulator over and over.
* This chapter is your layout flight simulator: test your understanding against tricky edge cases, fix simulated bugs, and solidify your instincts before building real production apps.

---

## 🧠 Internal Working

When analyzing layout questions, always trace execution through Flutter's **Three Layout Phases**:

```text
                     Phase 1: Constraint Negotiation
Parent passes constraints (min/max width and height) down to each child
                                   │
                                   ▼
                     Phase 2: Intrinsic Sizing
Children compute their content sizes and report sizes back up to the parent
                                   │
                                   ▼
                     Phase 3: Position Assignment
Parent assigns coordinate offsets (X, Y) to children and passes data to Paint phase
```

---

## ⚙ Syntax

Review of key layout syntax structures:

```dart
// 1. Horizontal distribution
Row(mainAxisAlignment: MainAxisAlignment.spaceBetween, children: [...])

// 2. Vertical distribution
Column(crossAxisAlignment: CrossAxisAlignment.stretch, children: [...])

// 3. Proportional space allocation
Expanded(flex: 2, child: myWidget)

// 4. Content-hugging flexibility
Flexible(fit: FlexFit.loose, child: myWidget)

// 5. 2D Depth Layering
Stack(alignment: Alignment.center, children: [BaseWidget(), Positioned(top: 0, child: ...)])

// 6. Line-breaking flow
Wrap(spacing: 8.0, runSpacing: 8.0, children: [...])

// 7. Virtualized scrolling
ListView.builder(itemCount: count, itemBuilder: (context, index) => ...)
```

---

## 📋 Important Properties

### Core Layout Error Checklist:

| Error Message | Typical Cause | Instant Fix |
|---|---|---|
| `A RenderFlex overflowed by XXX pixels on the right` | Long text or wide items inside an unconstrained `Row` | Wrap the text/child in `Expanded` or `Flexible` |
| `A RenderFlex overflowed by XXX pixels on the bottom` | Too many items in a `Column` or keyboard popped up | Wrap the `Column` in a `SingleChildScrollView` |
| `Vertical viewport was given unbounded height` | `ListView` or `GridView` placed directly in a `Column` | Wrap the scrollable in an `Expanded` widget |
| `Horizontal viewport was given unbounded width` | `TextField` or horizontal list placed directly in a `Row` | Wrap in `Expanded` or give explicit `width` |
| `Expanded widgets must be placed inside Flex widgets` | `Expanded` placed in a `Container` or `Stack` | Remove `Expanded` or move it inside a `Row`/`Column` |
| `Cannot provide both a color and a decoration` | Both `color` and `decoration` set on a `Container` | Move `color` inside `BoxDecoration(color: ...)` |

---

## 💻 Complete Runnable Example

Below is a self-testing Interactive Layout Quiz application built in Flutter. You can run it directly to test your layout knowledge interactively on a phone screen.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const LayoutQuizApp());
}

class LayoutQuizApp extends StatelessWidget {
  const LayoutQuizApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: LayoutQuizScreen(),
    );
  }
}

class LayoutQuizScreen extends StatefulWidget {
  const LayoutQuizScreen({super.key});

  @override
  State<LayoutQuizScreen> createState() => _LayoutQuizScreenState();
}

class _LayoutQuizScreenState extends State<LayoutQuizScreen> {
  int _score = 0;
  final Set<int> _answeredQuestions = {};

  final List<QuizItem> _questions = [
    QuizItem(
      question: '1. Which widget is used to prevent keyboard overflow in static forms?',
      options: ['ListView.builder', 'SingleChildScrollView', 'Stack', 'Expanded'],
      correctIndex: 1,
      explanation: 'SingleChildScrollView provides a scrollable viewport for a single document, allowing forms to scroll when keyboards appear.',
    ),
    QuizItem(
      question: '2. What is childAspectRatio in a GridView?',
      options: ['Height / Width', 'Width / Height', 'Number of columns', 'Margin factor'],
      correctIndex: 1,
      explanation: 'childAspectRatio is the ratio of cross-axis to main-axis extent (Width ÷ Height).',
    ),
    QuizItem(
      question: '3. What widget makes tags wrap onto the next line automatically?',
      options: ['Row', 'Column', 'Wrap', 'Stack'],
      correctIndex: 2,
      explanation: 'Wrap displays its children in multiple runs, automatically wrapping when horizontal space runs out.',
    ),
    QuizItem(
      question: '4. What does Expanded render under the hood?',
      options: ['Flexible(fit: FlexFit.loose)', 'Flexible(fit: FlexFit.tight)', 'ConstrainedBox', 'SizedBox'],
      correctIndex: 1,
      explanation: 'Expanded is simply a subclass of Flexible with fit permanently locked to FlexFit.tight.',
    ),
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFFF1F5F9),
      appBar: AppBar(
        title: const Text(
          'Layout Mastery Self-Check',
          style: TextStyle(fontWeight: FontWeight.bold),
        ),
        backgroundColor: Colors.blueGrey.shade900,
        foregroundColor: Colors.white,
        elevation: 2,
      ),
      body: SafeArea(
        child: Column(
          children: [
            // Score Banner
            Container(
              padding: const EdgeInsets.all(16),
              color: Colors.blueGrey.shade800,
              child: Row(
                mainAxisAlignment: MainAxisAlignment.spaceBetween,
                children: [
                  Text(
                    'Score: $_score / ${_questions.length}',
                    style: const TextStyle(
                      color: Colors.white,
                      fontWeight: FontWeight.bold,
                      fontSize: 16,
                    ),
                  ),
                  Text(
                    'Answered: ${_answeredQuestions.length} of ${_questions.length}',
                    style: const TextStyle(color: Colors.white70, fontSize: 13),
                  ),
                ],
              ),
            ),

            // Quiz Cards List
            Expanded(
              child: ListView.builder(
                padding: const EdgeInsets.all(16),
                itemCount: _questions.length,
                itemBuilder: (context, qIndex) {
                  final q = _questions[qIndex];
                  final isAnswered = _answeredQuestions.contains(qIndex);

                  return Container(
                    margin: const EdgeInsets.only(bottom: 16),
                    padding: const EdgeInsets.all(16),
                    decoration: BoxDecoration(
                      color: Colors.white,
                      borderRadius: BorderRadius.circular(12),
                      border: Border.all(color: Colors.grey.shade300),
                    ),
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        Text(
                          q.question,
                          style: const TextStyle(fontSize: 15, fontWeight: FontWeight.bold),
                        ),
                        const SizedBox(height: 12),
                        ...List.generate(q.options.length, (optIndex) {
                          return Padding(
                            padding: const EdgeInsets.only(bottom: 6.0),
                            child: SizedBox(
                              width: double.infinity,
                              child: OutlinedButton(
                                style: OutlinedButton.styleFrom(
                                  alignment: Alignment.centerLeft,
                                  padding: const EdgeInsets.symmetric(horizontal: 14, vertical: 12),
                                  backgroundColor: isAnswered
                                      ? (optIndex == q.correctIndex
                                          ? Colors.green.shade50
                                          : Colors.transparent)
                                      : Colors.transparent,
                                  side: BorderSide(
                                    color: isAnswered
                                        ? (optIndex == q.correctIndex
                                            ? Colors.green
                                            : Colors.grey.shade300)
                                        : Colors.grey.shade300,
                                  ),
                                ),
                                onPressed: isAnswered
                                    ? null
                                    : () {
                                        setState(() {
                                          _answeredQuestions.add(qIndex);
                                          if (optIndex == q.correctIndex) {
                                            _score++;
                                          }
                                        });
                                      },
                                child: Text(
                                  q.options[optIndex],
                                  style: TextStyle(
                                    color: isAnswered
                                        ? (optIndex == q.correctIndex
                                            ? Colors.green.shade800
                                            : Colors.grey)
                                        : Colors.black87,
                                  ),
                                ),
                              ),
                            ),
                          );
                        }),
                        if (isAnswered) ...[
                          const SizedBox(height: 8),
                          Text(
                            '💡 ${q.explanation}',
                            style: TextStyle(
                              fontSize: 12,
                              color: Colors.blueGrey.shade700,
                              fontStyle: FontStyle.italic,
                            ),
                          ),
                        ],
                      ],
                    ),
                  );
                },
              ),
            ),
          ],
        ),
      ),
    );
  }
}

class QuizItem {
  final String question;
  final List<String> options;
  final int correctIndex;
  final String explanation;

  QuizItem({
    required this.question,
    required this.options,
    required this.correctIndex,
    required this.explanation,
  });
}
```

---

## 🔍 Example Explanation

### 1. `Score: $_score / ${_questions.length}` Banner
* **What it does:** Displays an interactive scorecard at the top using `Row` and `MainAxisAlignment.spaceBetween`.
* **Why it exists:** Shows how simple flex layouts provide responsive metadata headers.
* **What Flutter does internally:** Insets children to opposite edges of the blue-grey header container.

### 2. `Expanded(child: ListView.builder(...))`
* **What it does:** Hosts the quiz question cards inside a bounded lazy scroll view.
* **Why it exists:** `ListView.builder` requires bounded vertical constraints.
* **What Flutter does internally:** Gives the list the remaining screen height below the score banner.

### 3. Interactive State & Option Highlighting
* **What it does:** Disables buttons and paints the correct option green upon selection.
* **Why it exists:** Provides instant pedagogical feedback.

---

## 📱 Phone Screen Preview

```text
+------------------------------------------+
| Layout Mastery Self-Check                |
+------------------------------------------+
| Score: 3 / 4             Answered: 3 of 4|
+------------------------------------------+
| 1. Which widget is used to prevent       |
|    keyboard overflow in static forms?    |
|                                          |
| [  ListView.builder                    ] |
| [✔ SingleChildScrollView (Green)       ] |
| [  Stack                               ] |
| [  Expanded                            ] |
| 💡 SingleChildScrollView provides a     |
|    scrollable viewport for a single form.|
| ---------------------------------------- |
| 2. What is childAspectRatio in GridView? |
| [  Height / Width                      ] |
| [✔ Width / Height (Green)              ] |
|                                          |
| (Scrolls through all practice questions) |
+------------------------------------------+
```

---

## 🌳 Widget Tree

```text
LayoutQuizApp
└── MaterialApp
    └── LayoutQuizScreen (StatefulWidget)
        └── Scaffold
            ├── AppBar
            │   └── Text ("Layout Mastery Self-Check")
            └── SafeArea
                └── Column
                    ├── Container (Score Banner)
                    │   └── Row (Score text + Answered text)
                    └── Expanded
                        └── ListView.builder
                            └── Container (Question Card)
                                └── Column
                                    ├── Text (Question title)
                                    ├── OutlinedButton (Option 1)
                                    ├── OutlinedButton (Option 2)
                                    ├── OutlinedButton (Option 3)
                                    └── Text (Explanation)
```

---

## ⚙ Flutter Execution Flow

```text
1. User taps an option button:
   - Evaluates whether selected option index matches correctIndex
   - Increments _score if correct
   - Adds question index to _answeredQuestions set
   │
2. setState() triggers rebuild of the affected card:
   - Correct button turns green with checkmark
   - Explanation text appears below options
   - Score banner re-renders with updated score
   │
3. Viewport updates instantly
```

---

## ⚠ Common Beginner Mistakes

### The Top 3 Layout Trap Questions in Flutter Interviews:

1. **Trap 1:** *"Can you put an `Expanded` inside a `SingleChildScrollView`?"*
   * **Wrong Answer:** *"Yes, to make the item take the rest of the screen."*
   * **Correct Answer:** *"NO! `SingleChildScrollView` gives unbounded height, so `Expanded` has no finite bounds to expand into and causes a layout crash."*

2. **Trap 2:** *"Does `ListView.builder` cache items forever?"*
   * **Wrong Answer:** *"Yes, once built, items stay in memory."*
   * **Correct Answer:** *"NO! When items scroll past the `cacheExtent` boundary, their RenderObjects are disposed and garbage collected to keep memory flat."*

3. **Trap 3:** *"How do you make a `Container` take 100% of the screen width?"*
   * **Wrong Answer:** *"Use `MediaQuery.of(context).size.width`."*
   * **Correct Answer:** *"Use `width: double.infinity` or `SizedBox.expand()`. It is cleaner, faster, and automatically respects parent padding and constraints."*

---

## 🚀 Professional Usage

In production apps, understanding these nuance edge cases prevents critical store release rejections:
* **Banking Apps:** Prevents payment OTP text fields from being covered by on-screen keyboards.
* **E-Commerce:** Ensures product card images don't distort across thousands of device screen densities.
* **Social Feeds:** Ensures feeds maintain smooth 120fps scrolling without dropping frames.

---

## 💡 Best Practices

1. **Always Draw Widget Trees First:** When approaching a complex UI design, sketch the tree hierarchy on paper before writing a single line of Dart.
2. **Never Ignore Yellow-and-Black Stripes:** Treat overflow stripes as fatal compiler errors; resolve them immediately using `Expanded`, `Wrap`, or scroll views.
3. **Use the Flutter Inspector Tool:** In VS Code and Android Studio, use the Flutter DevTools Widget Inspector to inspect live constraints in real time.

---

## 🧪 Try It Yourself

1. Copy the runnable quiz example above into your Flutter project.
2. Add a new `QuizItem` to the `_questions` list testing your knowledge of `Positioned` vs `Align`.
3. Hot reload the app and test yourself on your newly added question!

---

## 🎯 Challenge

Build a **"Layout Debugger" Sandbox**:
* Create a screen with a toggle switch: `"Toggle Bug"`.
* When the toggle is ON, intentionally render broken code (e.g. an unconstrained `ListView` inside a `Column`).
* When the toggle is OFF, render the corrected code (wrapped in `Expanded`).
* Observe the console error and visual error in the Flutter DevTools inspector.

---

## ❓ Practice Questions

---

### Question 1: Multiple Choice Question (MCQ)
Which widget should you use if you want 10 filter chips to display horizontally, but automatically wrap to line 2 and line 3 when they reach the screen edge?
* A) `Row`
* B) `Column`
* C) `Wrap`
* D) `Stack`

**Answer: C (Wrap)**  
*Explanation:* `Row` places all children along a single line and overflows if content exceeds available width. `Wrap` automatically breaks children onto a new run (line) when space runs out.

---

### Question 2: Output Prediction
Consider the following code snippet:
```dart
Row(
  children: const [
    Text('Fixed'),
    Expanded(
      flex: 2,
      child: Container(color: Colors.blue),
    ),
    Expanded(
      flex: 1,
      child: Container(color: Colors.red),
    ),
  ],
)
```
If the total row width is 360px and the `Text('Fixed')` widget takes 60px:
What will be the exact width of the blue container and the red container?

**Answer:**
* Free space remaining = 360px - 60px = 300px.
* Total flex factor = 2 + 1 = 3.
* **Blue Container:** `(2 / 3) * 300px = 200px`.
* **Red Container:** `(1 / 3) * 300px = 100px`.

---

### Question 3: True or False
**Statement:** Setting `shrinkWrap: true` on a `ListView.builder` with 10,000 items is good for performance because it shrinks the memory footprint.

**Answer: False.**  
*Explanation:* `shrinkWrap: true` forces Flutter to measure the exact height of every single child in memory to calculate total height, completely destroying the virtualization optimization and causing severe UI freeze and frame drops.

---

### Question 4: Theory
Explain the difference between `FlexFit.loose` (in `Flexible`) and `FlexFit.tight` (in `Expanded`).

**Answer:**
* **`FlexFit.loose`:** The child is given loose constraints (`minSize = 0`, `maxSize = availableSpace`). The child is allowed to be smaller than the available space and will hug its own content.
* **`FlexFit.tight`:** The child is given tight constraints (`minSize = maxSize = availableSpace`). The child is forced to stretch and occupy the entire allocated space, regardless of content size.

---

### Question 5: Multiple Choice Question (MCQ)
What is the default value of `childAspectRatio` on a `GridView`?
* A) `0.5`
* B) `1.0` (Square)
* C) `1.5`
* D) `double.infinity`

**Answer: B (1.0)**  
*Explanation:* `childAspectRatio` defaults to `1.0`, which enforces equal width and height, creating square cells.

---

### Question 6: Coding Exercise
Fix the broken code below so that the long text wraps cleanly onto multiple lines without throwing an overflow exception:

```dart
// Broken Code:
Row(
  children: const [
    Icon(Icons.warning),
    Text('This is a very long description that causes an overflow error on narrow phone screens...'),
  ],
)
```

**Answer:**
```dart
// Corrected Code:
Row(
  children: const [
    Icon(Icons.warning),
    SizedBox(width: 8),
    Expanded(
      child: Text('This is a very long description that causes an overflow error on narrow phone screens...'),
    ),
  ],
)
```

---

### Question 7: Theory
Why does placing `Positioned(top: 10, child: ...)` directly inside a `Row` trigger a runtime exception?

**Answer:**  
Because `Positioned` is specifically designed to provide layout data to a `RenderStack`. A `Row` uses a `RenderFlex`, which does not recognize `Positioned` data. `Positioned` can ONLY be placed as an immediate child of a `Stack`.

---

### Question 8: Output Prediction
What will the user see if you execute this code?
```dart
Stack(
  children: [
    Container(width: 200, height: 200, color: Colors.blue),
    Container(width: 200, height: 200, color: Colors.red),
  ],
)
```

**Answer:**
The user will only see a **Red square** (200x200).  
*Explanation:* In Flutter's `Stack`, children are painted in chronological list order. The red container sits at index 1 and is painted directly over the blue container (index 0), completely obscuring it.

---

### Question 9: Multiple Choice Question (MCQ)
Which sliver widget allows you to place a standard non-sliver widget (like a `Card` or `Padding`) inside a `CustomScrollView`?
* A) `SliverBox`
* B) `SliverToBoxAdapter`
* C) `SliverBridge`
* D) `SliverWrapper`

**Answer: B (SliverToBoxAdapter)**  
*Explanation:* `SliverToBoxAdapter` bridges regular `RenderBox` widgets into the sliver layout stream.

---

### Question 10: Coding Exercise
Write the code to create an exact 60x60 circular container with a red background and a white centered heart icon using `Container` or `SizedBox`.

**Answer:**
```dart
Container(
  width: 60,
  height: 60,
  decoration: const BoxDecoration(
    color: Colors.red,
    shape: BoxShape.circle,
  ),
  child: const Center(
    child: Icon(Icons.favorite, color: Colors.white, size: 28),
  ),
)
```

---

## 📝 Summary

* **Layout Principle:** Always remember *"Constraints go down, Sizes go up, Parent sets position."*
* **Keyboard Safety:** Wrap form screens in `SingleChildScrollView` to prevent keyboard collision overflow.
* **Z-Axis Pinning:** Use `Stack` with `Positioned` for badges and layered overlays.
* **Elastic Pushing:** Use `Spacer` to push widgets to opposite sides of a `Row` or `Column`.
* **Scalability:** Always use `.builder` constructors (`ListView.builder`, `GridView.builder`) when displaying dynamic or large datasets to ensure smooth 120fps performance.
* **Unified Scrolling:** Use `CustomScrollView` with Slivers to coordinate collapsing headers, grids, and lists into one seamless user experience.
