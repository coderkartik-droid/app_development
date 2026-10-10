# InputFormatters in Flutter

## 📖 Introduction
Validation stops bad data after it is submitted. Keyboard types suggest friendly layouts to the user. But how do you prevent users from typing invalid characters in the first place? And how do you automatically format text in real time as they type—such as inserting hyphens into phone numbers `(555) 123-4567`, spaces into credit cards `4111 2222 3333 4444`, or converting stock tickers to uppercase automatically?

In Flutter, [TextInputFormatter](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/11_InputFormatters.md) is the live gatekeeper and transformer of text entry. An input formatter intercepts raw keystrokes and clipboard pastes **before** they reach the text editing controller or render on the screen. It can reject prohibited characters instantly, modify strings on the fly, and re-position the text cursor selection precisely.

---

## 🤔 Why Do We Need It?
Without input formatters:
1. **Bad Data Penetration**: Users can paste letters into integer-only fields or bypass soft keyboards via hardware Bluetooth keyboards.
2. **Ugly Text Formatting**: Users are forced to manually type parentheses, dashes, or spaces into credit card or phone fields.
3. **Jarring Validation Errors**: Users get scolded by red validation text for minor typos (like typing a letter in a phone number) instead of the app simply ignoring the invalid character.
4. **Messy State Management**: You would need to write complex controller listener callbacks that mutate strings and fight with cursor jumping issues.

With [TextInputFormatter](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/11_InputFormatters.md):
- **Filtering**: `FilteringTextInputFormatter.digitsOnly` completely ignores non-digit keystrokes.
- **Length Limiting**: `LengthLimitingTextInputFormatter(10)` stops accepting input at 10 characters.
- **Custom Masking**: Custom formatters format raw digits into standard patterns dynamically while maintaining smooth cursor positioning.

---

## 🌍 Real-Life Analogy
Think of [TextInputFormatter](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/11_InputFormatters.md) as a **Security Bouncer and Coin Sorter Conveyor Belt**:

```
+-------------------------------------------------------------+
|                     COIN SORTER CONVEYOR                    |
|                                                             |
|   User dumps coins and foreign tokens onto the belt:        |
|   [$0.25, Button, $0.10, Bottlecap, $1.00]                  |
|                                                             |
|   [ SCREENING FILTER (TextInputFormatter) ]                |
|   - Rejects bottlecaps and buttons instantly (dropped).     |
|   - Allows only genuine coins.                              |
|   - Stacks them into neat rolls of 10 coins automatically.  |
|                                                             |
|   [ SAFE VAULT (TextEditingController) ]                    |
|   Only receives pre-screened, perfectly formatted stacks!   |
+-------------------------------------------------------------+
```

The vault never sees garbage or misaligned tokens; the formatter cleans and arranges the input before it ever lands in the vault.

---

## 🧠 Internal Working
Whenever the user types a character, deletes a letter, or pastes text from the clipboard, Flutter's text input pipeline executes this synchronous transformation chain:

```
                     OS Soft/Hardware Keyboard
                                │
                                ▼
                       Raw User Keystroke
                                │
                                ▼
                   Old TextEditingValue (oldValue)
                   New TextEditingValue (newValue)
                                │
                                ▼
                 TextInputFormatter.formatEditUpdate()
                                │
       ┌────────────────────────┼────────────────────────┐
       ▼                        ▼                        ▼
  [REJECT]                 [TRANSFORM]                [ACCEPT]
  Return oldValue          Format string,             Return newValue
  (Keystroke blocked)      adjust cursor              (Keystroke accepted)
       │                        │                        │
       └────────────────────────┼────────────────────────┘
                                │
                                ▼
                  Updated TextEditingController
                                │
                                ▼
                     Screen Repaints With Text
```

The core contract of every formatter is the method:
```dart
TextEditingValue formatEditUpdate(
  TextEditingValue oldValue,
  TextEditingValue newValue,
);
```
- `oldValue`: The complete text string and cursor selection right before this keystroke.
- `newValue`: The proposed text string and cursor selection including the new keystroke.
- **Return Value**: The final `TextEditingValue` that will actually be applied to the field.

---

## ⚙ Syntax

### Built-in Formatters
```dart
TextField(
  inputFormatters: [
    // 1. Only allow numerical digits
    FilteringTextInputFormatter.digitsOnly,

    // 2. Allow only specific characters via regex
    FilteringTextInputFormatter.allow(RegExp(r'[a-zA-Z\s]')),

    // 3. Deny specific characters (e.g. no spaces or symbols)
    FilteringTextInputFormatter.deny(RegExp(r'[\s]')),

    // 4. Restrict character count
    LengthLimitingTextInputFormatter(16),
  ],
)
```

