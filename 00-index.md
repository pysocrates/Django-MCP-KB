---
feature_id: django.index
summary: "Index of Django project guidance based on official documentation."
keywords: [django, project, documentation, index]
tasks:
  - find guidance for building a Django project
related_docs:
  - 01-project-structure.md
  - 02-models-and-migrations.md
  - 03-security-and-settings.md
  - 04-testing.md
  - 05-deployment.md
  - 06-continuous-integration.md
  - 07-performance-and-query-efficiency.md
  - 08-celery-and-concurrency.md
last_verified: "2026-10-30"
---

# Django Knowledge Base

A starting point for building Django projects. These guides draw on Django's
official documentation and clearly identified additional sources; they do not describe an existing application or define
an MCP server implementation.

**Baseline:** Django 6.1 documentation, reviewed 2026-10-30. Check the
application's installed Django version and use the matching documentation before
applying guidance. Choose a supported release based on your project's needs.

## Guides

- [Project structure and development](01-project-structure.md) — create a
  project and apps, configure URLs, and work locally.
- [Models and migrations](02-models-and-migrations.md) — manage schema changes
  and inspect migration behavior.
- [Security and settings](03-security-and-settings.md) — handle secrets,
  authentication, request protections, and environment-specific settings.
- [Testing](04-testing.md) — test application behavior with Django's tools.
- [Deployment](05-deployment.md) — prepare settings, static assets, and a
  production server.
- [Continuous integration](06-continuous-integration.md) — check migrations,
  test behavior, and select project-appropriate CI checks.
- [Performance and query efficiency](07-performance-and-query-efficiency.md) —
  measure and prevent query growth, excessive memory use, and slow database work.
- [Celery workers and concurrency](08-celery-and-concurrency.md) — coordinate
  background tasks, database transactions, retries, and concurrent updates.

## Working with this knowledge base

Use the linked primary sources for detail and version-specific behavior. For a
real project, supplement this general guidance with documents that cite its own
code, contracts, and architectural decisions. Keep verified facts separate from
project preferences, and review documents when their sources or assumptions change.
