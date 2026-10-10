# Icon

## 📖 Introduction

`Icon` displays a symbolic graphic from an icon font or another `IconData` source. Icons communicate actions, status, navigation, and categories in a compact form.

## 🤔 Why Do We Need It?

An icon can communicate familiar meaning quickly and save space. In a good interface, it supports nearby text or has an accessible label when it stands alone.

## 🧠 Internal Working

Flutter's built-in Material icons are glyphs in an icon font. `Icon` selects a glyph from `IconData`, applies size and color, and paints it within the constraints supplied by its parent. It does not automatically become interactive; place it inside `IconButton` when it should respond to taps.

## ⚙ Constructor (if applicable)

```dart
const Icon(
  Icons.favorite,
  size: 40,
  color: Colors.red,
)
```

## 📋 Properties (only relevant ones)

| Property | Purpose |
|---|---|
| `icon` | The `IconData` glyph to display |
| `size` | Width and height preference |
| `color` | Icon color |
| `semanticLabel` | Spoken meaning for accessibility |
| `textDirection` | Direction for directional icons |
| `semanticLabel` | Description when the icon itself conveys meaning |

## 💻 Complete Runnable Example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const IconApp());

class IconApp extends StatelessWidget {
  const IconApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('Icon')),
        body: const Center(
          child: Row(
            mainAxisSize: MainAxisSize.min,
            children: <Widget>[
              Icon(
                Icons.favorite,
                color: Colors.red,
                size: 48,
                semanticLabel: 'Favorite',
              ),
              SizedBox(width: 12),
              Text('Favorite'),
            ],
          ),
        ),
      ),
    );
  }
}
```

## 🔍 Line-by-Line Example Explanation

- `IconApp` is the root widget.
- `Scaffold` supplies the page structure.
- `Center` positions the row in the available body.
- `Row` places the icon and label horizontally.
- `mainAxisSize: MainAxisSize.min` keeps the row close to its content width.
- `Icons.favorite` selects the Material favorite glyph.
- `color` and `size` customize its appearance.
- `semanticLabel` gives the icon an accessible description.
- `SizedBox` creates space between the icon and text.

## 📱 ASCII Phone Screen Preview

```text
+----------------------+
| Icon                 |
|----------------------|
|       ♥  Favorite    |
|                      |
+----------------------+
```

## 🌳 Widget Tree Diagram

```text
Scaffold
├── AppBar → Text
└── Center
    └── Row
        ├── Icon
        ├── SizedBox
        └── Text
```

## ⚙ Flutter Execution Flow

```text
IconData selected → Icon applies size/color → glyph is laid out
                  → semantic information is exposed → glyph is painted
```

## ⚠ Common Beginner Mistakes

- Assuming an `Icon` is automatically tappable.
- Using an icon without a tooltip or label when it stands alone.
- Choosing a symbol whose meaning is unclear.
- Making icons too small for comfortable interaction.
- Relying on color alone to communicate status.

## 🚀 Professional Usage

Use icons consistently with the platform's visual language. Put interactive icons in `IconButton`, provide tooltips, and test their meaning with screen readers and users unfamiliar with the product.

## 💡 Best Practices

- Use `IconButton` for actions.
- Add `semanticLabel` when an icon communicates information by itself.
- Keep icon style and size consistent.
- Pair unfamiliar icons with visible text.
- Check contrast against the background.

## 🧪 Try It Yourself

Replace `Icons.favorite` with `Icons.star`, then change its color and size.

## 🎯 Challenge

Create a row containing three status icons—success, warning, and error—with a text label for each.

## ❓ 10 Practice Questions

1. What does `Icon` display?
2. What is `IconData`?
3. Is an Icon automatically interactive?
4. Which widget is designed for tappable icons?
5. What does `size` control?
6. Why use `semanticLabel`?
7. Why should color not be the only status signal?
8. What does `mainAxisSize` do in the example?
9. How can an unfamiliar icon be clarified?
10. Where do Material glyphs come from?

## 📝 Summary

`Icon` displays a symbolic glyph and supports size, color, and accessibility labeling. Use it for communication, and use `IconButton` when the icon represents an action.
