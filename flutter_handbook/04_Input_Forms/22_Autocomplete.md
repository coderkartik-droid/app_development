# Autocomplete in Flutter

## 📖 Introduction
Typing full names, email addresses, airport codes, or postal locations into text boxes is tedious and error-prone. When searching for a flight to San Francisco, typing `"San"` should immediately present `"San Francisco (SFO)"` in a floating overlay beneath the cursor so the user can complete the entry with a single tap.

In Flutter, this typeahead suggestion pattern is powered by the [Autocomplete](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/22_Autocomplete.md) widget. [Autocomplete](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/22_Autocomplete.md) connects a text input field with an automatic floating overlay list of options. It is strongly typed using generics `Autocomplete<T>`, meaning it can filter and display complex domain objects (like `UserContact` or `Airport`) rather than just raw strings.

---

## 🤔 Why Do We Need It?
Without `Autocomplete`:
- Developers had to manually position an `OverlayEntry` using `CompositedTransformTarget` and `CompositedTransformFollower`.
- Tracking keyboard focus, scrolling listeners, outside-tap dismissals, and highlight selections required hundreds of lines of boilerplate.
- Handling backspaces and text changes while keeping the floating dropdown aligned with the text box was fragile.

With [Autocomplete](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/22_Autocomplete.md):
- **Built-in Floating Overlay**: Automatically mounts a dropdown menu directly beneath the text field.
- **Object Support**: Filter custom classes via `optionsBuilder` and control display formatting via `displayStringForOption`.
- **Full Customizability**: Override `fieldViewBuilder` to style the input text box and `optionsViewBuilder` to design custom suggestion tiles.

---

## 🌍 Real-Life Analogy
Think of [Autocomplete](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/22_Autocomplete.md) as a **Librarian Who Finishes Your Sentences**:

```
+-------------------------------------------------------------+
|                     LIBRARY SEARCH DESK                     |
|                                                             |
|   You walk up to the librarian and say:                     |
|   "Do you have books by Shake..."                           |
|                                                             |
|   Before you even finish speaking, the librarian holds      |
|   up 3 index cards:                                         |
|   +-----------------------------------------------------+   |
|   | 1. William Shakespeare (Playwright)                 |   |
|   | 2. Shakira (Biography)                              |   |
|   | 3. Shaker Architecture (History)                    |   |
|   +-----------------------------------------------------+   |
|                                                             |
|   You point to Card #1. The librarian stamps it!            |
|   You saved 10 seconds of speaking!                         |
+-------------------------------------------------------------+
```

---

## 🧠 Internal Working
Flutter routes `Autocomplete` through an internal coordination pipeline:

```
                     User Types Into Field ("mar")
                                  │
                                  ▼
               optionsBuilder(TextEditingValue textValue)
                                  │
                                  ▼
              Filters Options Database / Model List
                                  │
                                  ▼
                    Returns Iterable<T> matches
                                  │
                                  ▼
        Are matches empty OR field text empty?
        ├── YES ──► Closes Overlay
        └── NO  ──► Opens/Updates Overlay via CompositedTransformFollower
                                  │
                                  ▼
                    optionsViewBuilder Paints:
            ┌────────────────────────────────────────┐
            │ Martin Fowler (martin@refactoring.com) │
            │ Margaret Hamilton (apollo@nasa.gov)    │
            └────────────────────────────────────────┘
                                  │
                                  ▼
                         User Taps An Option
                                  │
                                  ▼
                    1. onSelected(T option) Fires
                    2. Field text updates with displayStringForOption(T)
                    3. Overlay closes smoothly
```

1. **RawAutocomplete**: `Autocomplete` is a high-level wrapper around `RawAutocomplete`.
2. **Overlay Tracking**: Uses `CompositedTransformTarget` on the text field and `CompositedTransformFollower` on the options overlay so the dropdown tracks the field even when scrolling.
3. **Selection Contract**: When an item is tapped, `displayStringForOption` converts the object to a string for the text field, while `onSelected` receives the raw object `T`.