### Custom Formatter Implementation
```dart
class UpperCaseTextFormatter extends TextInputFormatter {
  @override
  TextEditingValue formatEditUpdate(
    TextEditingValue oldValue,
    TextEditingValue newValue,
  ) {
    return TextEditingValue(
      text: newValue.text.toUpperCase(),
      selection: newValue.selection,
    );
  }
}
```

---

## 📋 Important Built-in Formatters Comparison

| Formatter | Description | Example Usage |
| :--- | :--- | :--- |
| `FilteringTextInputFormatter.digitsOnly` | Permits only decimal digit characters `0-9`. Discards everything else. | OTP boxes, card numbers, age. |
| `FilteringTextInputFormatter.allow(regex)` | Whitelists only characters matching the regular expression. | Name fields (`r'[a-zA-Z\s]'`), alphanumeric IDs. |
| `FilteringTextInputFormatter.deny(regex)` | Blacklists characters matching the regular expression. | Username fields (deny spaces `r'\s'`), email fields. |
| `LengthLimitingTextInputFormatter(max)` | Halts input when text reaches the specified length. | CVV (3 digits), Postal codes. |

---

## 💻 Complete Runnable Example
The following production-grade application demonstrates:
1. **Digits-only filtering** on integer inputs.
2. **Auto-capitalizing stock ticker** formatter.
3. **Real-time Credit Card 4-4-4-4 spacing** formatter with automatic cursor management.
4. **US Phone Number masking** `(XXX) XXX-XXXX`.

```dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() {
  runApp(const FormattersMasterclassApp());
}

class FormattersMasterclassApp extends StatelessWidget {
  const FormattersMasterclassApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'InputFormatters Guide',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.teal),
        useMaterial3: true,
      ),
      home: const FormatterShowcaseScreen(),
    );
  }
}

class FormatterShowcaseScreen extends StatefulWidget {
  const FormatterShowcaseScreen({super.key});

  @override
  State<FormatterShowcaseScreen> createState() => _FormatterShowcaseScreenState();
}

class _FormatterShowcaseScreenState extends State<FormatterShowcaseScreen> {
  final TextEditingController _cardController = TextEditingController();
  final TextEditingController _phoneController = TextEditingController();
  final TextEditingController _tickerController = TextEditingController();

  @override
  void dispose() {
    _cardController.dispose();
    _phoneController.dispose();
    _tickerController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: () => FocusScope.of(context).unfocus(),
      child: Scaffold(
        appBar: AppBar(
          title: const Text('InputFormatters in Action'),
          centerTitle: true,
        ),
        body: SafeArea(
          child: SingleChildScrollView(
            padding: const EdgeInsets.all(24.0),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.stretch,
              children: [
                const Text(
                  'Real-Time Text Formatting',
                  style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
                ),
                const SizedBox(height: 6),
                const Text(
                  'Characters are sanitized, transformed, and masked live during typing.',
                  style: TextStyle(color: Colors.grey),
                ),
                const SizedBox(height: 24),

                // 1. Credit Card Formatter
                const Text('Credit Card (Spaced 4-4-4-4)', style: TextStyle(fontWeight: FontWeight.w600)),
                const SizedBox(height: 6),
                TextField(
                  controller: _cardController,
                  keyboardType: TextInputType.number,
                  inputFormatters: [
                    FilteringTextInputFormatter.digitsOnly,
                    LengthLimitingTextInputFormatter(16),
                    CreditCardNumberFormatter(),
                  ],
                  decoration: const InputDecoration(
                    hintText: '4000 1234 5678 9010',
                    prefixIcon: Icon(Icons.credit_card),
                    border: OutlineInputBorder(),
                  ),
                ),
                const SizedBox(height: 20),

                // 2. US Phone Number Formatter
                const Text('Phone Number Mask (XXX) XXX-XXXX', style: TextStyle(fontWeight: FontWeight.w600)),
                const SizedBox(height: 6),
                TextField(
                  controller: _phoneController,
                  keyboardType: TextInputType.phone,
                  inputFormatters: [
                    FilteringTextInputFormatter.digitsOnly,
                    LengthLimitingTextInputFormatter(10),
                    UsPhoneNumberFormatter(),
                  ],
                  decoration: const InputDecoration(
                    hintText: '(555) 019-2834',
                    prefixIcon: Icon(Icons.phone),
                    border: OutlineInputBorder(),
                  ),
                ),
                const SizedBox(height: 20),

                // 3. Stock Market Ticker Formatter
                const Text('Stock Market Ticker (Auto-Uppercase & Alpha Only)', style: TextStyle(fontWeight: FontWeight.w600)),
                const SizedBox(height: 6),
                TextField(
                  controller: _tickerController,
                  inputFormatters: [
                    FilteringTextInputFormatter.allow(RegExp(r'[a-zA-Z]')),
                    LengthLimitingTextInputFormatter(5),
                    UpperCaseTextFormatter(),
                  ],
                  decoration: const InputDecoration(
                    hintText: 'e.g. AAPL, GOOG',
                    prefixIcon: Icon(Icons.show_chart),
                    border: OutlineInputBorder(),
                  ),
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }
}

/// Custom Formatter: Automatically converts all input to UPPERCASE
class UpperCaseTextFormatter extends TextInputFormatter {
  @override
  TextEditingValue formatEditUpdate(
    TextEditingValue oldValue,
    TextEditingValue newValue,
  ) {
    return TextEditingValue(
      text: newValue.text.toUpperCase(),
      selection: newValue.selection,
    );
  }
}

/// Custom Formatter: Formats 16 digits into "XXXX XXXX XXXX XXXX"
class CreditCardNumberFormatter extends TextInputFormatter {
  @override
  TextEditingValue formatEditUpdate(
    TextEditingValue oldValue,
    TextEditingValue newValue,
  ) {
    final text = newValue.text.replaceAll(' ', '');
    final buffer = StringBuffer();

    for (int i = 0; i < text.length; i++) {
      buffer.write(text[i]);
      final nonZeroIndex = i + 1;
      if (nonZeroIndex % 4 == 0 && nonZeroIndex != text.length) {
        buffer.write(' ');
      }
    }

    final formatted = buffer.toString();
    return TextEditingValue(
      text: formatted,
      selection: TextSelection.collapsed(offset: formatted.length),
    );
  }
}

/// Custom Formatter: Masks 10 digits into "(XXX) XXX-XXXX"
class UsPhoneNumberFormatter extends TextInputFormatter {
  @override
  TextEditingValue formatEditUpdate(
    TextEditingValue oldValue,
    TextEditingValue newValue,
  ) {
    final digits = newValue.text.replaceAll(RegExp(r'\D'), '');
    final buffer = StringBuffer();

    for (int i = 0; i < digits.length; i++) {
      if (i == 0) buffer.write('(');
      if (i == 3) buffer.write(') ');
      if (i == 6) buffer.write('-');
      buffer.write(digits[i]);
    }

    final formatted = buffer.toString();
    return TextEditingValue(
      text: formatted,
      selection: TextSelection.collapsed(offset: formatted.length),
    );
  }
}
```

---

## 🔍 Line-by-Line Example Explanation
- `FilteringTextInputFormatter.digitsOnly`:
  - **What it does**: Strips out all non-digit characters (`0-9`) as keys are pressed or pasted.
  - **Why it exists**: Guarantees that only valid numerals pass through to the custom masking formatters.
- `LengthLimitingTextInputFormatter(16)`:
  - **What it does**: Restricts raw character input to 16 characters.
  - **Why it exists**: Prevents users from typing infinitely long numbers.
- `final text = newValue.text.replaceAll(' ', '');`:
  - **What it does**: Strips existing space characters out of the input string before re-evaluating spacing intervals.
  - **Why it exists**: Avoids accumulating duplicate or runaway spaces when users edit or delete numbers mid-string.
- `selection: TextSelection.collapsed(offset: formatted.length)`:
  - **What it does**: Sets the cursor position to the end of the newly formatted string.
  - **What happens if omitted**: The cursor can jump to index 0 (the beginning of the field), causing subsequent keystrokes to type in reverse!

---

## 📱 ASCII Phone Screen Preview
```
+------------------------------------------+
|         InputFormatters in Action        |
+------------------------------------------+
| Real-Time Text Formatting                |
| Characters are sanitized, transformed... |
|                                          |
| Credit Card (Spaced 4-4-4-4)             |
| +--------------------------------------+ |
| | [💳] 4532 8920 1123 7894|            | |
| +--------------------------------------+ |
|                                          |
| Phone Number Mask (XXX) XXX-XXXX         |
| +--------------------------------------+ |
| | [📞] (555) 234-8901|                 | |
| +--------------------------------------+ |
|                                          |
| Stock Market Ticker (Auto-Uppercase)     |
| +--------------------------------------+ |
| | [📈] TSLA|                           | |
| +--------------------------------------+ |
|                                          |
+------------------------------------------+
```

