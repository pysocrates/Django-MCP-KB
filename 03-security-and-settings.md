---
feature_id: django.security_settings
summary: "Configure Django securely and understand its built-in protections."
keywords: [security, settings, csrf, xss, secret_key, authentication, authorization]
tasks:
  - configure production Django settings
  - review authentication and request security
related_docs: [00-index.md, 01-project-structure.md, 05-deployment.md]
last_verified: "2026-10-30"
---

# Security and settings

## Scope

Django supplies security features, but correct use and deployment remain the
application's responsibility. Review the official security guide and deployment
checklist for the application's Django version and environment.

## Configuration

- Keep `SECRET_KEY` secret and out of source control. Supply sensitive settings
  through an appropriate deployment configuration or secret manager.
- Set `DEBUG = False` in production and configure `ALLOWED_HOSTS` for the hosts
  that should serve the site.
- Use HTTPS throughout authenticated sites; review secure session and CSRF
  cookies and proxy settings against the actual deployment topology.
- Review installed middleware and settings deliberately; turning off built-in
  protections requires understanding what replaces them.

## Application behavior

- Use Django's authentication and permission facilities where appropriate;
  check authorization for each operation and object, not only authentication.
- Retain CSRF protection for applicable browser requests; understand the
  conditions before exempting any endpoint.
- Template escaping helps protect against XSS, but unsafe HTML, user-controlled
  URLs, uploads, and other contexts require care.
- Prefer ORM query construction and parameterized queries over interpolating
  user input into raw SQL. Treat uploaded files as untrusted.

## Official sources

- [Security in Django](https://docs.djangoproject.com/en/6.1/topics/security/)
- [Deployment checklist](https://docs.djangoproject.com/en/6.1/howto/deployment/checklist/)
- [Authentication and permissions](https://docs.djangoproject.com/en/6.1/topics/auth/)
- [CSRF protection](https://docs.djangoproject.com/en/6.1/ref/csrf/)
- [Settings reference](https://docs.djangoproject.com/en/6.1/ref/settings/)
