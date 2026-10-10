# Introduction

Encrypted local data is information transformed with a cryptographic key so that reading the stored bytes alone does not reveal the original content. Encryption can protect a database or file at rest, but it does not automatically protect data while the app is running.

# Why this concept exists

Apps may store sensitive journals, health records, or business data. If a threat model requires protection beyond the operating system sandbox, encryption at rest may be appropriate.

# Real-world analogy

Encryption is a locked document box. The lock helps only if the key is protected separately; hiding the key beside the box defeats the protection.

# Internal Working

Encryption algorithms transform plaintext to ciphertext using a key. Decryption requires the correct key. A secure design uses vetted cryptographic libraries, random nonces/IVs where required, authenticated encryption, and a key lifecycle backed by platform secure storage or a user-derived key strategy.

Do not invent cryptography. Database encryption plugins have platform/version constraints. Decide what happens when keys are lost: data may become unrecoverable.

# Syntax

Illustrative pseudocode:

```dart
final encrypted = await encryptionService.encrypt(plainText);
await file.writeAsBytes(encrypted);
```

This is not a concrete cryptographic implementation. Use a reviewed library and documented key-management design; never implement encryption by simply encoding or obfuscating text.

# Complete Example 1

This complete runnable Flutter UI demonstrates the correct architectural boundary without pretending to implement cryptography: it accepts plaintext, calls an injected `Vault` interface, and displays only operation status. A production implementation must use a vetted encryption library and secure key management.

```dart
import 'package:flutter/material.dart';

abstract interface class Vault {
  Future<void> store(String plaintext);
}

class DemoVault implements Vault {
  @override
  Future<void> store(String plaintext) async {
    // Deliberately no storage: replace with an audited implementation.
    if (plaintext.isEmpty) throw ArgumentError('Text cannot be empty.');
  }
}

void main() => runApp(const EncryptionBoundaryApp());

class EncryptionBoundaryApp extends StatelessWidget {
  const EncryptionBoundaryApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: EncryptionBoundaryPage());
}

class EncryptionBoundaryPage extends StatefulWidget {
  const EncryptionBoundaryPage({super.key});

  @override
  State<EncryptionBoundaryPage> createState() => _EncryptionBoundaryPageState();
}

class _EncryptionBoundaryPageState extends State<EncryptionBoundaryPage> {
  final _controller = TextEditingController();
  final Vault _vault = DemoVault();
  String _status = 'Nothing stored';

  Future<void> _save() async {
    try {
      await _vault.store(_controller.text);
      if (!mounted) return;
      setState(() => _status = 'Vault operation completed (demo only)');
    } catch (error) {
      if (!mounted) return;
      setState(() => _status = 'Vault operation failed');
    }
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Encrypted data boundary')),
        body: Padding(
          padding: const EdgeInsets.all(24),
          child: Column(
            children: [
              const Text('Demo only: this screen does not encrypt or persist data.'),
              TextField(
                controller: _controller,
                decoration: const InputDecoration(labelText: 'Sensitive text'),
              ),
              FilledButton(onPressed: _save, child: const Text('Store')),
              Text(_status),
            ],
          ),
        ),
      );
}
```

### Line-by-line explanation

- `Vault` defines the boundary between UI and the protected-data implementation.
- `DemoVault` intentionally performs no storage and explicitly rejects empty content; it must not be mistaken for encryption.
- `main` starts the demonstration app.
- The page owns the input controller, a vault implementation, and operation status.
- `_save` awaits the vault, then reports success-shaped completion only for the mock boundary; errors are shown as failure.
- `mounted` prevents late state updates.
- The UI explicitly warns that it does not encrypt or persist data.
- `dispose` releases the text controller.

### Widget tree

```text
MaterialApp → EncryptionBoundaryPage → Scaffold → Column
                                               ├→ warning
                                               ├→ TextField
                                               ├→ Store button
                                               └→ status
```

### Execution flow

```text
UI input → Vault interface → audited encrypted implementation (not included)
                         \→ demo adapter validates only; stores nothing
```

### ASCII phone preview

