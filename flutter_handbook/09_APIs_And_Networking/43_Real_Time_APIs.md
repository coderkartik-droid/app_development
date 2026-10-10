# Introduction

Real-time APIs deliver updates while an app is open instead of requiring the app to repeatedly ask whether anything changed. Examples include chat messages, live sports scores, delivery tracking, and collaborative editing.

“Real-time” usually means low-latency updates, not a guarantee that every event arrives instantly. Networks disconnect, devices sleep, and servers may throttle connections.

# Why this concept exists

Polling can be wasteful when updates are frequent or unpredictable. A server-initiated channel can deliver changes as they occur, improving responsiveness and reducing repeated empty requests.

# Real-world analogy

Polling is repeatedly calling a friend to ask whether the bus has arrived. A real-time channel is the friend staying on the phone and telling you when it arrives.

# Internal Working

Common approaches include:

- **Polling:** The client periodically requests changes.
- **Long polling:** The server holds a request open until data is available or a timeout occurs.
- **WebSockets:** A persistent, bidirectional connection.
- **Server-Sent Events (SSE):** A persistent, server-to-client event stream over HTTP.

Every persistent channel needs connection lifecycle handling: opening, authentication, heartbeat/keepalive where required, reconnect backoff, duplicate-event handling, and cleanup. A transport connection does not replace a durable event history; clients may miss messages while disconnected.

# Syntax

```text
connect -> authenticate -> receive event -> validate -> update state
         -> disconnect -> reconnect with backoff -> resume/reconcile
```

# Complete Example 1

This runnable app demonstrates periodic polling with a fake service. It avoids a real backend while teaching timer ownership. Polling is simple but is not a push channel.

```dart
import 'dart:async';
import 'package:flutter/material.dart';

class ScoreService {
  int _score = 0;

  Future<int> fetchScore() async {
    await Future<void>.delayed(const Duration(milliseconds: 150));
    return ++_score;
  }
}

void main() => runApp(const RealtimeApiApp());

class RealtimeApiApp extends StatelessWidget {
  const RealtimeApiApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: LiveScorePage());
}

class LiveScorePage extends StatefulWidget {
  const LiveScorePage({super.key});

  @override
  State<LiveScorePage> createState() => _LiveScorePageState();
}

class _LiveScorePageState extends State<LiveScorePage> {
  final _service = ScoreService();
  Timer? _timer;
  int? _score;
  String _status = 'Connecting';

  @override
  void initState() {
    super.initState();
    _refresh();
    _timer = Timer.periodic(const Duration(seconds: 2), (_) => _refresh());
  }

  Future<void> _refresh() async {
    try {
      final value = await _service.fetchScore();
      if (!mounted) return;
      setState(() {
        _score = value;
        _status = 'Updated just now';
      });
    } catch (error) {
      if (!mounted) return;
      setState(() => _status = 'Update unavailable');
    }
  }

  @override
  void dispose() {
    _timer?.cancel();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Live score demo')),
        body: Center(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              Text(_score?.toString() ?? '—',
                  style: Theme.of(context).textTheme.displayLarge),
              Text(_status),
              FilledButton(onPressed: _refresh, child: const Text('Refresh now')),
            ],
          ),
        ),
      );
}
```

### Line-by-line explanation

- `ScoreService` simulates a remote score source and increments it on each read.
- The page owns a `Timer`, latest score, and status.
- `initState` loads immediately and schedules later refreshes every two seconds.
- `_refresh` awaits the service, checks `mounted`, and updates either score or status.
- `dispose` cancels the periodic timer so it does not continue after leaving the page.
- The UI shows a placeholder before the first result, the current score, and manual refresh.

### Widget tree

```text
MaterialApp → LiveScorePage → Scaffold → Center → Column
                                               ├→ score
                                               ├→ update status
                                               └→ refresh action
```

### Execution flow

```text
Timer fires -> request latest score -> success/failure -> update state -> rebuild
Page disposed -> cancel timer
```

### ASCII UI preview

```text
+---------------------------+
|      Live score demo      |
+---------------------------+
|            8              |
| Updated just now          |
| [      Refresh now      ] |
+---------------------------+
```

### Example explanation

This uses polling to approximate live updates. A real server might change between polls, and the client only notices at its next scheduled read. The timer interval trades freshness for network and battery use.

If the timer is not canceled, requests continue after the screen is gone. If multiple refreshes overlap, results can arrive out of order; production code should serialize polling or ignore stale responses.

**Common mistakes:** Do not poll every fraction of a second without a product and battery rationale.

**Professional use case:** Low-frequency status refresh where push infrastructure is unavailable or unnecessary.

# Complete Example 2

Use a stream abstraction to keep transport separate from UI:

```dart
Stream<ScoreUpdate> watchScore();
```

The service can implement this with WebSockets or SSE; a state object subscribes and cancels the subscription in its lifecycle cleanup.

# Complete Example 3

On reconnect, reconcile from a server cursor:

```text
GET events?after=<lastSeenEventId>
```

This fills gaps missed while a live connection was unavailable, if the server supports event history.

# Advanced Example

For production real-time behavior, define event IDs, ordering, deduplication, resume cursors, heartbeat policy, backoff with jitter, auth refresh, app background behavior, and fallback reconciliation. Avoid assuming every event is delivered exactly once.

# Under the Hood

Polling repeatedly creates HTTP requests. WebSockets upgrade a connection and allow messages in both directions. SSE streams text events from server to client. Long polling keeps an HTTP request open until an event or timeout. Each option has different proxy, mobile lifecycle, and server-scaling behavior.

# Performance Notes

- Prefer push channels when low latency is required and infrastructure supports them.
- Use backoff and stop work when the feature is not active.
- Batch or coalesce events before expensive UI updates.
- Avoid rebuilding a large page for every message.

# Best Practices

- Choose the simplest transport meeting latency needs.
- Treat reconnect as normal.
- Resume/reconcile missed changes.
- Deduplicate by event ID.
- Cancel streams and timers when no longer needed.
- Respect battery and background execution limits.

# Common Mistakes

- Treating a socket connection as guaranteed delivery.
- No reconnect or backoff strategy.
- Duplicate event application.
- Polling too frequently.
- Leaving subscriptions active after screen disposal.

# Debugging

- Log connection lifecycle, event ID, and timing without sensitive payloads.
- Simulate airplane mode and app backgrounding.
- Check duplicate/out-of-order events.
- Compare server history with last-seen cursor.

# Interview Questions

1. What is the difference between polling and WebSockets? Polling is client initiated at intervals; WebSockets provide a persistent bidirectional channel.
2. What is SSE? A server-to-client event stream over HTTP.
3. Why need reconciliation? Events can be missed while disconnected.
4. Why deduplicate? Reconnects and retries can deliver repeated events.

# Try It Yourself

1. Change the polling interval and discuss battery/freshness trade-offs.
2. Add an in-flight request guard.
3. Model last-seen event ID.
4. Simulate reconnection and duplicate updates.

# Challenge Project

Build a live delivery-status screen with connection state, event ID deduplication, reconnect backoff, and history reconciliation.

# Quick Revision Sheet

- Real-time is a latency goal, not an absolute delivery guarantee.
- Polling, long polling, WebSockets, and SSE have different trade-offs.
- Reconnect, deduplicate, resume, and reconcile.
- Cancel resources with their owner.
- Profile battery, network, and rebuild costs.
