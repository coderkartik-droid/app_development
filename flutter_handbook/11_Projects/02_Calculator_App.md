# Project Overview

Build a basic four-operation calculator. The example supports addition, subtraction, multiplication, and division with a clear action. It uses a deliberately simple two-operand interaction so the lesson can focus on state and safe numeric handling.

# Features

- Two numeric input fields
- Add, subtract, multiply, and divide operations
- Validation for empty and invalid input
- Division-by-zero protection
- Result shown in the interface

# Folder Structure

```text
lib/
  main.dart
  calculator/
    calculator_page.dart
    calculator_service.dart
```

The runnable version keeps the service logic in the page. Extract `calculate` to a pure service and test it independently as the app grows.

# ASCII UI Preview

```text
+--------------------------+
| Calculator               |
| First number  [      ]   |
| Second number [      ]   |
| [+] [-] [×] [÷]         |
| Result: 15               |
+--------------------------+
```

# Step-by-Step Development Plan

1. Create controllers for the two input fields.
2. Parse input and validate it before arithmetic.
3. Keep the selected operation explicit.
4. Handle division by zero and non-finite results.
5. Dispose controllers when the screen is removed.

# Complete Runnable Code

Place this in `lib/main.dart` of a Flutter project. No additional package is needed.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const CalculatorApp());

class CalculatorApp extends StatelessWidget {
  const CalculatorApp({super.key});

  @override
  Widget build(BuildContext context) => MaterialApp(
        title: 'Calculator',
        theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
        home: const CalculatorPage(),
      );
}

class CalculatorPage extends StatefulWidget {
  const CalculatorPage({super.key});

  @override
  State<CalculatorPage> createState() => _CalculatorPageState();
}

class _CalculatorPageState extends State<CalculatorPage> {
  final _firstController = TextEditingController();
  final _secondController = TextEditingController();
  String _result = 'Enter two numbers';

  void _calculate(String operation) {
    final first = double.tryParse(_firstController.text.trim());
    final second = double.tryParse(_secondController.text.trim());
    if (first == null || second == null) {
      setState(() => _result = 'Enter two valid numbers');
      return;
    }
    if (operation == '÷' && second == 0) {
      setState(() => _result = 'Cannot divide by zero');
      return;
    }

    final value = switch (operation) {
      '+' => first + second,
      '-' => first - second,
      '×' => first * second,
      '÷' => first / second,
      _ => throw ArgumentError.value(operation, 'operation'),
    };
    setState(() => _result = 'Result: $value');
  }

  @override
  void dispose() {
    _firstController.dispose();
    _secondController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Calculator')),
        body: ListView(
          padding: const EdgeInsets.all(20),
          children: [
            TextField(
              controller: _firstController,
              keyboardType: const TextInputType.numberWithOptions(decimal: true),
              decoration: const InputDecoration(labelText: 'First number'),
            ),
            TextField(
              controller: _secondController,
              keyboardType: const TextInputType.numberWithOptions(decimal: true),
              decoration: const InputDecoration(labelText: 'Second number'),
            ),
            const SizedBox(height: 20),
            Wrap(
              spacing: 8,
              children: ['+', '-', '×', '÷']
                  .map((symbol) => FilledButton(
                        onPressed: () => _calculate(symbol),
                        child: Text(symbol),
                      ))
                  .toList(),
            ),
            const SizedBox(height: 24),
            Semantics(
              liveRegion: true,
              child: Text(_result, style: Theme.of(context).textTheme.titleLarge),
            ),
          ],
        ),
      );
}
```

# Line-by-Line Explanation

- The app entry point builds `CalculatorApp`, which configures theme and home page.
- `CalculatorPage` is stateful because input controllers and result text change.
- The controllers expose current text; `double.tryParse` returns `null` for invalid input instead of throwing.
- The division guard runs before arithmetic, preventing an invalid result.
- The `switch` maps a known operation to its arithmetic expression and rejects unsupported values.
- `setState` updates the result label and schedules a Flutter rebuild.
- `dispose` releases both controllers; omitting it can retain resources unnecessarily.
- `TextField` specifies decimal-capable numeric keyboards but still validates input because keyboard hints do not guarantee valid values.
- `Wrap` allows operator buttons to move onto another line at narrow widths.
- `Semantics(liveRegion: true)` helps announce result changes to assistive technology.

# Widget Tree

```text
CalculatorApp > MaterialApp > CalculatorPage > Scaffold
├── AppBar
└── ListView
    ├── First TextField
    ├── Second TextField
    ├── Wrap of operation buttons
    └── Accessible result text
```

# State Management Used

Local `StatefulWidget` state is enough for one calculator screen. A shared state framework would add complexity without a real cross-screen state requirement.

# API/Database Used (if applicable)

None. The calculator operates entirely on user-provided values and stores no history.

# Common Mistakes

- Calling `double.parse` on unchecked input.
- Dividing by zero.
- Assuming a numeric keyboard prevents pasted invalid text.
- Forgetting to dispose text controllers.
- Comparing floating-point values for exact financial calculations.

# Improvements

- Add keyboard submission and a clear button.
- Format results with a deliberate precision policy.
- Extract and unit-test a pure calculator function.
- Use decimal arithmetic if extending this to money.

# Challenge Version

Build a full expression calculator with operator precedence, parentheses, backspace, keyboard support, and a tested expression parser. Do not evaluate arbitrary user strings as code.

# Summary

The app demonstrates input validation, predictable operation selection, local state, controller lifecycle, and accessible result updates.
