# Introduction

`Hero` animation is one of the most recognizable animations in Flutter.

It is used when an element appears to move from one screen to another, creating a strong visual continuity between pages.

Typical examples include:

- image moving from a list to a detail screen
- product thumbnails transitioning to a detail page
- avatar moving into a profile view

The hero animation is part of the app's visual narrative. It tells the user, "This item is the same object, just now shown in a different place."

# Why this animation exists

Navigation often changes the screen context drastically. A hero animation helps keep the object identity clear.

Without it, the user could lose track of where the item came from. With a hero, the user sees a smooth transition that reinforces continuity.

# Real-world analogy

Think of a performer stepping from a stage into a spotlight.

The audience sees the same person, now in a different environment. The movement says, "It is the same person, changing context."

A hero animation works in the same way for UI objects.

# Internal Working

Flutter hero animation uses a shared tag to match one widget in one page to another widget in another page.

When a route changes:

1. the source hero is identified with a tag
2. the destination hero uses the same tag
3. Flutter creates two versions of the same widget and animates between them
4. the user sees the object move across screens

The tag is the key that links the two widgets.

# Syntax

```dart
Hero(
  tag: 'profile-image',
  child: Image.network('https://example.com/image.jpg'),
)
```

Then on the destination page, use the same tag.

```dart
Hero(
  tag: 'profile-image',
  child: Image.network('https://example.com/image.jpg'),
)
```

# Complete Example 1

```dart
import 'package:flutter/material.dart';

void main() => runApp(const HeroApp());

class HeroApp extends StatelessWidget {
  const HeroApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: const HomePage(),
    );
  }
}

class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Hero Animation')),
      body: Center(
        child: GestureDetector(
          onTap: () {
            Navigator.push(
              context,
              MaterialPageRoute(builder: (_) => const DetailsPage()),
            );
          },
          child: Hero(
            tag: 'avatar',
            child: Container(
              width: 120,
              height: 120,
              decoration: const BoxDecoration(
                shape: BoxShape.circle,
                color: Colors.blue,
              ),
              child: const Center(
                child: Text(
                  'A',
                  style: TextStyle(color: Colors.white, fontSize: 42),
                ),
              ),
            ),
          ),
        ),
      ),
    );
  }
}

class DetailsPage extends StatelessWidget {
  const DetailsPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Details')),
      body: Center(
        child: Hero(
          tag: 'avatar',
          child: Container(
            width: 220,
            height: 220,
            decoration: const BoxDecoration(
              shape: BoxShape.circle,
              color: Colors.blue,
            ),
            child: const Center(
              child: Text(
                'A',
                style: TextStyle(color: Colors.white, fontSize: 72),
              ),
            ),
          ),
        ),
      ),
    );
  }
}
```

## Line-by-line explanation

- `Hero` wraps both source and destination widgets.
- The same `tag` links them.
- Tapping the avatar pushes a new screen.
- Flutter animates the widget between the two positions and sizes.
- The result is a smooth identity-preserving transition.

## Widget tree

```text
HomePage
└── Scaffold
    └── Center
        └── GestureDetector
            └── Hero
                └── Container
                    └── Text

DetailsPage
└── Scaffold
    └── Center
        └── Hero
            └── Container
                └── Text
```

## Execution flow

```text
User taps source hero
   |
   v
Navigator pushes DetailsPage
   |
   v
Flutter recognizes same hero tag
   |
   v
Transition animation moves and resizes object
   |
   v
Detail page shows the same item in new context
```

## ASCII phone preview

```text
Home screen               Details screen
┌───────────────┐      ┌───────────────┐
authored by : hero  │      │ authored by : hero |
│   [A]         │      │      [A]       │
│               │      │               │
│               │      │               │
│               │      │               │
└───────────────┘      └───────────────┘
```

## Example explanation

The user sees the same object move and reshape as the screen changes, which creates a stronger understanding of continuity. Without the hero tag, the object would appear as a new unrelated widget.

## Common mistakes

- using different tags on the source and destination
- not placing both heroes in the same route flow
- using hero on too many unrelated UI elements

## Professional use case

Used in product detail screens, galleries, chat avatars, and app navigation flows where the object should feel persistent through the transition.

# Complete Example 2

```dart
import 'package:flutter/material.dart';

void main() => runApp(const HeroImageApp());

class HeroImageApp extends StatelessWidget {
  const HeroImageApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: const GalleryPage(),
    );
  }
}

class GalleryPage extends StatelessWidget {
  const GalleryPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Gallery')),
      body: GridView.count(
        crossAxisCount: 2,
        children: [
          GestureDetector(
            onTap: () => Navigator.push(
              context,
              MaterialPageRoute(builder: (_) => const DetailImagePage()),
            ),
            child: Hero(
              tag: 'photo-1',
              child: Container(
                margin: const EdgeInsets.all(8),
                color: Colors.orange,
                child: const Center(child: Text('Photo')),
              ),
            ),
          ),
        ],
      ),
    );
  }
}

class DetailImagePage extends StatelessWidget {
  const DetailImagePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Photo')),
      body: Center(
        child: Hero(
          tag: 'photo-1',
          child: Container(
            width: 220,
            height: 220,
            color: Colors.orange,
            child: const Center(child: Text('Photo', style: TextStyle(fontSize: 28))),
          ),
        ),
      ),
    );
  }
}
```

