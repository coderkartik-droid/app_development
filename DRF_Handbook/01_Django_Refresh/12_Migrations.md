# Introduction

Django migrations are versioned Python operations that evolve database schema and, when needed, data. They let a team apply model changes consistently across development and deployment environments.

# Why this concept exists

Editing a model class does not alter an existing database table. Migrations provide a repeatable history for adding, changing, and removing schema objects.

# Real-world analogy

A migration is a numbered renovation instruction that tells every building how to move from its current layout to the next without losing important contents.

# Internal Working

`makemigrations` compares current model state with migration history and writes operations. `migrate` applies unapplied operations and records them in Django's migration table.

```text
model change -> makemigrations -> migration file -> migrate -> DB schema updated
```

# Syntax

```bash
python manage.py makemigrations
python manage.py migrate
python manage.py showmigrations
```

# Complete Example 1

Add a field to an existing model and apply the change:

```python
# catalog/models.py
class Book(models.Model):
    title = models.CharField(max_length=200)
    is_available = models.BooleanField(default=True)
```

```bash
python manage.py makemigrations catalog
python manage.py sqlmigrate catalog 0002
python manage.py migrate
```

**Line-by-line:** Add a Boolean field with a default for existing/new rows. `makemigrations` generates an operation; inspect `sqlmigrate` output for the SQL Django plans; `migrate` applies it. **Flow:** model state -> migration graph -> database operation -> migration history row. **Internal:** Django orders migrations by dependency graph, not merely filename. **Professional tip:** Review generated migrations before deployment and test against a copy of representative old data.

# Complete Example 2

Inspect applied migrations:

```bash
python manage.py showmigrations
```

The command displays migrations and whether each is applied. It reads the configured database's migration records. **Flow:** settings select database -> migration recorder is queried -> status list is printed. **Tip:** Confirm the environment/database before applying operations; development and production settings may point to different data.

# Advanced Example

Use a `RunPython` migration for data transformation when schema changes require moving existing values. Provide a reverse operation when possible, use historical models from the migration app registry, and batch large tables appropriately.

# Common Mistakes

- Forgetting `makemigrations` or `migrate`.
- Deleting migration history in a shared project.
- Editing an already-applied migration casually.
- Making destructive schema changes without backup/rollout planning.

# Best Practices

- Commit migration files with the model changes.
- Test fresh database setup and upgrades from supported versions.
- Review SQL and preserve data deliberately.
- Keep migrations deterministic and environment-independent.

# Interview Questions

1. Difference between `makemigrations` and `migrate`? Generate operations vs apply them.
2. What tracks applied migrations? Django's migration recorder table.
3. Why inspect `sqlmigrate`? Understand SQL effects before applying.
4. What is a data migration? A versioned transformation of stored records.

# Practice

Add an optional field, inspect its generated migration, apply it, and verify with `showmigrations`.

# Summary

Migrations bridge Python model state and persistent schema. Generate, inspect, apply, and test them; never assume a model edit updates existing databases automatically.
