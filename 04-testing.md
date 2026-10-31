---
feature_id: django.testing
summary: "Test Django behavior using its test runner and testing utilities."
keywords: [testing, unittest, test client, database, fixtures]
tasks:
  - write and run Django tests
  - test a view or database behavior
related_docs: [00-index.md, 02-models-and-migrations.md, 03-security-and-settings.md, 06-continuous-integration.md]
last_verified: "2026-10-30"
---

# Testing Django applications

## Scope

Choose tests around the behavior and risks of the project. Django integrates
with Python's `unittest` and provides utilities for request handling and database
isolation; other test frameworks can also integrate with Django.

## Workflow

- Run the project suite with `python manage.py test`; pass an app or test label
  to focus a run when appropriate.
- Use Django's test client to exercise views, responses, sessions, and relevant
  permissions without running a web server.
- Use suitable Django test case classes for database behavior and isolation;
  use transaction-aware tests where real transaction behavior matters.
- Cover success, invalid input, authorization failures, and important edge cases
  in proportion to the feature's risk. Avoid asserting only implementation details.
- Check behavior against the production database backend where backend-specific
  SQL, constraints, or transactions are important.

The test runner creates and uses test databases; read the testing documentation
before changing test database settings or using an existing database.

For automated pipelines, see [Continuous integration](06-continuous-integration.md).

## Official sources

- [Testing overview](https://docs.djangoproject.com/en/6.1/topics/testing/)
- [Writing and running tests](https://docs.djangoproject.com/en/6.1/topics/testing/overview/)
- [Testing tools and client](https://docs.djangoproject.com/en/6.1/topics/testing/tools/)
- [Advanced testing topics](https://docs.djangoproject.com/en/6.1/topics/testing/advanced/)
