# Introduction

WebSockets provide a persistent, full-duplex connection between a client and server. Both sides can send messages at any time after the HTTP connection is upgraded.

# Why this concept exists

Chat, multiplayer presence, collaborative editing, and live dashboards need updates in both directions with less overhead than creating a new HTTP request for each message.

# Real-world analogy

An HTTP request is a letter-and-reply exchange. A WebSocket is an open phone call where both people can speak whenever needed.

# Internal Working

The client begins with an HTTP handshake; the server accepts an upgrade and the connection becomes a WebSocket. Messages can be text or binary frames. The app must handle stream errors, closure, reconnect, authentication, heartbeat, event ordering, and duplicate messages.

# Syntax

The `web_socket_channel` package offers a cross-platform stream/sink abstraction:

```dart
final channel = WebSocketChannel.connect(Uri.parse('wss://example.com/socket'));
channel.sink.add('hello');
await for (final message in channel.stream) {
  // Process message.
}
```

# Complete Example 1

This runnable app connects to a public WebSocket echo service. Add `web_socket_channel` to dependencies. Public echo availability may vary; the application handles connection errors.

```dart
import 'package:flutter/material.dart';
import 'package:web_socket_channel/web_socket_channel.dart';

void main() => runApp(const WebSocketApp());

class WebSocketApp extends StatelessWidget {
  const WebSocketApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: WebSocketPage());
}

class WebSocketPage extends StatefulWidget {
  const WebSocketPage({super.key});

  @override
  State<WebSocketPage> createState() => _WebSocketPageState();
}

class _WebSocketPageState extends State<WebSocketPage> {
  late final WebSocketChannel _channel;
  final _input = TextEditingController();
  String _reply = 'Connecting...';

  @override
  void initState() {
    super.initState();
    _channel = WebSocketChannel.connect(
      Uri.parse('wss://ws.postman-echo.com/raw'),
    );
    _channel.stream.listen(
      (message) {
        if (mounted) setState(() => _reply = 'Echo: $message');
      },
      onError: (Object error) {
        if (mounted) setState(() => _reply = 'Connection error');
      },
      onDone: () {
        if (mounted) setState(() => _reply = 'Connection closed');
      },
    );
  }

  void _send() {
    final message = _input.text.trim();
    if (message.isEmpty) return;
    _channel.sink.add(message);
    _input.clear();
  }

  @override
  void dispose() {
    _input.dispose();
    _channel.sink.close();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('WebSocket echo')),
        body: Padding(
          padding: const EdgeInsets.all(20),
          child: Column(
            children: [
              Text(_reply),
              TextField(
                controller: _input,
                decoration: const InputDecoration(labelText: 'Message'),
                onSubmitted: (_) => _send(),
              ),
              FilledButton(onPressed: _send, child: const Text('Send')),
            ],
          ),
        ),
      );
}
```

### Line-by-line explanation

- `WebSocketChannel.connect` opens a secure `wss` connection.
- The listener handles incoming messages, errors, and closure.
- `_send` trims text and adds a message to the sink.
- `dispose` closes the channel and text controller.
- The UI displays connection/reply status and a text entry control.

### Widget tree

```text
MaterialApp → WebSocketPage → Scaffold → Padding → Column
                                                  ├→ reply/status
                                                  ├→ TextField
                                                  └→ Send
```

### Execution flow

```text
Open wss connection -> server echoes
Send text -> sink.add -> WebSocket frame -> server
Receive frame -> stream listener -> setState -> UI
```

### ASCII UI preview

```text
+---------------------------+
|      WebSocket echo       |
+---------------------------+
| Echo: Hello!              |
| Message                   |
| [ Hello!               ]  |
| [          Send         ] |
+---------------------------+
```

### Example explanation

The channel provides a stream for messages and sink for outgoing messages. `wss` encrypts the connection. This echo endpoint is for demonstration, not a production API contract.

If the stream listener is omitted, incoming messages are not processed. If the channel is not closed, the connection can outlive the screen. Production code should retain and cancel the `StreamSubscription` explicitly when appropriate.

**Common mistakes:** Do not create a new WebSocket connection on every widget rebuild.

# Complete Example 2

Send structured JSON messages:

```dart
channel.sink.add(jsonEncode({
  'type': 'chat.send',
  'text': message,
}));
```

Parse and validate the incoming envelope before updating chat state.

# Complete Example 3

Reconnect after closure with exponential backoff and jitter. Reset the backoff only after a stable connection, not immediately on each handshake.

# Advanced Example

Production WebSockets need authentication strategy, heartbeat/ping handling, message schema versioning, size limits, backpressure, reconnect, resume cursor, idempotency, and app lifecycle handling. Avoid sending long-lived secrets in the URL; use secure handshake mechanisms supported by the server/platform.

# Under the Hood

An HTTP handshake negotiates protocol upgrade. WebSocket frames then travel over a persistent connection. The package exposes asynchronous events; Flutter state management translates them into UI state.

# Performance Notes

- Reuse one connection per logical session.
- Batch high-volume messages if appropriate.
- Limit message sizes and parsing cost.
- Stop connections when feature/app lifecycle requires it.

# Best Practices

- Use `wss`.
- Validate message type and schema.
- Handle errors, done, reconnect, and disposal.
- Deduplicate with event IDs.
- Keep credentials out of logs and URLs.

# Common Mistakes

- No reconnect/backoff.
- Ignoring stream error/done.
- Leaking channel/subscription.
- Assuming exactly-once delivery.
- Sending arbitrary unvalidated JSON to domain state.

# Debugging

- Inspect handshake status, close code, and connection lifecycle.
- Test offline transitions and server restarts.
- Log message type/ID, not sensitive content.
- Verify proxies/firewalls permit upgrades.

# Interview Questions

1. What does full-duplex mean? Both sides can send independently.
2. Why use `wss`? Encrypted WebSocket transport.
3. Does a WebSocket guarantee delivery through disconnect? No.
4. What is required after reconnect? Resume/reconcile and deduplicate.

# Try It Yourself

1. Add JSON message types.
2. Display connection state separately from last message.
3. Add reconnect backoff.
4. Close channel when navigating away.

# Challenge Project

Build a chat screen with typed events, optimistic outgoing messages, delivery IDs, reconnect, duplicate suppression, and history reconciliation.

# Quick Revision Sheet

- WebSocket is persistent and bidirectional.
- Use `wss`, handle stream/sink lifecycle.
- Reconnect, deduplicate, and resume.
- Validate every message.
- Close resources at owner lifecycle.
