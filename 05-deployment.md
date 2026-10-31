---
feature_id: django.deployment
summary: "Prepare a Django application for production deployment."
keywords: [deployment, production, asgi, wsgi, static, check, logging]
tasks:
  - review Django production deployment
  - run deployment checks
related_docs: [00-index.md, 03-security-and-settings.md, 04-testing.md]
last_verified: "2026-10-30"
---

# Deployment

## Scope

This guide is a starting point for deployment planning, not a complete runbook.
Infrastructure, runtime, database, traffic, and security requirements vary.

## Before deployment

1. Review Django's deployment checklist and the project's production settings.
2. Run `python manage.py check --deploy` using production-appropriate settings;
   investigate its output. Passing checks is not a complete security review.
3. Use a production-capable WSGI or ASGI server appropriate to the application;
   Django's `runserver` is not a production server.
4. Plan database migrations, backups, rollback, and health monitoring for the
   actual environment.
5. Configure static and user-uploaded media handling. Run `collectstatic` as
   required by the chosen static-file deployment approach and protect uploads.
6. Configure application logging and error reporting, then verify the deployed
   behavior and operational access.

## Official sources

- [Deployment overview](https://docs.djangoproject.com/en/6.1/howto/deployment/)
- [Deployment checklist](https://docs.djangoproject.com/en/6.1/howto/deployment/checklist/)
- [Deploying static files](https://docs.djangoproject.com/en/6.1/howto/static-files/deployment/)
- [Logging](https://docs.djangoproject.com/en/6.1/topics/logging/)