---

## 🌳 Widget Tree
```
MaterialApp
 └── FormatterShowcaseScreen (StatefulWidget)
      └── GestureDetector (unfocus)
           └── Scaffold
                ├── AppBar
                └── SafeArea
                     └── SingleChildScrollView
                          └── Column
                               ├── Text (Headers)
                               ├── TextField (Card: digitsOnly -> limit -> CreditCardFormatter)
                               ├── TextField (Phone: digitsOnly -> limit -> UsPhoneFormatter)
                               └── TextField (Ticker: allow -> limit -> UpperCaseFormatter)
```

---

## ⚙ Flutter Execution Flow
1. **User Types**: User presses `"9"` on the credit card field.
2. **Formatter 1**: `FilteringTextInputFormatter.digitsOnly` inspects `"9"`. It is a digit, so it passes.
3. **Formatter 2**: `LengthLimitingTextInputFormatter(16)` verifies the length is $\le 16$, so it passes.
4. **Formatter 3**: `CreditCardNumberFormatter` runs `formatEditUpdate()`.
   - Strips existing spaces.
   - Loops through characters, injecting a space after every 4th digit.
   - Adjusts the `TextSelection` cursor position to index `formatted.length`.
5. **UI Update**: `TextField` paints the spaced card string on the canvas.

---

## ⚠ Common Beginner Mistakes

### 1. Cursor Snapping to the Beginning of the Field
```dart
// ❌ BAD: Omitting selection or leaving it at index 0
return TextEditingValue(
  text: formattedText,
  // No selection specified! Defaults to offset 0! Cursor jumps to start!
);

// ✅ GOOD: Preserve or update the cursor selection
return TextEditingValue(
  text: formattedText,
  selection: TextSelection.collapsed(offset: formattedText.length),
);
```

### 2. Formatters in Wrong Execution Order
```dart
// ❌ WRONG ORDER: Formatter adds dashes, then digitsOnly deletes the dashes!
inputFormatters: [
  UsPhoneNumberFormatter(), // Injects '(' and ')' and '-'
  FilteringTextInputFormatter.digitsOnly, // WIPES OUT the parens and dashes!
]

// ✅ CORRECT ORDER: Clean digits first, then format!
inputFormatters: [
  FilteringTextInputFormatter.digitsOnly, // Allows only raw digits
  UsPhoneNumberFormatter(), // Injects formatting cleanly
]
```

### 3. LengthLimiter Cutting Off Formatted Characters
```dart
// ❌ BAD: LengthLimiter limits to 10, but the mask adds 4 punctuation chars!
inputFormatters: [
  UsPhoneNumberFormatter(), // "(555) 123-4567" is 14 chars long!
  LengthLimitingTextInputFormatter(10), // Truncates phone number to "(555) 123-"!
]
```

---

## 🚀 Professional Usage
- **Fintech & Banking (e.g., Revolut, PayPal)**: Formats IBAN codes with spaces every 4 characters (`GB29 NWBK 6016 1331 9268 19`) and forces currency amounts to two decimal places.
- **Flight & Passport Systems (e.g., Delta, Emirates)**: Automatically converts passport numbers to uppercase alphanumeric characters while discarding punctuation.
- **Social Media (e.g., Twitter/X, Instagram)**: Enforces `@` prefixes on handles and denies spaces dynamically.

---

## 💡 Best Practices
1. **Order Matters**: Always place filtering/whitelisting formatters before custom masking formatters.
2. **Handle Backspaces in Custom Formatters**: When users press delete, ensure your formatter doesn't trap them in an infinite loop of re-adding the deleted punctuation.
3. **Store Raw Data**: When submitting the form, strip formatting masks before sending payloads to APIs (e.g., send `5551234567` instead of `(555) 123-4567`).
4. **Use Regex Sparingly in Hot Paths**: Complex regular expressions run on every keystroke; precompile `RegExp` instances as `static final` variables.

---

