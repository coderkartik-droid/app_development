# Dart Cheat Sheet

## Most Important Concepts

- Statically typed; type inference uses `var`, `final`, and `const`.
- `final`: assigned once at runtime. `const`: compile-time constant.
- Null safety: `T?` nullable; `!` asserts non-null; `??` supplies fallback.
- Functions, classes, enums, records, patterns, and extensions are core building blocks.
- `Future<T>` represents one eventual value; `Stream<T>` represents multiple async values.

## Syntax Reference

| Need | Syntax |
|---|---|
| Immutable local | `final name = 'Ada';` |
| Constant | `const timeout = Duration(seconds: 5);` |
| Nullable | `String? nickname;` |
| Fallback | `nickname ?? 'Guest'` |
| Function | `int add(int a, int b) => a + b;` |
| Async | `Future<User> load() async { return await fetch(); }` |
| Collection | `final ids = <int>[1, 2, 3];` |
| Map | `final ages = <String, int>{'Ada': 36};` |
| Named args | `User(name: 'Ada', active: true)` |

## Common Classes / Methods

- Collections: `List`, `Set`, `Map`, `Iterable`
- Async: `Future`, `Stream`, `Completer`, `Future.wait`
- Errors: `try`, `on`, `catch`, `finally`, `throw`
- Collection methods: `map`, `where`, `fold`, `any`, `firstWhere`

## Parameters & Properties

- Positional required: `void log(String message)`
- Named required: `void save({required String id})`
- Optional positional: `void greet([String? name])`
- Defaults: `void retry({int attempts = 3})`
- Getters: `int get count => items.length;`

## Short Examples

```dart
final activeNames = users
    .where((user) => user.isActive)
    .map((user) => user.name)
    .toList();

Future<String> greeting() async {
  await Future<void>.delayed(const Duration(milliseconds: 10));
  return 'Hello';
}
```

## Common Mistakes

- Using `var` when the value should never be reassigned; prefer `final`.
- Applying `!` instead of handling a possible `null`.
- Forgetting `await`, which leaves a `Future` rather than its result.
- Mutating a collection while iterating over it.
- Using `const` for values known only at runtime.

## Interview Revision

- `final` vs `const`: one-time runtime assignment vs compile-time constant.
- `List` allows duplicates; `Set` enforces uniqueness; `Map` associates keys and values.
- `Future` completes once; `Stream` can emit repeatedly.
- `async` functions return a `Future`; errors propagate through that future.

## Best Practices

- Prefer immutable values and explicit domain types.
- Use named parameters for APIs with several options.
- Keep async errors observable; avoid empty catch blocks.
- Model nullable values honestly instead of asserting non-null.

## Memory Tricks

- **F**inal = **f**inal assignment; **C**onst = **c**ompile-time.
- **Future once, Stream sequence.**
- `?` permits null; `!` says “I guarantee it isn't null.”

## One-Page Summary

Write typed, null-safe code; choose `final` by default; use `const` for compile-time values; compose collections with `where`/`map`; and model asynchronous work with `Future` or `Stream`. Keep errors explicit and APIs easy to read.