This shows a thumbnail-to-detail transition.

# Complete Example 3

```dart
import 'package:flutter/material.dart';

void main() => runApp(const HeroAvatarApp());

class HeroAvatarApp extends StatelessWidget {
  const HeroAvatarApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: const ListPage(),
    );
  }
}

class ListPage extends StatelessWidget {
  const ListPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: ListView(
        children: [
          ListTile(
            leading: const Hero(
              tag: 'user-avatar',
              child: CircleAvatar(
                radius: 24,
                child: Text('H'),
              ),
            ),
            title: const Text('Hari'),
            onTap: () {
              Navigator.push(
                context,
                MaterialPageRoute(builder: (_) => const ProfilePage()),
              );
            },
          ),
        ],
      ),
    );
  }
}

class ProfilePage extends StatelessWidget {
  const ProfilePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Profile')),
      body: Center(
        child: Hero(
          tag: 'user-avatar',
          child: const CircleAvatar(
            radius: 72,
            child: Text('H', style: TextStyle(fontSize: 52)),
          ),
        ),
      ),
    );
  }
}
```

This is a very common hero implementation for avatars and user identity.

# Advanced Example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const HeroTransitionApp());

class HeroTransitionApp extends StatelessWidget {
  const HeroTransitionApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: const StartingPage(),
    );
  }
}

class StartingPage extends StatelessWidget {
  const StartingPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: GestureDetector(
          onTap: () => Navigator.push(
            context,
            MaterialPageRoute(builder: (_) => const TargetPage()),
          ),
          child: Hero(
            tag: 'market-card',
            child: Container(
              width: 180,
              height: 110,
              decoration: BoxDecoration(
                color: Colors.teal,
                borderRadius: BorderRadius.circular(18),
              ),
              child: const Center(
                child: Text(
                  'Product',
                  style: TextStyle(color: Colors.white, fontSize: 24),
                ),
              ),
            ),
          ),
        ),
      ),
    );
  }
}

class TargetPage extends StatelessWidget {
  const TargetPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Product Page')),
      body: Center(
        child: Hero(
          tag: 'market-card',
          child: Container(
            width: 260,
            height: 180,
            decoration: BoxDecoration(
              color: Colors.teal,
              borderRadius: BorderRadius.circular(18),
            ),
            child: const Center(
              child: Text(
                'Product',
                style: TextStyle(color: Colors.white, fontSize: 32),
              ),
            ),
          ),
        ),
      ),
    );
  }
}
```

This is a product or card detail transition that focuses on continuity.

# Under the Hood

The hero animation is part of Flutter's route transition system. The framework looks for `Hero` widgets with matching tags and animates between their bounds and positions when navigating.

The key is identity. The user sees the same conceptual object moving and resizing across screens rather than a completely unrelated widget appearing.

# Performance Notes

- hero animations work best for a small number of important visual objects
- avoid using hero tags on every widget in a list
- keep the wrapped widget simple and meaningful

# Best Practices

- use hero animations only for strong object continuity
- choose unique tags per object identity
- make sure the same logical item appears on both screens

# Common Mistakes

- using the same tag for different objects
- switching pages without matching hero tags
- overusing hero animation on too many widgets

# Debugging

If the hero does not animate:

1. check that both widgets use the exact same tag
2. verify there is a route push or navigation transition
3. confirm the widgets are not different types with conflicting state
4. ensure the hero is not hidden behind another widget

# Interview Questions

1. What does `Hero` animation do?
2. Why is the `tag` property important?
3. What is the value of a hero transition in navigation?
4. How does Flutter know which widgets match?
5. What kind of app features often use hero animations?
6. Why is identity continuity important for user experience?
7. What happens if the tags differ?
8. Why is it important not to overuse hero transitions?
9. Which widget is usually involved in route transition?
10. How would you explain hero animation to a beginner?

# Try It Yourself

Use a `Hero` animation for a profile image between a list and a detail view. Try to make the visual continuity seamless.

# Challenge Project

Build a shopping app landing page where a product card hero transitions into a product detail page with a larger image and title.

# Quick Revision Sheet

- `Hero` animation connects the same item across route changes
- `tag` identifies the item across two screens
- it improves continuity and navigation comprehension
- best for important objects such as avatars, cards, or product images
- use a unique tag for each object
- it is a storytelling animation, not just a decoration