---

## ⚙ Syntax

### Basic String Autocomplete
```dart
Autocomplete<String>(
  optionsBuilder: (TextEditingValue textEditingValue) {
    if (textEditingValue.text.isEmpty) {
      return const Iterable<String>.empty();
    }
    return allCountries.where((country) =>
        country.toLowerCase().contains(textEditingValue.text.toLowerCase()));
  },
  onSelected: (String selection) {
    print('Selected: $selection');
  },
)
```

### Custom Object Autocomplete
```dart
Autocomplete<Airport>(
  displayStringForOption: (Airport a) => '${a.city} (${a.code})',
  optionsBuilder: (TextEditingValue textValue) {
    if (textValue.text.isEmpty) return const Iterable<Airport>.empty();
    return airportList.where((a) =>
        a.city.toLowerCase().contains(textValue.text.toLowerCase()) ||
        a.code.toLowerCase().contains(textValue.text.toLowerCase()));
  },
  onSelected: (Airport airport) {
    print('Booking flight to: ${airport.code}');
  },
)
```

---

## 📋 Important Properties

| Property | Type | Description | Default |
| :--- | :--- | :--- | :--- |
| `optionsBuilder` | `AutocompleteOptionsBuilder<T>` | Callback returning the options matching the current input text. | *Required* |
| `onSelected` | `AutocompleteOnSelected<T>?` | Called when the user taps an option from the suggestions overlay. | `null` |
| `displayStringForOption` | `AutocompleteOptionToString<T>` | Transforms object `T` into the display string set on the text field. | `RawAutocomplete.defaultStringForOption` |
| `fieldViewBuilder` | `AutocompleteFieldViewBuilder?` | Customizes the input `TextField` (controller, focusNode, decoration). | Material default |
| `optionsViewBuilder` | `AutocompleteOptionsViewBuilder<T>?`| Customizes the appearance, width, elevation, and shape of the dropdown menu. | Material default |
| `initialValue` | `TextEditingValue?` | Initial text value when the field first mounts. | `null` |

---

## 💻 Complete Runnable Example
The following production-ready application demonstrates:
1. Autocomplete for strongly-typed `RecipientContact` objects.
2. Custom `fieldViewBuilder` with clear button and theme styling.
3. Custom `optionsViewBuilder` with user avatars, subtext, and scroll constraints.
4. Handling selection events cleanly.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const AutocompleteMasterclassApp());
}

class AutocompleteMasterclassApp extends StatelessWidget {
  const AutocompleteMasterclassApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Autocomplete Masterclass',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const EmailComposerScreen(),
    );
  }
}

// 1. Strongly typed data model
class RecipientContact {
  final String name;
  final String email;
  final String role;

  const RecipientContact({
    required this.name,
    required this.email,
    required this.role,
  });

  @override
  String toString() => '$name ($email)';
}

class EmailComposerScreen extends StatefulWidget {
  const EmailComposerScreen({super.key});

  @override
  State<EmailComposerScreen> createState() => _EmailComposerScreenState();
}

class _EmailComposerScreenState extends State<EmailComposerScreen> {
  // Database of contacts
  static const List<RecipientContact> _allContacts = [
    RecipientContact(name: 'Sarah Connor', email: 'sarah@skynet.defense', role: 'Security Lead'),
    RecipientContact(name: 'John Doe', email: 'john.doe@enterprise.com', role: 'Project Manager'),
    RecipientContact(name: 'Sam Altman', email: 'sam@openai.org', role: 'Executive'),
    RecipientContact(name: 'Sundar Pichai', email: 'sundar@google.com', role: 'Executive'),
    RecipientContact(name: 'Ada Lovelace', email: 'ada@computing.history', role: 'Chief Architect'),
    RecipientContact(name: 'Alan Turing', email: 'alan@enigma.code', role: 'Research Fellow'),
  ];

