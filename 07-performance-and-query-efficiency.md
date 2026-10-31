---
feature_id: django.performance_query_efficiency
summary: "Measure and limit Django query growth, memory use, and database work using built-in tools."
keywords: [performance, time complexity, N+1, querysets, select_related, prefetch_related, pagination, indexes, caching]
tasks:
  - prevent N+1 database queries
  - investigate slow Django views and query growth
  - choose efficient ORM operations and pagination
related_docs: [00-index.md, 02-models-and-migrations.md, 04-testing.md, 05-deployment.md]
last_verified: "2026-10-30"
---

# Performance and query efficiency

## Scope and measurement

Use Django's built-in ORM and database features to keep work proportional to
what a request needs. Query count, rows scanned, network round trips, Python CPU,
memory, and response size are different costs; reducing one does not guarantee a
faster request. There is no universal Big-O guarantee for an ORM call: plans,
indexes, data distribution, and backend behavior matter.

Profile a representative workload first. Record query counts, latency, and data
sizes, then inspect expensive query plans with `QuerySet.explain()` on the actual
database backend. Re-measure after each change. Avoid running plan options that
execute a query on production without understanding their effects.

## Avoid query growth in loops

Accessing a related object for each result can cause an N+1 pattern: one query
for the collection and potentially another for each row. If every displayed
order needs its customer and items, fetch those relationships deliberately:

```python
orders = (
    Order.objects.filter(status="open")
    .select_related("customer")
    .prefetch_related("items")
)
for order in orders:
    print(order.customer.name, [item.name for item in order.items.all()])
```

This example assumes `customer` is a foreign key and `items` is a relation;
adapt names and access patterns to the actual models. `select_related()` joins
single-valued relationships; `prefetch_related()` uses additional queries and
associates results in Python, including for many-valued relationships. Specify
only relationships you need: prefetching large collections increases memory and
large `IN` clauses can be expensive. A later different related-manager filter
may issue another query instead of using the prefetch. Django 6.1 deprecates
calling `select_related()` without field names.

## Choose the work deliberately

- QuerySets are lazy, but iteration, `list()`, `len()`, boolean evaluation, and
  other operations may evaluate them. Avoid accidental repeated evaluation and
  inspect the generated queries.
- Use `exists()` or `count()` when existence or a count is all you need. If you
  will fetch the same results immediately afterward, a separate check can add a
  query; use the queryset's result cache where appropriate.
- Filter and aggregate in the database when appropriate, using `filter()`,
  `annotate()`, `aggregate()`, and `F()` expressions rather than fetching every
  row for Python processing. Check SQL and correctness, especially across joins.
- Fetch only required fields using `values()` or `values_list()` when model
  instances aren't needed. Use `only()` and `defer()` cautiously: later access to
  deferred fields can cause extra queries, and reducing columns may not improve
  performance. Measure.
- For large one-pass reads consider `iterator(chunk_size=...)` to avoid the
  QuerySet result cache. Driver and backend behavior affects actual streaming;
  with `prefetch_related()`, supply a chunk size and measure query/memory tradeoffs.
- Prefer bulk operations when their semantics match the task. `update()`,
  `bulk_create()`, and `bulk_update()` have documented caveats around model
  methods, signals, batching, and related objects. Do not assume they behave like
  calling `save()` for every instance.

## Bound result size and plan for scale

Use `Paginator` or suitable bounded queries to avoid loading and returning an
unbounded collection. Django's paginator commonly counts rows, and deep
`OFFSET` pages may be expensive; for large or rapidly changing datasets consider
keyset pagination with a stable, indexed ordering appropriate to the application.
Always measure with realistic data.

Add database indexes for demonstrated filter, join, and ordering patterns; use
`explain()` and database tools to verify they are used. Indexes consume storage
and add write cost, so avoid adding them blindly. Caching can reduce repeated
work, but requires an invalidation strategy, appropriate tenant/user separation,
and a suitable backend; it does not correct an inefficient cold path.

## Regression checks

Test representative code paths with Django's `assertNumQueries()` where a stable
query budget is meaningful, and compare small and larger input sizes to detect
N+1 regressions. Evaluate the queryset and access the related fields inside the
assertion. Query counts alone do not reveal slow scans or large result sets;
combine them with query-plan inspection and realistic performance measurements.

## Official sources

- [Database access optimization](https://docs.djangoproject.com/en/6.1/topics/db/optimization/)
- [QuerySet API](https://docs.djangoproject.com/en/6.1/ref/models/querysets/)
- [Database indexes](https://docs.djangoproject.com/en/6.1/ref/models/indexes/)
- [Pagination](https://docs.djangoproject.com/en/6.1/topics/pagination/)
- [Django cache framework](https://docs.djangoproject.com/en/6.1/topics/cache/)
- [Testing tools: query assertions](https://docs.djangoproject.com/en/6.1/topics/testing/tools/#django.test.TransactionTestCase.assertNumQueries)
