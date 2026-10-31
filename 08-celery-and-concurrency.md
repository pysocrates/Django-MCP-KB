---
feature_id: django.celery_concurrency
summary: "Design Django Celery workers for safe transactions, retries, and concurrent execution."
keywords: [celery, workers, concurrency, race conditions, idempotency, transactions, retries, locking, on_commit]
tasks:
  - enqueue Celery work after a Django transaction commits
  - prevent duplicate task effects and concurrent updates
  - test worker retries and database locking
related_docs: [00-index.md, 02-models-and-migrations.md, 04-testing.md, 05-deployment.md, 07-performance-and-query-efficiency.md]
last_verified: "2026-10-30"
---

# Celery workers and concurrency

## Scope

Guidance for Django 6.1 and Celery 5.6; Celery is an optional third-party task
queue, not a Django built-in. Verify compatibility with the versions and broker
your application uses. Workers, web requests, scheduled tasks, and retries can
all operate on the same data concurrently. Identify the business invariant and
failure modes before choosing a concurrency mechanism.

## Integrate and operate workers

- Follow Celery's Django integration guide to configure a Celery app, load
  `CELERY_` settings, and discover task modules. Run a broker and worker as
  separate processes; a task called with `.delay()` publishes a message.
- Pass stable identifiers and small, serializable arguments instead of model
  instances or secrets. Reload current data in the worker and handle records
  that were changed or deleted before the task starts.
- Set appropriate network timeouts, bounded retries with backoff, and monitoring
  for failures and queue latency. Size concurrency and prefetch for the workload
  and database connection limits. Avoid unbounded work or long-held locks.
- For periodic work, run one scheduler for a given schedule; separate invocations
  can still overlap if a task takes longer than its interval. Design overlap
  handling based on the business operation, including failure recovery.

## Publish after a database commit

A worker may execute before a transaction that created its input has committed.
Register publication after the relevant database transaction commits:

```python
from django.db import transaction

with transaction.atomic():
    order = Order.objects.create(...)  # Illustrative project model
    transaction.on_commit(lambda order_id=order.pk: process_order.delay(order_id))
```

Callbacks registered in rolled-back transactions do not run; outside an active
transaction, `on_commit()` runs immediately. Use the correct database alias with
`transaction.atomic(using=...)` and `transaction.on_commit(..., using=...)` in
multi-database applications. Celery's `DjangoTask.delay_on_commit()` is a
convenience for the default connection and does not return a task ID at call
time; verify your task uses the appropriate task class and Celery version.

**Delivery gap:** committing the database and publishing to the broker are not
one atomic operation. A process crash or broker failure after commit can leave
work unpublished. Where eventual delivery matters, use a transactional outbox
or equivalent durable record written in the same database transaction, with a
separate retrying publisher and idempotent consumers. `on_commit()` alone is not
an exactly-once delivery guarantee.

## Prevent duplicate effects and lost updates

- Make tasks safe to run more than once. Use a stable business operation key,
  database uniqueness constraints, and atomic state transitions as appropriate.
  A task ID alone is not necessarily a business-level deduplication key. For
  external APIs, use a supported provider idempotency key and reconcile uncertain
  outcomes; a local database lock cannot atomically commit a remote side effect.
- For a simple counter, use an atomic database-side update rather than reading,
  incrementing in Python, and saving a stale value:

  ```python
  from django.db.models import F

  Account.objects.filter(pk=account_id).update(attempts=F("attempts") + 1)
  ```

  This avoids a lost-update race for that counter, but it does not make a
  repeated task idempotent. `update()` bypasses model `save()` and its signals.
- For a state transition, use a conditional update such as
  `Job.objects.filter(pk=job_id, status="pending").update(status="processing")`
  and inspect the affected-row count. One worker can claim the transition; design
  a recovery policy for a worker that dies after claiming it.
- When a multi-step database decision requires a row lock, evaluate
  `select_for_update()` inside `transaction.atomic()` on a supported backend,
  keep the transaction short, use a consistent lock order, and handle contention
  or deadlocks. SQLite does not enforce `SELECT FOR UPDATE`; backend support and
  options differ. A row lock does not protect the absence of a row: use database
  constraints for uniqueness.

## Retries and delivery semantics

Celery normally acknowledges tasks before execution. `acks_late` changes the
acknowledgement timing for idempotent tasks and can result in redelivery; worker
termination and broker behavior have important exceptions. Review Celery's
acknowledgement, worker-loss, retry, and broker settings rather than assuming
at-most-once or exactly-once behavior. For Redis brokers, visibility timeouts can
also cause redelivery of long-running or delayed tasks. Retrying a task and
redelivering an unacknowledged message are distinct mechanisms.

## Test the failure paths

Use the production database backend for locking and concurrency tests. Django's
`TransactionTestCase` is suitable for testing real transaction boundaries;
`TestCase` wraps tests in transactions and can mask locking mistakes. Exercise
overlapping claims with separate connections or processes, duplicate execution,
rollback before enqueue, retry after partial work, and a worker failure after
claiming work. Test remote effects with a controlled fake or sandbox and verify
idempotency. Eager task execution alone does not exercise broker delivery or
worker concurrency; include integration tests with a real broker and worker where
those guarantees matter.

## Official sources

- [Celery: Django integration and commit hooks](https://docs.celeryq.dev/en/stable/django/first-steps-with-django.html)
- [Celery: tasks, acknowledgement, and retries](https://docs.celeryq.dev/en/stable/userguide/tasks.html)
- [Celery: periodic tasks](https://docs.celeryq.dev/en/stable/userguide/periodic-tasks.html)
- [Celery: Redis broker visibility timeout](https://docs.celeryq.dev/en/stable/getting-started/backends-and-brokers/redis.html)
- [Django: transactions and on_commit](https://docs.djangoproject.com/en/6.1/topics/db/transactions/)
- [Django: select_for_update](https://docs.djangoproject.com/en/6.1/ref/models/querysets/#select-for-update)
- [Django: F expressions and race conditions](https://docs.djangoproject.com/en/6.1/ref/models/expressions/#avoiding-race-conditions-using-f)
