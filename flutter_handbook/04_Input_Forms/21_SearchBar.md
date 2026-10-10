# SearchBar & SearchAnchor in Flutter

## 📖 Introduction
Search is the primary discovery engine for modern mobile applications. Whether browsing music on Spotify, shopping on Amazon, or searching contacts on WhatsApp, users expect an immediate, responsive search experience: a rounded search field with clear icons, auto-suggestions appearing on keystrokes, and full-screen overlay views that take over the screen.

In Flutter 3.10 and later, the Material 3 design system introduced two dedicated widgets that replace legacy custom search bars:
- [SearchBar](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/21_SearchBar.md): A pre-styled Material 3 search input container with integrated leading icons, trailing clear buttons, elevation shadows, and custom surface tints.
- [SearchAnchor](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/21_SearchBar.md): An all-in-one search coordinator that pairs a [SearchBar](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/21_SearchBar.md) with an animated, full-screen or popup search view managed via a [SearchController](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/21_SearchBar.md) and a `suggestionsBuilder`.

---

## 🤔 Why Do We Need It?
Before `SearchBar` and `SearchAnchor`:
- Developers had to manually style `TextField` with rounded `OutlineInputBorder`, embed custom prefix/suffix icons, and manage focus nodes.
- Displaying real-time search suggestions required building custom overlay portals or invoking `showSearch()` with a cumbersome `SearchDelegate` class.
- Animating the transition from an in-page search bar to an expanded full-screen search mode required complex hero animations and custom page routes.

With [SearchBar](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/21_SearchBar.md) & [SearchAnchor](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/21_SearchBar.md):
- **Turnkey Material 3 Aesthetics**: Pill shape, elevation, hint text, and leading magnifying glass are built-in.
- **Dynamic Suggestions**: `suggestionsBuilder` returns an iterable list of suggestion widgets (such as `ListTile`s) that filter in real-time as the user types.
- **Unified Controller**: `SearchController` allows opening, closing, clearing, or reading query text programmatically with clean methods like `controller.openView()` and `controller.closeView()`.

---

## 🌍 Real-Life Analogy
Think of [SearchAnchor](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/21_SearchBar.md) as a **Department Store Directory Kiosk**:

```
+-------------------------------------------------------------+
|                     DEPARTMENT STORE KIOSK                  |
|                                                             |
|   [ When Idle (SearchBar) ]:                                |
|   +-----------------------------------------------------+   |
|   | [🔍] Search products, brands, or departments...     |   |
|   +-----------------------------------------------------+   |
|                                                             |
|   [ When You Tap the Screen (SearchAnchor Expands) ]:       |
|   The screen expands to show:                               |
|   • Recent Searches: "Running Shoes", "Wireless Earbuds"    |
|   • Live Category Suggestions matching your keystrokes      |
|   • Instant results matching floor numbers                  |
|                                                             |
|   Once you pick a destination, the kiosk closes back down!  |
+-------------------------------------------------------------+
```

---

## 🧠 Internal Working
How `SearchAnchor` coordinates the search experience:

```
                    SearchAnchor Widget
                             │
             ┌───────────────┴───────────────┐
             ▼                               ▼
       builder Callback              suggestionsBuilder Callback
    (Renders the SearchBar)         (Renders the live dropdown/view)
             │                               │
             ▼                               ▼
     User Taps SearchBar               User Types "Fl"
             │                               │
             ▼                               ▼
   controller.openView()            Filters database / list
             │                               │
             ▼                               ▼
   Animates full-screen view        Returns Iterable<Widget>
   with back button & input        ([Flutter, Flight, Flora])
```

1. **Dual Representation**: The `builder` constructs the anchor widget (typically a `SearchBar`).
2. **View Transition**: When the user taps the search bar, `controller.openView()` pushes a full-screen Material search view or drops down a floating suggestions menu on larger screens.
3. **Reactive Query Pipeline**: `suggestionsBuilder(context, controller)` executes whenever text changes, returning a list of widgets that populate the search view.
4. **Closing the View**: Calling `controller.closeView(selectedString)` dismisses the search view and updates the search bar text.

