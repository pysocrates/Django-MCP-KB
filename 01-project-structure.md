---
feature_id: django.project_structure
summary: "Create and organize a Django project and its applications."
keywords: [django, startproject, startapp, urls, settings, manage.py]
tasks:
  - create a Django project
  - organize Django apps and URL routes
related_docs: [00-index.md, 02-models-and-migrations.md, 03-security-and-settings.md]
last_verified: "2026-10-30"
---

# Project structure and development

## Scope

General Django 6.1 project setup, not a prescribed architecture for every team.
Confirm the supported Python version and dependencies for the Django release you
choose. The official tutorial distinguishes a **project** (configuration for a
Django installation) from an **app** (a Python package providing functionality).

## Create and run locally

Use an isolated Python environment and install the chosen supported Django
release. From the desired parent directory:

```sh
python -m django --version
mkdir myproject
django-admin startproject config myproject
cd myproject
python manage.py migrate
python manage.py startapp example
python manage.py runserver
```

`startproject` creates the project package, settings, URL configuration, ASGI
and WSGI entry points, and `manage.py`. `startapp` creates an application
package. Register the app in `INSTALLED_APPS` when Django needs to discover it.
The development server is for development, not production.

## Organize and route

- Group related functionality in apps and choose boundaries that reflect the
  project; Django does not require a universal service-layer layout.
- Map URL patterns to views, and use `include()` to compose app URLconfs when
  useful. Consider namespaced URL names to avoid collisions.
- Use Django's configuration and system checks. Do not copy an example setting
  into production without reviewing its implications.
- Keep project-specific decisions and commands in the project's own documentation.

## Official sources

- [Tutorial: project and app setup](https://docs.djangoproject.com/en/6.1/intro/tutorial01/)
- [Tutorial: models, database, and installed apps](https://docs.djangoproject.com/en/6.1/intro/tutorial02/)
- [URL dispatcher](https://docs.djangoproject.com/en/6.1/topics/http/urls/)
- [Applications](https://docs.djangoproject.com/en/6.1/ref/applications/)
- [Supported Python versions](https://docs.djangoproject.com/en/6.1/faq/install/#what-python-version-can-i-use-with-django)
