# Project Overview

Build a polished Flutter profile screen that presents a person's identity, short biography, interests, and contact actions. The project focuses on reusable widgets, layout, accessibility, and responsive constraints. It uses local sample data and needs no external package.

# Features

- Profile avatar with a graceful initials fallback
- Name, role, biography, and compact statistics
- Interest chips
- Accessible contact action with visible feedback
- Scrollable, centered layout suitable for small screens

# Folder Structure

```text
lib/
  main.dart
  models/profile.dart        # Optional extraction as the app grows
  widgets/profile_header.dart
  widgets/stat_card.dart
```

For the first pass, put the complete code in `lib/main.dart`; split it into the folders above when more screens or tests are added.

# ASCII UI Preview

```text
+--------------------------+
|        Profile           |
|           ( A )           |
|       Alex Morgan         |
|    Flutter Developer      |
| Builds thoughtful apps... |
|   12 posts  840 followers |
| [Flutter] [Design] [UX]   |
| [       Contact       ]  |
+--------------------------+
```

# Step-by-Step Development Plan

1. Start with a `MaterialApp` and a `Scaffold`.
2. Constrain the content width so the profile stays readable on tablets.
3. Compose the avatar, text, statistics, and interests from small widgets.
4. Make the contact action keyboard- and screen-reader-accessible.
5. Test narrow width, large text scaling, and long names.

# Complete Runnable Code

Paste this into `lib/main.dart` in a Flutter project and run it. It uses Flutter's built-in Material library only.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const ProfileApp());

class ProfileApp extends StatelessWidget {
  const ProfileApp({super.key});

  @override
  Widget build(BuildContext context) => MaterialApp(
        debugShowCheckedModeBanner: false,
        theme: ThemeData(
          colorScheme: ColorScheme.fromSeed(seedColor: const Color(0xFF6750A4)),
          useMaterial3: true,
        ),
        home: const ProfilePage(),
      );
}

class ProfilePage extends StatelessWidget {
  const ProfilePage({super.key});

  @override
  Widget build(BuildContext context) {
    final colors = Theme.of(context).colorScheme;
    return Scaffold(
      appBar: AppBar(title: const Text('Profile')),
      body: SafeArea(
        child: Center(
          child: ConstrainedBox(
            constraints: const BoxConstraints(maxWidth: 520),
            child: ListView(
              padding: const EdgeInsets.all(24),
              children: [
                CircleAvatar(
                  radius: 52,
                  backgroundColor: colors.primaryContainer,
                  child: Text(
                    'AM',
                    style: Theme.of(context).textTheme.headlineMedium,
                  ),
                ),
                const SizedBox(height: 16),
                Text(
                  'Alex Morgan',
                  textAlign: TextAlign.center,
                  style: Theme.of(context).textTheme.headlineSmall,
                ),
                Text(
                  'Flutter Developer',
                  textAlign: TextAlign.center,
                  style: Theme.of(context).textTheme.titleMedium,
                ),
                const SizedBox(height: 12),
                const Text(
                  'I build thoughtful mobile experiences and enjoy making '
                  'complex ideas feel simple.',
                  textAlign: TextAlign.center,
                ),
                const SizedBox(height: 24),
                const Row(
                  mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                  children: [
                    _Stat(label: 'Projects', value: '12'),
                    _Stat(label: 'Followers', value: '840'),
                    _Stat(label: 'Following', value: '210'),
                  ],
                ),
                const SizedBox(height: 24),
                Wrap(
                  alignment: WrapAlignment.center,
                  spacing: 8,
                  runSpacing: 8,
                  children: const [
                    Chip(label: Text('Flutter')),
                    Chip(label: Text('UI design')),
                    Chip(label: Text('Accessibility')),
                  ],
                ),
                const SizedBox(height: 20),
                FilledButton.icon(
                  onPressed: () => ScaffoldMessenger.of(context).showSnackBar(
                    const SnackBar(content: Text('Contact action selected')),
                  ),
                  icon: const Icon(Icons.mail_outline),
                  label: const Text('Contact'),
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }
}

class _Stat extends StatelessWidget {
  const _Stat({required this.label, required this.value});
  final String label;
  final String value;

  @override
  Widget build(BuildContext context) => Column(
        children: [
          Text(value, style: Theme.of(context).textTheme.titleLarge),
          Text(label),
        ],
      );
}
```

# Line-by-Line Explanation

- `main` starts the widget tree with `ProfileApp`.
- `MaterialApp` supplies navigation, theming, and Material defaults. `ColorScheme.fromSeed` creates a coordinated palette.
- `ProfilePage` uses a `Scaffold` with an app bar and a safe content area.
- `Center` and `ConstrainedBox` avoid excessively wide content on larger screens.
- `ListView` makes the whole page scroll when text scaling or screen height requires it.
- `CircleAvatar`, headings, and biography form the identity section. Initials avoid a broken network-image dependency.
- `_Stat` is a small reusable widget; the `Row` arranges three statistics.
- `Wrap` moves chips onto a new line on narrow displays rather than overflowing.
- `FilledButton.icon` makes the contact action discoverable; `SnackBar` confirms the tap.

Removing the `ListView` scroll behavior can cause a bottom overflow on small screens. Removing `ConstrainedBox` makes lines overly long on tablets. Replacing the button callback with `null` disables the action.

# Widget Tree

```text
ProfileApp
└── MaterialApp
    └── ProfilePage
        └── Scaffold
            ├── AppBar
            └── SafeArea > Center > ConstrainedBox > ListView
                ├── Avatar + identity text
                ├── Statistics row
                ├── Interest chips
                └── Contact button
```

# State Management Used

The profile is immutable sample content, so a `StatelessWidget` is sufficient. The contact confirmation is transient framework UI state managed by `ScaffoldMessenger`; no global state package is warranted.

# API/Database Used (if applicable)

None. Replace sample values with a typed profile model when data comes from a database or another source; do not fetch data inside `build`.

# Common Mistakes

- Using a fixed-width `Row` for content that should wrap.
- Omitting scroll behavior for larger accessibility text.
- Using an unlabeled icon as the only contact control.
- Placing personal contact details in a public profile without consent.

# Improvements

- Add a profile model and avatar asset with loading/error fallback.
- Add edit mode and form validation.
- Add semantic labels for statistics and test large text scaling.
- Localize labels and support right-to-left layouts.

# Challenge Version

Create an editable profile page that validates a display name, lets users choose interests, and shows a discard-changes confirmation without losing unsaved input.

# Summary

This first project practices clean composition, responsive constraints, accessible actions, and reusable stat widgets. It deliberately stays data-local so layout fundamentals remain the focus.