## 🧪 Try It Yourself
1. Create a custom formatter that formats dates as `DD/MM/YYYY` while typing digits.
2. Build an IBAN bank account formatter that formats text into blocks of 4 uppercase alphanumeric characters.
3. Add backspace tolerance to `UsPhoneNumberFormatter` so deleting past a parenthesis removes the digit behind it.

---

## 🎯 Challenge
Create a dynamic currency input formatter:
- As the user types digits `1` $\to$ `12` $\to$ `1250`, the field formats dynamically from `$0.01` $\to$ `$0.12` $\to$ `$12.50`.
- The cursor stays pinned at the rightmost edge.
- Non-digits are prohibited.

---

## ❓ Practice Questions

### Q1. What is the primary role of a `TextInputFormatter` in Flutter?
- A) To encrypt passwords in memory
- B) To intercept, filter, and transform text input in real-time before it reaches the controller and screen
- C) To render hint text inside text fields
- D) To translate text into different languages
*Answer: B* — Input formatters synchronously intercept and mutate text values on every keystroke or paste.

### Q2. What does returning `oldValue` from `formatEditUpdate()` achieve?
- A) Clears the field completely
- B) Rejects the new keystroke or paste, keeping the field in its previous state
- C) Moves the cursor to index 0
- D) Throws an exception
*Answer: B* — Returning `oldValue` discards the proposed change, effectively blocking the invalid input.

### Q3. Which built-in formatter allows only numeric digits from 0 to 9?
- A) `FilteringTextInputFormatter.numbers`
- B) `FilteringTextInputFormatter.digitsOnly`
- C) `NumericInputFormatter()`
- D) `DigitLimiter()`
*Answer: B* — `digitsOnly` is a preconfigured static instance that permits only `[0-9]`.

### Q4. What happens if you modify the text in a custom formatter but do not adjust `newValue.selection`?
- A) Flutter crashes
- B) The cursor position may jump to the beginning of the text or become misaligned
- C) The text turns invisible
- D) The keyboard closes
*Answer: B* — The cursor must be explicitly positioned via `TextSelection` to match the new string length.

### Q5. What is the difference between `FilteringTextInputFormatter.allow()` and `FilteringTextInputFormatter.deny()`?
- A) `allow` accepts only characters matching the regex; `deny` rejects characters matching the regex
- B) `allow` works on iOS; `deny` works on Android
- C) `allow` requires integers; `deny` requires strings
- D) There is no difference
*Answer: A* — `allow` functions as a whitelist, while `deny` functions as a blacklist.

### Q6. In what order are formatters executed when multiple formatters are provided in the `inputFormatters` list?
*Answer*: They execute sequentially in the exact index order of the list. The output `TextEditingValue` of each formatter becomes the `newValue` input to the next formatter.

### Q7. Why should you place `FilteringTextInputFormatter.digitsOnly` before a phone number masking formatter?
*Answer*: Because the phone number mask introduces non-digit characters (like parentheses, spaces, and hyphens). If `digitsOnly` ran after the mask, it would immediately strip away the formatting punctuation that the mask just added.

### Q8. Output Prediction: What text will be displayed if the user pastes `"A1B2C3"` into a field with `FilteringTextInputFormatter.digitsOnly`?
*Answer*: `"123"`. The letters `A`, `B`, and `C` are stripped out, leaving only the digits.

### Q9. How do you limit a text field to a maximum of 4 characters?
*Answer*:
```dart
inputFormatters: [
  LengthLimitingTextInputFormatter(4),
]
```

### Q10. Write a custom formatter class that removes all spaces from input text.
*Answer*:
```dart
class NoSpaceFormatter extends TextInputFormatter {
  @override
  TextEditingValue formatEditUpdate(
    TextEditingValue oldValue,
    TextEditingValue newValue,
  ) {
    final stripped = newValue.text.replaceAll(' ', '');
    return TextEditingValue(
      text: stripped,
      selection: TextSelection.collapsed(offset: stripped.length),
    );
  }
}
```

---

## 📝 Summary
- [TextInputFormatter](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/11_InputFormatters.md) provides real-time character filtering, formatting, and masking on every user input event.
- Built-in formatters include `FilteringTextInputFormatter.digitsOnly`, `allow()`, `deny()`, and `LengthLimitingTextInputFormatter`.
- Custom formatters implement `formatEditUpdate(oldValue, newValue)` and return an updated `TextEditingValue`.
- Always maintain proper `TextSelection` offsets in custom formatters to prevent cursor jumping glitches.
- Position sanitizing formatters before formatting masks in the `inputFormatters` list.