```text
+---------------------------+
| Encrypted data boundary   |
+---------------------------+
| Demo only: no encryption  |
| Sensitive text            |
| [                     ]   |
| [        Store        ]   |
| Nothing stored            |
+---------------------------+
```

### Example explanation

The key lesson is architectural: keep encryption behind an interface and never claim that a demo or Base64 encoding is encryption. The provided `DemoVault` is runnable precisely because it does not persist secrets.

If the warning is removed, a reader may mistake the sample for secure storage. If a real implementation catches an error and reports success anyway, users can lose sensitive data or believe it is protected when it is not.

**Common mistakes:** Never invent a cipher, reuse a nonce improperly, hard-code encryption keys, or store a key next to ciphertext without a threat-model rationale.

**Professional use case:** Protecting user-generated sensitive local data using a maintained encrypted database/file solution with a reviewed key lifecycle.

# Complete Example 2

For a file, use an authenticated encryption API from a maintained cryptography library:

```dart
final box = await secretBox.encrypt(
  plaintextBytes,
  secretKey: key,
  nonce: secureRandomNonce,
);
await file.writeAsBytes(box.concatenation());
```

Exact APIs vary by library. Store nonce/tag as required by that library and verify authentication during decryption.

# Complete Example 3

A database encryption setup must define key retrieval before opening the database:

```dart
final key = await keyStore.readDatabaseKey();
final database = await openEncryptedDatabase(key: key);
```

Both methods are illustrative interfaces, not drop-in package calls. Follow the encrypted database package's supported setup.

# Advanced Example

Specify threat model, key derivation/creation, storage, rotation, backup, recovery, and deletion. If users derive keys from passwords, use a vetted password-based key derivation function with salt and suitable work factors. Do not use a plain hash of the password as an encryption key.

# Under the Hood

Authenticated encryption protects both confidentiality and integrity. A nonce must satisfy the algorithm's uniqueness requirements for a key. Authentication failure should reject modified ciphertext. Keys are the critical secret and need a separate lifecycle.

# Performance Notes

- Encrypt large files in streaming/chunked form when supported.
- Avoid encrypting/decrypting on the UI isolate for expensive payloads.
- Cache decrypted data in memory only as long as necessary.
- Measure database open/decrypt latency.

# Best Practices

- Use maintained cryptography libraries and encrypted storage products.
- Protect keys separately.
- Prefer authenticated encryption.
- Document recovery and data-loss consequences.
- Keep plaintext out of logs and crash reports.
- Get security review for sensitive data designs.

# Common Mistakes

- Calling Base64 or obfuscation encryption.
- Hard-coding keys in source.
- Reusing nonces incorrectly.
- Ignoring authentication tags.
- Writing custom crypto.
- Assuming encryption protects a compromised running process.

# Debugging

- Distinguish wrong key, corrupted ciphertext, and unsupported format.
- Never log plaintext or key material.
- Test key loss and restore behavior in a test environment.
- Verify tampering causes decryption failure.
- Confirm backups do not undermine intended key separation.

# Interview Questions

1. What is authenticated encryption? Encryption that also verifies ciphertext integrity/authenticity.
2. Why is key management central? Anyone with the key can decrypt the data.
3. Is Base64 encryption? No; it is encoding.
4. What does a nonce do? It supplies per-encryption input required by many cryptographic schemes; uniqueness rules matter.
5. Does at-rest encryption protect a running compromised process? Not necessarily; plaintext and keys may be accessible in memory.

# Try It Yourself

1. Draw a data flow showing plaintext, encryption boundary, ciphertext, key store, and decryption.
2. Identify where keys and backup copies would exist.
3. Design an error state for wrong-key or tampered data.
4. Compare secure-storage and encrypted-database responsibilities.

# Challenge Project

Write a threat model and interface design for an encrypted offline journal. Specify library selection criteria, key lifecycle, backup, recovery, migration, and what happens when a user forgets a passphrase. Do not implement cryptography from scratch.

# Quick Revision Sheet

- Encryption protects data at rest only within a threat model.
- Use vetted authenticated encryption or encrypted database libraries.
- Protect keys separately; never hard-code them.
- Nonce/key lifecycle must follow library rules.
- Plan recovery, migration, and deletion.
- Never confuse encoding with encryption.