  RecipientContact? _selectedRecipient;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Compose Secure Message'),
        centerTitle: true,
      ),
      body: SafeArea(
        child: Padding(
          padding: const EdgeInsets.all(24.0),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: [
              const Text(
                'Recipient Lookup',
                style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
              ),
              const SizedBox(height: 6),
              const Text(
                'Type a name or email address to trigger smart suggestions.',
                style: TextStyle(color: Colors.grey),
              ),
              const SizedBox(height: 20),

              // Strongly typed Autocomplete widget
              Autocomplete<RecipientContact>(
                displayStringForOption: (RecipientContact contact) => contact.name,
                optionsBuilder: (TextEditingValue textEditingValue) {
                  final query = textEditingValue.text.trim().toLowerCase();
                  if (query.isEmpty) {
                    return const Iterable<RecipientContact>.empty();
                  }

                  return _allContacts.where((contact) {
                    return contact.name.toLowerCase().contains(query) ||
                        contact.email.toLowerCase().contains(query);
                  });
                },
                onSelected: (RecipientContact selection) {
                  setState(() {
                    _selectedRecipient = selection;
                  });
                },

                // Custom Input TextField Builder
                fieldViewBuilder: (
                  BuildContext context,
                  TextEditingController fieldController,
                  FocusNode fieldFocusNode,
                  VoidCallback onFieldSubmitted,
                ) {
                  return TextField(
                    controller: fieldController,
                    focusNode: fieldFocusNode,
                    decoration: InputDecoration(
                      labelText: 'To: (Recipient Name or Email)',
                      hintText: 'e.g. Sarah or ada@...',
                      prefixIcon: const Icon(Icons.alternate_email),
                      suffixIcon: fieldController.text.isNotEmpty
                          ? IconButton(
                              icon: const Icon(Icons.clear),
                              onPressed: () {
                                fieldController.clear();
                                setState(() {
                                  _selectedRecipient = null;
                                });
                              },
                            )
                          : null,
                      border: const OutlineInputBorder(),
                    ),
                  );
                },

                // Custom Suggestions Floating Overlay Builder
                optionsViewBuilder: (
                  BuildContext context,
                  AutocompleteOnSelected<RecipientContact> onSelected,
                  Iterable<RecipientContact> options,
                ) {
                  return Align(
                    alignment: Alignment.topLeft,
                    child: Material(
                      elevation: 6,
                      borderRadius: BorderRadius.circular(12),
                      color: Colors.white,
                      child: ConstrainedBox(
                        constraints: const BoxConstraints(maxHeight: 250, maxWidth: 340),
                        child: ListView.separated(
                          padding: const EdgeInsets.symmetric(vertical: 8),
                          shrinkWrap: true,
                          itemCount: options.length,
                          separatorBuilder: (_, __) => const Divider(height: 1),
                          itemBuilder: (BuildContext context, int index) {
                            final contact = options.elementAt(index);
                            return InkWell(
                              onTap: () => onSelected(contact),
                              child: Padding(
                                padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 10),
                                child: Row(
                                  children: [
                                    CircleAvatar(
                                      backgroundColor: Colors.deepPurple.shade100,
                                      child: Text(
                                        contact.name[0],
                                        style: const TextStyle(fontWeight: FontWeight.bold),
                                      ),
                                    ),
                                    const SizedBox(width: 12),
                                    Expanded(
                                      child: Column(
                                        crossAxisAlignment: CrossAxisAlignment.start,
                                        children: [
                                          Text(
                                            contact.name,
                                            style: const TextStyle(fontWeight: FontWeight.bold),
                                          ),
                                          Text(
                                            contact.email,
                                            style: TextStyle(color: Colors.grey.shade600, fontSize: 12),
                                          ),
                                        ],
                                      ),
                                    ),
                                    Chip(
                                      label: Text(contact.role, style: const TextStyle(fontSize: 10)),
                                      visualDensity: VisualDensity.compact,
                                      padding: EdgeInsets.zero,
                                    ),
                                  ],
                                ),
                              ),
                            );
                          },
                        ),
                      ),
                    ),
                  );
                },
              ),
              const SizedBox(height: 32),

              // Selected Contact Card Display
              if (_selectedRecipient != null)
                Card(
                  elevation: 0,
                  color: Colors.deepPurple.shade50,
                  shape: RoundedRectangleBorder(
                    borderRadius: BorderRadius.circular(12),
                    side: BorderSide(color: Colors.deepPurple.shade200),
                  ),
                  child: Padding(
                    padding: const EdgeInsets.all(16.0),
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        const Row(
                          children: [
                            Icon(Icons.verified_user, color: Colors.deepPurple, size: 20),
                            SizedBox(width: 8),
                            Text(
                              'Recipient Verified',
                              style: TextStyle(fontWeight: FontWeight.bold, color: Colors.deepPurple),
                            ),
                          ],
                        ),
                        const SizedBox(height: 10),
                        Text('Name: ${_selectedRecipient!.name}'),
                        Text('Email: ${_selectedRecipient!.email}'),
                        Text('Role: ${_selectedRecipient!.role}'),
                      ],
                    ),
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

## 🔍 Line-by-Line Example Explanation
- `Autocomplete<RecipientContact>(...)`:
  - **What it does**: Binds the typeahead system to a custom Dart class rather than plain strings.
  - **Why it exists**: Enables accessing properties like `contact.email` or `contact.role` directly in callbacks.
- `displayStringForOption: (RecipientContact c) => c.name`:
  - **What it does**: Tells Flutter what string to write into the text field once the user selects an option.
  - **Fatal mistake if omitted**: Flutter defaults to calling `.toString()`, which could populate the text box with ugly text like `Instance of 'RecipientContact'`.
- `optionsBuilder: (TextEditingValue val) { ... }`:
  - **What it does**: Takes the current text input and returns matching elements.
  - **Why check `query.isEmpty`**: If the user hasn't typed anything, returning an empty iterable prevents the overlay from popping open immediately on screen launch.
- `fieldViewBuilder: (context, controller, focusNode, onFieldSubmitted) { ... }`:
  - **What it does**: Gives you full control over the `TextField` while wiring up the framework's internal controller and focus node.
  - **Why it exists**: Allows adding icons, labels, borders, and clear buttons while preserving autocomplete behavior.
- `optionsViewBuilder: (context, onSelected, options) { ... }`:
  - **What it does**: Renders the floating popover menu.
  - **Crucial layout constraint**: Always wrap the options in `Align(alignment: Alignment.topLeft)` and `ConstrainedBox(maxHeight: 250)` so the overlay doesn't expand infinitely across the entire screen.

---

## 📱 ASCII Phone Screen Preview
```
+------------------------------------------+
|          Compose Secure Message          |
+------------------------------------------+
| Recipient Lookup                         |
| Type a name or email address...          |
|                                          |
| +--------------------------------------+ |
| | [@] To: Sa|                      [✕] | |
| +--------------------------------------+ |
|                                          |
|   ┌──────────────────────────────────┐   |
|   │ (S) Sarah Connor                 │   |
|   │     sarah@skynet.defense [Security]  |
|   │----------------------------------│   |
|   │ (S) Sam Altman                   │   |
|   │     sam@openai.org       [Exec]  │   |
|   └──────────────────────────────────┘   |
|                                          |
| +--------------------------------------+ |
| | [✓] Recipient Verified               | |
| | Name: Sarah Connor                   | |
| | Email: sarah@skynet.defense          | |
| | Role: Security Lead                  | |
| +--------------------------------------+ |
+------------------------------------------+
```

---

## 🌳 Widget Tree
```
MaterialApp
 └── EmailComposerScreen (StatefulWidget)
      └── Scaffold
           ├── AppBar
           └── SafeArea
                └── Padding
                     └── Column
                          ├── Text ("Recipient Lookup")
                          ├── Autocomplete<RecipientContact>
                          │    ├── fieldViewBuilder: TextField
                          │    └── optionsViewBuilder: Material -> ConstrainedBox -> ListView
                          └── Card (Selected Recipient details)
```

---

## ⚙ Flutter Execution Flow
1. **User Types**: User enters `"sa"` into the text field.
2. **Evaluation**: `optionsBuilder` inspects `"sa"`.
3. **Filtering**: Matches `Sarah Connor` and `Sam Altman`.
4. **Overlay Render**: `optionsViewBuilder` executes; an `OverlayEntry` renders floating beneath the text field using `CompositedTransformFollower`.
5. **Option Tapped**: User touches the Sarah Connor row.
6. **Selection Hook**:
   - `onSelected(contact)` is invoked.
   - Text field controller text updates to `"Sarah Connor"` via `displayStringForOption`.
   - The floating overlay dismisses automatically.
7. **UI Update**: `_selectedRecipient` updates and displays the green verification card.

---

## ⚠ Common Beginner Mistakes

### 1. Forgetting `displayStringForOption` on Custom Classes
```dart
// ❌ UGLY TEXT: Selecting option displays "Instance of 'User'" in text box!
Autocomplete<User>(
  optionsBuilder: (text) => users.where(...),
  // Missing displayStringForOption!
)

// ✅ GOOD: Explicitly extract the display string
Autocomplete<User>(
  displayStringForOption: (user) => user.fullName,
  optionsBuilder: (text) => users.where(...),
)
```

### 2. Missing `Align` and `ConstrainedBox` in `optionsViewBuilder`
```dart
// ❌ LAYOUT CRASH: Dropdown stretches across the entire screen!
optionsViewBuilder: (context, onSelected, options) {
  return Material(
    child: ListView(...), // Stretches to fill entire phone height and width!
  );
}

// ✅ GOOD: Constrain height and align to top-left
optionsViewBuilder: (context, onSelected, options) {
  return Align(
    alignment: Alignment.topLeft,
    child: Material(
      elevation: 4,
      child: ConstrainedBox(
        constraints: const BoxConstraints(maxHeight: 200, maxWidth: 300),
        child: ListView(...),
      ),
    ),
  );
}
```

### 3. Throwing Exceptions on Empty Input
```dart
// ❌ BAD: Returns null or throws when empty
optionsBuilder: (val) {
  if (val.text.isEmpty) return null; // Compile error! Must return Iterable!
}

// ✅ GOOD: Return empty iterable
optionsBuilder: (val) {
  if (val.text.isEmpty) return const Iterable.empty();
  ...
}
```

---

## 🚀 Professional Usage
- **Flight & Hotel Search (e.g., Kayak, Skyscanner)**: Typing city names returns airport codes (`SFO`, `LHR`, `JFK`) with airline terminal info.
- **Social Media & Messaging (e.g., Twitter/X, Slack)**: Typing `@` or `#` triggers mention and hashtag suggestion dropdowns.
- **Address & Postal Code Lookup (e.g., Google Maps)**: Street address typeahead autocompleting full street names and ZIP codes.

---

## 💡 Best Practices
1. **Always provide `displayStringForOption` when using custom objects**: Guarantees clean text formatting upon selection.
2. **Constrain overlay dimensions**: Always cap the height and width in `optionsViewBuilder` using `ConstrainedBox(maxHeight: 250)` to prevent the menu from blocking the entire screen.
3. **Check `text.isEmpty` first**: Always return `Iterable.empty()` when the user hasn't typed anything yet to keep the UI clean.
4. **Use `InkWell` for suggestion items**: Provides responsive Material ink ripples when options are touched.

---

## 🧪 Try It Yourself
1. Add an airport code field where typing either city name or airport IATA code autocompletes both.
2. Add a `CircularProgressIndicator` inside `optionsViewBuilder` while simulating an asynchronous backend search.
3. Highlight matching characters in bold inside the suggestion list tiles.

---

## 🎯 Challenge
Create a dynamic **Stock Ticker Autocomplete**:
- Model: `Stock(symbol: 'AAPL', name: 'Apple Inc.', price: 185.50)`.
- As the user types either ticker symbol or company name, show suggestions formatted with ticker badge, name, and live price.
- Selecting a stock populates the field with the symbol and displays an interactive buy/sell card below.

---

## ❓ Practice Questions

### Q1. What is the primary purpose of `Autocomplete` in Flutter?
- A) To automatically correct spelling mistakes
- B) To provide a floating typeahead dropdown list of suggestions as the user types
- C) To autofill passwords from Keychain
- D) To generate random strings
*Answer: B* — It provides live suggestions in a floating overlay based on current text input.

