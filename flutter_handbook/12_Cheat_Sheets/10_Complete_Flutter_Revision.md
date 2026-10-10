# Complete Flutter Revision

## Most Important Concepts

- **Dart:** typed, null-safe language; `Future` for one result, `Stream` for many.
- **Widgets:** immutable UI descriptions composed into a tree.
- **Layout:** constraints down, sizes up, parent positions.
- **State:** changing data that affects UI; choose ownership and observation deliberately.
- **Navigation:** routes are pushed, popped, and may return typed results.
- **Animation:** controller drives progress; tween maps values; curve shapes timing.
- **Persistence:** storage must match data shape; database schema evolves through migrations.
- **Networking:** request/response is asynchronous and failure-prone.

## Syntax Reference

```dart
final count = 3;
String? nickname;
final label = nickname ?? 'Guest';

class HomePage extends StatefulWidget {
  const HomePage({super.key});
  @override
  State<HomePage> createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  int value = 0;
  @override
  Widget build(BuildContext context) => Scaffold(
        body: Center(child: Text('$value')),
        floatingActionButton: FloatingActionButton(
          onPressed: () => setState(() => value++),
          child: const Icon(Icons.add),
        ),
      );
}
```

## Common Widgets / Classes

| Topic | Core APIs |
|---|---|
| App/UI | `MaterialApp`, `Scaffold`, `AppBar`, `SafeArea` |
| Layout | `Row`, `Column`, `Wrap`, `Stack`, `Expanded`, `LayoutBuilder` |
| Input/lists | `TextField`, `Form`, `ListView.builder`, `ListTile` |
| State | `StatefulWidget`, `setState`, `ChangeNotifier`, `StreamBuilder` |
| Navigation | `Navigator`, `MaterialPageRoute`, declarative Router |
| Animation | `AnimationController`, `Tween`, `CurvedAnimation`, `AnimatedBuilder` |
| Database | `sqflite`, Drift, SQL constraints/transactions |
| Networking | `http.Client`, `Uri`, `jsonDecode`, typed models |

## Frequently Used Methods

- `setState`: schedule rebuild after local state changes.
- `initState` / `dispose`: initialize and release state-owned resources.
- `Navigator.push` / `pop`: move between routes and return values.
- `FutureBuilder` / `StreamBuilder`: render async results.
- Database: `query`, `insert`, `update`, `delete`, `transaction`.
- Collections: `where`, `map`, `fold`.

## Parameters & Properties

- Widget identity: `key`
- Flex layout: `mainAxisAlignment`, `crossAxisAlignment`
- Scrolling: `itemCount`, `itemBuilder`, `physics`
- Async: loading, data, error states
- HTTP: method, URI, headers, body, status
- Database: path, schema version, predicates, bound args, limit
- Animation: duration, curve, vsync, begin/end

## Short Examples

**Layout**

```dart
Row(children: [
  const Icon(Icons.person),
  const SizedBox(width: 8),
  Expanded(child: Text(name)),
])
```

**Parameterized database read**

```dart
final rows = await db.query(
  'tasks',
  where: 'id = ?',
  whereArgs: [id],
);
```

**Typed route result**

```dart
final selected = await Navigator.of(context).push<String>(route);
```

## Common Mistakes

- Doing I/O or state mutation in `build`.
- Ignoring nullability, async errors, and mounted checks.
- Unbounded scrollables or unnecessary fixed sizes.
- Missing controller disposal and unstable list keys.
- Assuming local data syncs or network requests always succeed.
- Updating/deleting database rows without a predicate.
- Embedding secrets or trusting client-side totals.

## Interview Revision

1. Widgets are immutable; state/elements preserve runtime identity.
2. `final` allows one runtime assignment; `const` is compile-time constant.
3. `Future` completes once; `Stream` can emit repeatedly.
4. Use repository boundaries to separate UI from data access.
5. A transaction makes related writes atomic.
6. An index trades storage/write cost for selected read performance.
7. A migration updates existing schemas without discarding data.
8. API responses require status and shape validation.

## Best Practices

- Keep UI declarative and side effects outside `build`.
- Prefer small composable widgets and typed models.
- Make loading, empty, success, and error states explicit.
- Dispose owned controllers and subscriptions.
- Validate inputs and use parameterized SQL.
- Test narrow layouts, async failures, persistence, and migrations.
- Add dependencies only for clear requirements; pin and update them carefully.

## Memory Tricks

- **Layout:** constraints down, sizes up, parent positions.
- **Animation:** controller = clock; tween = range; curve = feel.
- **Database:** keys relate; transactions group; migrations evolve.
- **API:** request = method + URI + headers + body.
- **State:** one source of truth; derived values are computed.

## One-Page Summary

Build Flutter screens by composing widgets and respecting constraints. Keep state ownership clear, validate input, and manage lifecycles. Use typed repositories for database or network work; await operations, handle errors, and update UI state deliberately. Persist structured data with constraints, transactions, indexes chosen by measurement, and tested migrations. Keep interfaces accessible, responsive, and covered by tests.
