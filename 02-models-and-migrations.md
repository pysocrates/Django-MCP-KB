---
feature_id: django.models_migrations
summary: "Develop Django models and manage database schema migrations."
keywords: [models, database, orm, migrations, makemigrations, migrate]
tasks:
  - change a Django model and generate a migration
  - inspect pending database migrations
related_docs: [00-index.md, 01-project-structure.md, 04-testing.md]
last_verified: "2026-10-30"
---

# Models and migrations

## Scope

Django models describe application data; migrations record changes to models and
other schema state. The right database, constraints, and operational plan depend
on the application and database backend.

## Workflow

1. Change the model and add or update tests for the intended behavior.
2. Run `python manage.py makemigrations` and inspect the generated migration.
3. Use `python manage.py sqlmigrate app_label migration_name` when you need to
   inspect generated SQL for a migration and backend.
4. Run `python manage.py migrate` against the intended database and run tests.
5. Commit migration files with the corresponding model changes so other
   environments can reproduce the schema history.

Use `python manage.py showmigrations` to inspect migration status and
`python manage.py makemigrations --check` to detect model changes without
migration files. Review generated operations and plan rollout for large tables,
data migrations, locking, reversibility, and backups; a generated migration is
not automatically safe for every production database.

## Rules and edge cases

- Database constraints are useful for enforcing data integrity, including when
  multiple processes write concurrently; select constraints for the actual rules.
- For data migrations use Django's historical model guidance: model definitions
  evolve, so importing current models can break older migrations.
- SQLite is convenient for local development, but test behavior and deployment
  plans against the database backend the project actually uses.

## Official sources

- [Models](https://docs.djangoproject.com/en/6.1/topics/db/models/)
- [Migrations and historical models](https://docs.djangoproject.com/en/6.1/topics/migrations/)
- [Constraints reference](https://docs.djangoproject.com/en/6.1/ref/models/constraints/)
- [Database backends](https://docs.djangoproject.com/en/6.1/ref/databases/)
