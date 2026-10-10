# Introduction

API architecture describes how networking code is organized in a Flutter app. A common layering is UI, state management, repository, API client, and server.

# Why this concept exists

If widgets own HTTP details, parsing, caching, and retry logic, code becomes difficult to test and reuse. Layers give each part a clear responsibility.

# Real-world analogy

A restaurant has a host, waiter, kitchen coordinator, and chef. Each role has a specific responsibility, but together they deliver the meal.

# Internal Working

The UI sends user intent to state management. A repository coordinates data sources and maps results. An API client handles HTTP transport and serialization. Models represent wire/domain data. Errors move back as typed outcomes that the UI can render.

# Syntax

```text
Widget -> notifier/cubit -> repository -> API client -> server
Widget <- state <- result/error <- mapped response <-
```

# Complete Example 1

This runnable app demonstrates a service/repository separation with an API client and typed model. Add `http`.

```dart
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

class Quote {
  const Quote(this.text);
  final String text;
}

class QuoteApi {
  QuoteApi(this.client);
  final http.Client client;

  Future<Quote> fetch() async {
    final response = await client.get(
      Uri.https('jsonplaceholder.typicode.com', '/posts/1'),
    );
    if (response.statusCode != 200) throw Exception('HTTP ${response.statusCode}');
    final json = jsonDecode(response.body) as Map<String, dynamic>;
    return Quote(json['title'] as String);
  }
}

class QuoteRepository {
  QuoteRepository(this.api);
  final QuoteApi api;
  Future<Quote> loadQuote() => api.fetch();
}

void main() => runApp(const ArchitectureApp());

class ArchitectureApp extends StatelessWidget {
  const ArchitectureApp({super.key});

  @override
  Widget build(BuildContext context) =>
      MaterialApp(home: ArchitecturePage(repository: QuoteRepository(QuoteApi(http.Client()))));
}

class ArchitecturePage extends StatefulWidget {
  const ArchitecturePage({required this.repository, super.key});
  final QuoteRepository repository;

  @override
  State<ArchitecturePage> createState() => _ArchitecturePageState();
}

class _ArchitecturePageState extends State<ArchitecturePage> {
  String _text = 'Tap load';

  Future<void> _load() async {
    try {
      final quote = await widget.repository.loadQuote();
      if (mounted) setState(() => _text = quote.text);
    } catch (error) {
      if (mounted) setState(() => _text = 'Load failed');
    }
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('API layers')),
        body: Center(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              Text(_text, textAlign: TextAlign.center),
              FilledButton(onPressed: _load, child: const Text('Load')),
            ],
          ),
        ),
      );
}
```

### Line-by-line explanation

- `Quote` is the value shown by the UI.
- `QuoteApi` owns HTTP transport and JSON parsing.
- `QuoteRepository` exposes application-facing data behavior and can later add cache/local source.
- The app constructs dependencies and supplies the repository to the page.
- The page handles user intent and a simple visible state.
- `mounted` prevents updates after disposal.

### Widget tree

```text
ArchitectureApp
└── ArchitecturePage
    └── Scaffold → Text + Load button

Data path: Page → Repository → API client → server
```

### Execution flow

```text
Tap -> Page -> Repository -> API client -> HTTP -> JSON -> Quote
     <- state update <- mapped result <-
```

### ASCII UI preview

```text
+---------------------------+
|          API layers       |
+---------------------------+
| sunt aut facere...        |
| [          Load         ] |
+---------------------------+
```

### Example explanation

The API class knows HTTP and JSON; the repository gives the app a place to add caching or combine local/remote sources. A production app should own and close the `http.Client` at a well-defined lifecycle, inject it, and use typed errors.

If transport code moves into `build`, requests repeat. If the repository layer is omitted, every screen may duplicate caching and mapping decisions.

**Common mistakes:** Avoid layers that only forward calls without providing a meaningful boundary; architecture should simplify tests and change.

# Complete Example 2

Inject a repository:

```dart
class ProductPage extends StatelessWidget {
  const ProductPage({required this.repository, super.key});
  final ProductRepository repository;
}
```

Tests can supply a fake; production supplies the real implementation.

# Complete Example 3

Map transport errors at repository boundary:

```dart
try {
  return await api.fetch();
} on SocketException {
  throw const OfflineFailure();
}
```

Use actual exception classes and preserve causes for diagnostics.

# Advanced Example

Keep dependencies constructible for tests, use feature modules, centralize auth/timeouts, and avoid global mutable clients. Add observability with request IDs and metrics. Keep domain rules out of widgets and HTTP interceptors.

# Under the Hood

Each layer transforms data/responsibility: widget events become commands; repositories select data sources; clients construct protocol messages; models map representations; state objects notify UI.

# Performance Notes

- Reuse clients and repositories.
- Avoid redundant calls and transformations.
- Use cache at repository/source boundary.
- Keep large parsing away from frame-critical work.

# Best Practices

- Make layer responsibilities explicit.
- Inject dependencies.
- Use typed models/errors.
- Keep request policy centralized.
- Test each layer independently and end-to-end where important.

# Common Mistakes

- Networking in `build`.
- A “god service” with UI state and persistence mixed in.
- Too many pass-through abstractions.
- Global client lifecycle ambiguity.
- Error mapping duplicated across screens.

# Debugging

- Trace operation through each layer.
- Use request IDs across logs.
- Test fake repository and real client separately.
- Verify client is closed by its owner.

# Interview Questions

1. What is repository responsibility? Abstract data sources and expose app-oriented operations.
2. What belongs in API client? HTTP construction, transport, and wire serialization.
3. Why dependency injection? Replace implementations and test deterministically.
4. Is more layering always better? No; abstractions should serve clear boundaries.

# Try It Yourself

1. Add local cache inside repository.
2. Inject a fake API for tests.
3. Move error mapping out of widget.
4. Diagram your feature's request path.

# Challenge Project

Refactor an API-driven screen into UI/state/repository/client/model layers with a fake test implementation and clear lifecycle ownership.

# Quick Revision Sheet

- UI expresses intent; state handles presentation.
- Repository abstracts data source.
- API client owns transport/serialization.
- Models and errors are typed.
- Inject dependencies and define client lifecycle.
