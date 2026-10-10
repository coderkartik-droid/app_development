# ListTile

## 📖 Introduction

`ListTile` is a Material widget for a standard row of list content. It can arrange a leading widget, a title, an optional subtitle, and a trailing widget with familiar alignment and spacing.

## 🤔 Why Do We Need It?

Many app screens show repeated rows such as settings, contacts, messages, and menu options. Building each row from scratch can lead to inconsistent alignment and touch behavior. `ListTile` provides a consistent starting point for this common pattern.

## 🧠 Internal Working

`ListTile` lays out its optional leading and trailing widgets around its text content. It uses Material styling and can respond to taps when `onTap` is supplied. Its vertical layout is affected by the title, subtitle, density, and available width.

When placing many tiles, use a scrolling parent such as `ListView`. A `Column` containing many rows can exceed the available screen height.

## ⚙ Constructor (if applicable)

```dart
ListTile(
  leading: const Icon(Icons.person),
  title: const Text('Profile'),
  subtitle: const Text('Manage your details'),
  trailing: const Icon(Icons.chevron_right),
  onTap: () {},
)
```

## 📋 Properties (only relevant ones)

| Property | Purpose |
|---|---|
| `leading` | Widget at the beginning of the row |
| `title` | Main row content |
| `subtitle` | Secondary content below the title |
| `trailing` | Widget at the end of the row |
| `onTap` | Optional tap callback; makes the tile interactive |
| `dense` | Requests a more compact tile layout |
| `isThreeLine` | Allows a layout intended for a title and two subtitle lines |

## 💻 Complete Runnable Example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const ListTileApp());

class ListTileApp extends StatelessWidget {
  const ListTileApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('ListTile')),
        body: ListView(
          children: [
            ListTile(
              leading: const Icon(Icons.person),
              title: const Text('Profile'),
              subtitle: const Text('Manage your details'),
              trailing: const Icon(Icons.chevron_right),
              onTap: () {},
            ),
            const Divider(height: 1),
            const ListTile(
              leading: Icon(Icons.notifications),
              title: Text('Notifications'),
              subtitle: Text('Choose what you receive'),
            ),
          ],
        ),
      ),
    );
  }
}
```

## 🔍 Line-by-Line Example Explanation

- `main` launches `ListTileApp`.
- `MaterialApp` and `Scaffold` provide the Material screen.
- `ListView` makes its children scrollable if they exceed the available height.
- The first `ListTile` shows how a row can have all four content areas.
- `leading` displays a person icon before the text.
- `title` is the primary label, and `subtitle` gives secondary information.
- `trailing` displays a chevron after the content.
- `onTap` makes the first row respond to taps; the empty callback keeps this sample self-contained.
- `Divider` draws a separator after the first row.
- The second tile is `const` and has no `onTap`, so it is presented as non-interactive content.

## 📱 ASCII Phone Screen Preview

```text
+----------------------+
| ListTile             |
|----------------------|
|  (person) Profile  > |
|           Manage...  |
|----------------------|
|  (bell) Notifications|
|         Choose...    |
+----------------------+
```

## 🌳 Widget Tree Diagram

```text
ListTileApp
└── MaterialApp
    └── Scaffold
        ├── AppBar
        └── ListView
            ├── ListTile (interactive)
            │   ├── leading: Icon
            │   ├── title: Text
            │   ├── subtitle: Text
            │   └── trailing: Icon
            ├── Divider
            └── ListTile
                ├── leading: Icon
                ├── title: Text
                └── subtitle: Text
```

## ⚙ Flutter Execution Flow

```text
ListView provides row constraints
        ↓
ListTile arranges leading, text, and trailing areas
        ↓
Material styling and optional tap behavior are applied
        ↓
Rows are laid out and painted; the list scrolls as needed
```

## ⚠ Common Beginner Mistakes

- Putting many tiles in a non-scrollable `Column`.
- Setting `onTap` to null while expecting the tile to be tappable.
- Treating visual arrows as proof that an action is implemented.
- Giving a long title, subtitle, and large leading/trailing widgets too little width.
- Using `isThreeLine` when the content does not need the extra line height.

## 🚀 Professional Usage

Use `ListTile` for familiar, consistent list rows. For long or data-driven lists, consider lazily built list patterns such as `ListView.builder`. Provide real navigation or action behavior for interactive tiles and ensure the row's text remains understandable at larger text scales.

## 💡 Best Practices

- Keep title and subtitle hierarchy clear.
- Use `ListView` for content that may need to scroll.
- Make interactive intent obvious and implement the callback.
- Avoid cramming too many controls into one tile.
- Use a purpose-built custom row only when `ListTile` cannot express the design clearly.

## 🧪 Try It Yourself

Add a third tile with a `leading` icon and a trailing switch-like control. Consider whether the entire row or just the control should handle interaction.

## 🎯 Challenge

Build a scrollable settings list with at least four tiles. Give each tile a distinct leading icon, concise title, and an appropriate action, and separate groups with a divider.

## ❓ 10 Practice Questions

1. What common layout does `ListTile` provide?
2. What is the role of `leading`?
3. Where does `subtitle` appear?
4. What does `onTap` control?
5. Why does the example use a `ListView`?
6. Does a trailing chevron automatically navigate?
7. What does `dense` request?
8. When is `isThreeLine` appropriate?
9. What can happen when a tile has too many wide widgets?
10. When might a custom row be preferable?

## 📝 Summary

`ListTile` standardizes a common Material list row with leading, text, trailing, and optional tap behavior. Place repeated tiles in a scrolling layout when needed, and connect interactive affordances to real actions.
