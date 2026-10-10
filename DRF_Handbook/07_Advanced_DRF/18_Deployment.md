# Introduction

Deployment prepares a DRF application to run safely and reliably in a production environment.

# Why this concept exists

Development defaults, local files, and debug settings are not designed for public traffic, multiple workers, secrets, or failure recovery.

# Real-world analogy

Deployment is opening a factory: equipment, safety systems, trained operators, monitoring, and emergency procedures must all be ready.

# Internal Working

Production traffic typically flows through DNS/TLS termination, reverse proxy, application server workers, Django middleware/DRF, database/cache/storage, and monitoring systems.

# Authentication Flow (where applicable)

TLS edge -> proxy headers -> Django security middleware -> authenticator -> permissions -> view/data services -> safe response.

# Syntax

```python
DEBUG = False
ALLOWED_HOSTS = ["api.example.com"]
```

# Complete Example 1

```python
# settings/production.py
import os

DEBUG = False
SECRET_KEY = os.environ["DJANGO_SECRET_KEY"]
ALLOWED_HOSTS = ["api.example.com"]
CSRF_COOKIE_SECURE = True
SESSION_COOKIE_SECURE = True
SECURE_SSL_REDIRECT = True
```

Production settings disable debug, load secret from environment/secret manager, restrict hosts, secure cookies, and redirect HTTP. Configure HSTS and proxy SSL headers only after confirming the trusted proxy topology. Do not commit environment secrets.

# Complete Example 2

```text
Reverse proxy -> Gunicorn/Uvicorn workers -> Django app
              -> PostgreSQL / Redis / object storage
              -> centralized logs, metrics, and traces
```

Each component has a defined role: TLS and request limits at edge, WSGI/ASGI server runs Django, database persists data, cache supports shared state, object storage holds uploads, and observability supports operations. Use managed secrets and health checks.

# Complete Example 3

```shell
python manage.py check --deploy --settings=config.settings.production
python manage.py migrate --settings=config.settings.production
```

The deployment check reports common security configuration issues; migrations apply schema changes. Run migrations as a controlled release step with backups and compatible rollout planning. This command sequence is illustrative; environment and release process vary.

# Advanced Example

Use immutable artifacts, staged rollout, readiness/liveness probes, graceful worker shutdown, database migration compatibility, rollback plans, backups, and incident runbooks. Avoid destructive schema changes in the same release that removes old code dependencies.

# Under the Hood

WSGI/ASGI servers create worker processes; static/media handling is typically delegated to a proxy or storage/CDN. Proxy headers affect scheme/client IP and must be trusted only from known infrastructure.

# Performance Notes

Tune worker count, DB connection limits, timeouts, body-size limits, and concurrency from load tests and resource measurements.

# Security Notes

Set `DEBUG=False`, secret rotation, allowed hosts, HTTPS, secure cookies, CORS/CSRF policy, least-privilege DB/storage credentials, patching, and backup protection.

# Best Practices

- Separate settings/secrets from source.
- Run deploy checks and migrations safely.
- Centralize logs/metrics and monitor health.
- Use staged deployments and rollback procedures.

# Common Mistakes

- Running `runserver` in production.
- Exposing debug pages or permissive hosts.
- Trusting forwarded headers from arbitrary clients.

# Debugging

Check deployment config, proxy headers, app logs, health endpoint, DB connectivity, migrations, static/media storage, and secret injection.

# Interview Questions

1. Why not use Django runserver in production?
2. What does `check --deploy` do?
3. Why do migrations need rollout planning?

# Practice Exercises

1. Create production settings with secrets externalized.
2. Diagram request flow through proxy and app server.
3. Write a rollback/backup checklist.

# Chapter Summary

Production deployment is a system of coordinated components and release practices. Secure configuration, trusted proxy boundaries, observability, migrations, and recovery plans are as important as application code.