### Q2. What does `displayStringForOption` do when using `Autocomplete<User>`?
- A) It prints the user to the debug console
- B) It converts the selected `User` object into a display string to place into the `TextField`
- C) It translates the user's name
- D) It encrypts the user's data
*Answer: B* — It defines which string property of the object is inserted into the text box.

### Q3. What should `optionsBuilder` return when the input text is empty and no suggestions should be shown?
- A) `null`
- B) `const Iterable.empty()`
- C) An empty String `""`
- D) A blank `Container()`
*Answer: B* — An empty iterable signals that no suggestions should be displayed.

### Q4. Which builder property allows you to fully customize the appearance of the input `TextField`?
- A) `inputBuilder`
- B) `fieldViewBuilder`
- C) `textFieldBuilder`
- D) `textCustomizer`
*Answer: B* — `fieldViewBuilder` provides access to the controller, focusNode, and input decoration.

### Q5. Why should you wrap `optionsViewBuilder` inside `Align(alignment: Alignment.topLeft)` and `ConstrainedBox`?
- A) To prevent the overlay from expanding across the entire screen and unconstrained vertical viewport crashes
- B) To center the text
- C) To make it load faster
- D) It is required by the Dart linter
*Answer: A* — Constrains the floating overlay dimensions so it remains an attached dropdown menu.