---

## ⚙ Syntax

### Standalone SearchBar
```dart
SearchBar(
  controller: _searchController,
  hintText: 'Search contacts...',
  leading: const Icon(Icons.search),
  trailing: [
    IconButton(
      icon: const Icon(Icons.clear),
      onPressed: () => _searchController.clear(),
    ),
  ],
  onChanged: (String query) {
    // Filter results locally
  },
)
```

### SearchAnchor (Full Search Flow with Suggestions)
```dart
SearchAnchor(
  searchController: _searchController,
  builder: (BuildContext context, SearchController controller) {
    return SearchBar(
      controller: controller,
      hintText: 'Search items...',
      leading: const Icon(Icons.search),
      onTap: () => controller.openView(),
      onChanged: (_) => controller.openView(),
    );
  },
  suggestionsBuilder: (BuildContext context, SearchController controller) {
    final query = controller.text.toLowerCase();
    final matches = allItems.where((item) => item.toLowerCase().contains(query));

    return matches.map((item) {
      return ListTile(
        title: Text(item),
        onTap: () {
          controller.closeView(item);
        },
      );
    });
  },
)
```

---

## 📋 Important Properties

| Property | Type | Description | Default |
| :--- | :--- | :--- | :--- |
| `controller` | `SearchController?` | Controller to programmatically open, close, and read search queries. | `null` |
| `hintText` | `String?` | Placeholder text displayed when the search bar is empty. | `null` |
| `leading` | `Widget?` | Widget displayed at the start of the bar (typically a magnifying glass icon). | `null` |
| `trailing` | `Iterable<Widget>?` | Widgets displayed at the end of the bar (e.g. clear button, mic icon). | `null` |
| `onChanged` | `ValueChanged<String>?` | Callback fired on every keystroke. | `null` |
| `onSubmitted` | `ValueChanged<String>?` | Callback fired when the user presses Enter/Search on the keyboard. | `null` |
| `suggestionsBuilder` *(Anchor)* | `SuggestionsBuilder` | Returns an `Iterable<Widget>` (or `Future<Iterable<Widget>>`) of suggestion rows. | *Required* |
| `isFullScreen` *(Anchor)* | `bool?` | Forces the search view into a full-screen route or anchored popup menu. | `null` (adaptive) |

---

## 💻 Complete Runnable Example
The following production-grade application demonstrates:
1. An embedded [SearchBar](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/21_SearchBar.md) inside an e-commerce catalog screen.
2. A [SearchAnchor](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/21_SearchBar.md) providing live suggestions with category tags.
3. Recent search history chips.
4. One-tap clear and search execution.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const SearchBarMasterclassApp());
}

class SearchBarMasterclassApp extends StatelessWidget {
  const SearchBarMasterclassApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'SearchBar & SearchAnchor Guide',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
        useMaterial3: true,
      ),
      home: const ProductCatalogSearchScreen(),
    );
  }
}

class ProductCatalogSearchScreen extends StatefulWidget {
  const ProductCatalogSearchScreen({super.key});

  @override
  State<ProductCatalogSearchScreen> createState() =>
      _ProductCatalogSearchScreenState();
}

