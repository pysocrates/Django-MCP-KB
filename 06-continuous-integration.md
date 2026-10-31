---
feature_id: django.continuous_integration
summary: "Build a Django CI pipeline for checks, migrations, tests, and optional quality gates."
keywords: [ci, continuous integration, github actions, testing, linting, migrations]
tasks:
  - design a Django CI pipeline
  - check for missing migrations in CI
  - configure services for Django tests
related_docs: [00-index.md, 02-models-and-migrations.md, 03-security-and-settings.md, 04-testing.md, 05-deployment.md]
last_verified: "2026-10-30"
---

# Continuous integration for Django

## Scope

A CI pipeline checks changes automatically; deployment is a separate decision.
This guide adapts [Gopal Gautam's Django CI article](https://www.gopalgautam.dev/blog/typical-django-ci-setup)
for the Django 6.1 baseline. The pipeline design is general advice, not an
official Django-required configuration. Match it to the project's dependencies,
supported versions, infrastructure, and risk.

## Pipeline sequence

On pull requests and relevant branch pushes, a typical job:

1. Checks out the proposed code and selects a supported Python version.
2. Installs the project's declared, reproducible dependencies, including any
   chosen test and lint tools. Cache dependencies where useful.
3. Configures a dedicated test environment with non-production values and starts
   only the services the tests require. For example, test against PostgreSQL if
   the application depends on its behavior; add Redis or a worker only when needed.
4. Runs the team's selected lint and formatting checks, then Django's system
   checks and migration consistency check.
5. Runs unit and integration tests against isolated services. Add coverage,
   dependency auditing, static analysis, or build checks if they help the project.

With `manage.py` in the working directory and the project's test settings loaded,
a Django-native starting point is:

```sh
python manage.py check
python manage.py makemigrations --check
python -Wa manage.py test --noinput
```

`makemigrations --check` exits nonzero when model changes lack migrations and
already implies `--dry-run`; it does not prove existing migrations are safe.
`-Wa` displays Python warnings. `test --noinput` suppresses prompts; use it only
with isolated test databases because an interrupted test database may be removed.
Django creates separate test databases, but caches are not automatically isolated:
configure test cache and external service endpoints so tests cannot touch live data.

## Version and configuration choices

- Django 6.1 supports Python 3.12, 3.13, and 3.14; consult Django's compatibility
  table for the exact Django patch release and newer supported Python versions.
  The article's 3.10/3.11 matrix does not apply to Django 6.1.
- A project can use `DJANGO_SETTINGS_MODULE` or `--settings` to select test
  settings. Environment variables such as `DATABASE_URL` and `DB_HOST` only work
  if the project's settings actually read them; Django does not interpret them
  automatically. Use disposable CI-only credentials and no production secrets.
- Select a formatter, linter, or `pytest` integration based on project needs;
  they are not Django requirements. `pytest --ds` requires `pytest-django`.
  Coverage percentages supplement review, rather than proving test quality.
- For production readiness, run `check --deploy` separately against
  production-appropriate settings. The normal `check` in a CI test environment
  does not replace deployment review.
- Pin and maintain CI actions, packages, and service images using your team's
  dependency policy. Avoid copying an old workflow's version pins blindly.

## Sources

- [Typical Django CI Setup for Production-Grade Projects](https://www.gopalgautam.dev/blog/typical-django-ci-setup) — third-party pipeline overview and inspiration.
- [Django: Python compatibility](https://docs.djangoproject.com/en/6.1/faq/install/#what-python-version-can-i-use-with-django)
- [Django: management commands](https://docs.djangoproject.com/en/6.1/ref/django-admin/)
- [Django: writing and running tests](https://docs.djangoproject.com/en/6.1/topics/testing/overview/)
- [Django: deployment checklist](https://docs.djangoproject.com/en/6.1/howto/deployment/checklist/)