### Q6. Can `optionsBuilder` be an asynchronous function that fetches suggestions from an API?
*Answer*: Yes, `optionsBuilder` can return `Future<Iterable<T>>`, allowing real-time network or database lookups.

### Q7. What callback is executed when the user taps an option in the autocomplete dropdown?
*Answer*: `onSelected: (T selection)`.

### Q8. Output Prediction: What will appear in the text field if `displayStringForOption` is omitted on a custom class without an overridden `toString()`?
*Answer*: `"Instance of 'ClassName'"`.

### Q9. What underlying widget positions the floating options overlay directly beneath the text box?
*Answer*: `CompositedTransformFollower` (anchored to `CompositedTransformTarget`).

### Q10. Write a minimal `optionsBuilder` function that filters a list of strings called `tags`.
*Answer*:
```dart
optionsBuilder: (TextEditingValue val) {
  if (val.text.isEmpty) return const Iterable<String>.empty();
  return tags.where((t) => t.toLowerCase().contains(val.text.toLowerCase()));
}
```

---

## 📝 Summary
- [Autocomplete](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/22_Autocomplete.md) provides automated typeahead suggestion overlays attached directly to text inputs.
- It is strongly typed using generics `Autocomplete<T>` for custom domain models.
- Always implement `displayStringForOption` when using custom objects to avoid `'Instance of...'` bugs.
- Customize inputs via `fieldViewBuilder` and style floating dropdown menus via `optionsViewBuilder`.
- Always constrain overlay height with `ConstrainedBox` inside `optionsViewBuilder`.