class _ProductCatalogSearchScreenState
    extends State<ProductCatalogSearchScreen> {
  // Search controller to manage queries and view states
  final SearchController _searchController = SearchController();

  // Mock product catalog database
  final List<Map<String, String>> _products = [
    {'name': 'Apple MacBook Pro M3', 'category': 'Laptops', 'price': '\$1,999'},
    {'name': 'Apple iPhone 15 Pro', 'category': 'Phones', 'price': '\$999'},
    {'name': 'Sony WH-1000XM5 Headphones', 'category': 'Audio', 'price': '\$399'},
    {'name': 'Dell XPS 15 OLED', 'category': 'Laptops', 'price': '\$1,699'},
    {'name': 'Google Pixel 8 Pro', 'category': 'Phones', 'price': '\$899'},
    {'name': 'Bose QuietComfort 45', 'category': 'Audio', 'price': '\$329'},
    {'name': 'Samsung Galaxy S24 Ultra', 'category': 'Phones', 'price': '\$1,199'},
    {'name': 'Logitech MX Master 3S', 'category': 'Accessories', 'price': '\$99'},
  ];

  final List<String> _recentSearches = ['MacBook', 'Headphones', 'Pixel'];
  String _activeFilter = '';

  @override
  void dispose() {
    _searchController.dispose();
    super.dispose();
  }

  void _applySearch(String query) {
    setState(() {
      _activeFilter = query;
      if (query.isNotEmpty && !_recentSearches.contains(query)) {
        _recentSearches.insert(0, query);
        if (_recentSearches.length > 5) _recentSearches.removeLast();
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    final filteredProducts = _products.where((p) {
      final name = p['name']!.toLowerCase();
      final cat = p['category']!.toLowerCase();
      final q = _activeFilter.toLowerCase();
      return name.contains(q) || cat.contains(q);
    }).toList();

    return Scaffold(
      appBar: AppBar(
        title: const Text('Gadget Catalog Search'),
        centerTitle: true,
      ),
      body: SafeArea(
        child: Column(
          children: [
            // SearchAnchor Bar Header
            Padding(
              padding: const EdgeInsets.all(16.0),
              child: SearchAnchor(
                searchController: _searchController,
                builder: (BuildContext context, SearchController controller) {
                  return SearchBar(
                    controller: controller,
                    hintText: 'Search laptops, phones, audio...',
                    padding: const WidgetStatePropertyAll<EdgeInsets>(
                      EdgeInsets.symmetric(horizontal: 16.0),
                    ),
                    leading: const Icon(Icons.search, color: Colors.indigo),
                    trailing: [
                      if (controller.text.isNotEmpty)
                        IconButton(
                          icon: const Icon(Icons.clear),
                          onPressed: () {
                            controller.clear();
                            _applySearch('');
                          },
                        ),
                    ],
                    onTap: () => controller.openView(),
                    onChanged: (_) => controller.openView(),
                    onSubmitted: (query) => _applySearch(query),
                  );
                },
                suggestionsBuilder: (BuildContext context, SearchController controller) {
                  final query = controller.text.toLowerCase();
                  final matches = _products.where((p) {
                    final name = p['name']!.toLowerCase();
                    final cat = p['category']!.toLowerCase();
                    return name.contains(query) || cat.contains(query);
                  }).toList();

                  if (matches.isEmpty) {
                    return [
                      const ListTile(
                        leading: Icon(Icons.sentiment_dissatisfied),
                        title: Text('No matching products found'),
                        subtitle: Text('Try a different keyword or category'),
                      ),
                    ];
                  }

                  return matches.map((item) {
                    return ListTile(
                      leading: const Icon(Icons.inventory_2_outlined, color: Colors.indigo),
                      title: Text(item['name']!),
                      subtitle: Text('${item['category']} • ${item['price']}'),
                      trailing: const Icon(Icons.arrow_outward, size: 18),
                      onTap: () {
                        controller.closeView(item['name']);
                        _applySearch(item['name']!);
                      },
                    );
                  });
                },
              ),
            ),

            // Recent Searches Chips
            if (_recentSearches.isNotEmpty)
              Padding(
                padding: const EdgeInsets.symmetric(horizontal: 16.0),
                child: Row(
                  children: [
                    const Text('Recent: ', style: TextStyle(fontWeight: FontWeight.bold, fontSize: 12)),
                    const SizedBox(width: 8),
                    Expanded(
                      child: SingleChildScrollView(
                        scrollDirection: Axis.horizontal,
                        child: Row(
                          children: _recentSearches.map((term) {
                            return Padding(
                              padding: const EdgeInsets.only(right: 6.0),
                              child: ActionChip(
                                label: Text(term),
                                onPressed: () {
                                  _searchController.text = term;
                                  _applySearch(term);
                                },
                              ),
                            );
                          }).toList(),
                        ),
                      ),
                    ),
                  ],
                ),
              ),
            const Divider(height: 24),

            // Results List Header
            Padding(
              padding: const EdgeInsets.symmetric(horizontal: 16.0),
              child: Row(
                mainAxisAlignment: MainAxisAlignment.spaceBetween,
                children: [
                  Text(
                    _activeFilter.isEmpty
                        ? 'All Catalog Items (${filteredProducts.length})'
                        : 'Results for "$_activeFilter" (${filteredProducts.length})',
                    style: const TextStyle(fontWeight: FontWeight.bold, fontSize: 16),
                  ),
                  if (_activeFilter.isNotEmpty)
                    TextButton(
                      onPressed: () {
                        _searchController.clear();
                        _applySearch('');
                      },
                      child: const Text('Reset'),
                    ),
                ],
              ),
            ),
            const SizedBox(height: 8),

            // Filtered Products ListView
            Expanded(
              child: filteredProducts.isEmpty
                  ? Center(
                      child: Column(
                        mainAxisAlignment: MainAxisAlignment.center,
                        children: [
                          Icon(Icons.search_off, size: 64, color: Colors.grey.shade400),
                          const SizedBox(height: 12),
                          const Text(
                            'No products match your search criteria',
                            style: TextStyle(color: Colors.grey, fontSize: 16),
                          ),
                        ],
                      ),
                    )
                  : ListView.separated(
                      padding: const EdgeInsets.all(16.0),
                      itemCount: filteredProducts.length,
                      separatorBuilder: (_, __) => const SizedBox(height: 10),
                      itemBuilder: (context, index) {
                        final product = filteredProducts[index];
                        return Card(
                          elevation: 1,
                          shape: RoundedRectangleBorder(
                            borderRadius: BorderRadius.circular(12),
                            side: BorderSide(color: Colors.grey.shade200),
                          ),
                          child: ListTile(
                            leading: CircleAvatar(
                              backgroundColor: Colors.indigo.shade50,
                              child: const Icon(Icons.devices, color: Colors.indigo),
                            ),
                            title: Text(
                              product['name']!,
                              style: const TextStyle(fontWeight: FontWeight.bold),
                            ),
                            subtitle: Text('Category: ${product['category']}'),
                            trailing: Text(
                              product['price']!,
                              style: const TextStyle(
                                fontWeight: FontWeight.bold,
                                fontSize: 16,
                                color: Colors.indigo,
                              ),
                            ),
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
```

---

## 🔍 Line-by-Line Example Explanation
- `final SearchController _searchController = SearchController();`:
  - **What it does**: Controls search query text, view expansion (`openView()`), and view closure (`closeView()`).
  - **Why it exists**: Subclasses `TextEditingController` with specialized methods for search overlays.
- `SearchAnchor(builder: ..., suggestionsBuilder: ...)`:
  - **What it does**: Bridges the embedded search bar with the expandable overlay view.
  - **Internal behavior**: Clicking the bar triggers `openView()`, animating a full-screen Material view with an integrated search header.
- `suggestionsBuilder: (BuildContext context, SearchController controller) { ... }`:
  - **What it does**: Evaluates on every keystroke and returns an `Iterable<Widget>`.
  - **Why it exists**: Automatically populates the dropdown or full-screen view with live filtered suggestions.
- `controller.closeView(item['name'])`:
  - **What it does**: Closes the search view, updates the search bar text with the chosen item, and restores the underlying screen.

---

## 📱 ASCII Phone Screen Preview
```
+------------------------------------------+
|          Gadget Catalog Search           |
+------------------------------------------+
| +--------------------------------------+ |
| | [🔍] Search laptops, phones...   [✕] | |
| +--------------------------------------+ |
|                                          |
| Recent: [ MacBook ] [ Headphones ]       |
| ---------------------------------------- |
| Results for "Apple" (2)          [Reset] |
|                                          |
| +--------------------------------------+ |
| | (💻) Apple MacBook Pro M3            | |
| |      Category: Laptops       $1,999  | |
| +--------------------------------------+ |
|                                          |
| +--------------------------------------+ |
| | (📱) Apple iPhone 15 Pro             | |
| |      Category: Phones          $999  | |
| +--------------------------------------+ |
|                                          |
| [ When SearchAnchor is Tapped (Open) ]:  |
| ┌──────────────────────────────────────┐ |
| │ [←] App|                             │ |
| │--------------------------------------│ |
| │ [📦] Apple MacBook Pro M3     $1,999 │ |
| │ [📦] Apple iPhone 15 Pro        $999 │ |
| └──────────────────────────────────────┘ |
+------------------------------------------+
```

---

## 🌳 Widget Tree
```
MaterialApp
 └── ProductCatalogSearchScreen (StatefulWidget)
      └── Scaffold
           ├── AppBar
           └── SafeArea
                └── Column
                     ├── SearchAnchor [controller: _searchController]
                     │    ├── builder: SearchBar
                     │    └── suggestionsBuilder: List<ListTile>
                     ├── Row (Recent search ActionChips)
                     ├── Divider
                     ├── Row (Results header count)
                     └── Expanded
                          └── ListView.separated
                               └── Card -> ListTile (Filtered Products)
```

---

## ⚙ Flutter Execution Flow
1. **User Taps Bar**: User taps the `SearchBar`.
2. **View Expansion**: `controller.openView()` executes.
3. **Route Overlay**: A full-screen Material search view animates into place with an active keyboard.
4. **Keystroke**: User types `"Son"`.
5. **Suggestions Calculation**: `suggestionsBuilder` executes, filtering `_products`. It returns `[ListTile("Sony WH-1000XM5")]`.
6. **Selection**: User taps the suggestion tile.
7. **View Closure**: `controller.closeView("Sony WH-1000XM5")` pops the overlay.
8. **Catalog Filter**: `_applySearch()` runs; the main list displays the single Sony product, and "Sony" is added to `_recentSearches`.

---

## ⚠ Common Beginner Mistakes

### 1. Recreating `SearchController` Inside `build()`
```dart
// ❌ CATASTROPHIC: New controller on every rebuild!
@override
Widget build(BuildContext context) {
  final controller = SearchController(); // Wipes text and closes search view on keystroke!
  return SearchBar(controller: controller);
}

// ✅ GOOD: Declare in State and dispose cleanly
class _MyState extends State<MyWidget> {
  final SearchController _controller = SearchController();
  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
}
```

### 2. Forgetting to Return an Iterable from `suggestionsBuilder`
```dart
// ❌ COMPILE ERROR: suggestionsBuilder must return Iterable<Widget> or Future<Iterable<Widget>>
suggestionsBuilder: (context, controller) {
  return ListView(...); // Compile error! Not an Iterable<Widget>!
}

// ✅ GOOD: Return a list or iterable of widgets (e.g. ListTiles)
suggestionsBuilder: (context, controller) {
  return matches.map((m) => ListTile(title: Text(m)));
}
```

### 3. Missing `controller.closeView()` on Suggestion Tap
```dart
// ❌ BAD: Tapping suggestion does NOT close the search view!
ListTile(
  title: Text(item),
  onTap: () {
    _doSearch(item); // Search view remains stuck covering the screen!
  },
)

// ✅ GOOD: Close view with selected value
ListTile(
  title: Text(item),
  onTap: () {
    controller.closeView(item);
    _doSearch(item);
  },
)
```

---

## 🚀 Professional Usage
- **App Stores (e.g., Google Play, Apple App Store)**: Top search bars that expand into full-screen trending keywords and autocomplete categories.
- **Media Apps (e.g., Spotify, Apple Music)**: Instant search filtering across Artists, Albums, Songs, and Playlists.
- **Navigation & Maps (e.g., Google Maps)**: Location search bars showing recent history, gas stations, and restaurants.

---

## 💡 Best Practices
1. **Always dispose `SearchController`**: Prevent memory leaks by calling `controller.dispose()` in `State.dispose()`.
2. **Close search views explicitly**: Call `controller.closeView(query)` to smoothly animate back to the parent screen.
3. **Provide immediate feedback on empty queries**: Show recent searches or trending categories when `controller.text.isEmpty`.
4. **Use trailing clear buttons**: Enable quick one-tap clearing so users don't have to backspace entire sentences.

---

## 🧪 Try It Yourself
1. Add an empty-query state in `suggestionsBuilder` that displays recent search history inside the search view.
2. Add a microphone trailing icon that simulates voice search.
3. Customize `SearchBar` elevation and surface tint colors to match your brand style.

---

## 🎯 Challenge
Build an asynchronous GitHub repository search bar:
- In `suggestionsBuilder`, simulate an async network search with a 500ms delay:
  ```dart
  suggestionsBuilder: (context, controller) async { ... }
  ```
- Display a `CircularProgressIndicator` while searching.
- Render repository stars, forks, and language badges in each suggestion tile.

---

## ❓ Practice Questions

### Q1. What is the relationship between `SearchBar` and `SearchAnchor`?
- A) They are identical widgets
- B) `SearchBar` is the visual input bar; `SearchAnchor` coordinates the bar with an expandable search overlay and suggestion list
- C) `SearchBar` is deprecated
- D) `SearchAnchor` only works on Android
*Answer: B* — `SearchAnchor` manages the search bar and its companion suggestions view.

### Q2. What class does `SearchController` extend?
- A) `ChangeNotifier`
- B) `TextEditingController`
- C) `ScrollController`
- D) `AnimationController`
*Answer: B* — It extends `TextEditingController`, adding `openView()` and `closeView()` methods.

### Q3. What must the `suggestionsBuilder` callback return?
- A) A single `Widget`
- B) An `Iterable<Widget>` or `Future<Iterable<Widget>>`
- C) A `List<String>`
- D) `void`
*Answer: B* — It returns an iterable collection of widgets (like `ListTile`s).

### Q4. How do you programmatically close the search view and populate the search bar with `"Flutter"`?
- A) `Navigator.pop(context)`
- B) `controller.closeView('Flutter')`
- C) `controller.dismiss('Flutter')`
- D) `controller.clear()`
*Answer: B* — `closeView(query)` dismisses the view and updates controller text.

### Q5. What happens when the user taps outside or taps the back arrow inside an open `SearchAnchor` view?
- A) The entire app closes
- B) The search view dismisses without updating the search bar text
- C) Flutter throws an error
- D) The search bar clears
*Answer: B* — The overlay route pops, returning the user to the previous screen state.

### Q6. Which property places an icon at the beginning of a `SearchBar`?
*Answer*: `leading: const Icon(Icons.search)`.

### Q7. How do you programmatically open the search suggestions view from code?
*Answer*: Call `controller.openView()`.

### Q8. Output Prediction: What will `controller.text` contain after calling `controller.closeView('Smart Watch')`?
*Answer*: `'Smart Watch'`.

### Q9. Can `suggestionsBuilder` perform asynchronous queries (like fetching suggestions from an API)?
*Answer*: Yes, `suggestionsBuilder` supports returning `Future<Iterable<Widget>>`, allowing asynchronous HTTP requests.

### Q10. Write a minimal code snippet defining a `SearchBar` with a search icon and a clear button.
*Answer*:
```dart
SearchBar(
  controller: _controller,
  leading: const Icon(Icons.search),
  trailing: [
    IconButton(
      icon: const Icon(Icons.clear),
      onPressed: () => _controller.clear(),
    ),
  ],
)
```

---

## 📝 Summary
- [SearchBar](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/21_SearchBar.md) and [SearchAnchor](file:///C:/Users/hari1/LOGIC_BUILDING/flutter_handbook/04_Input_Forms/21_SearchBar.md) provide the official Material 3 search interface in Flutter.
- `SearchAnchor` handles the transition between an in-page search bar and an expanded, full-screen search view.
- `SearchController` provides direct methods to `openView()`, `closeView()`, and manage query text.
- `suggestionsBuilder` reactively renders suggestion widgets as the user types.
- Always dispose `SearchController` in your state's `dispose()` method.
