---
id: backend-python.iterating-result-sets
concept: Iterating over a database result set
role: Backend engineer
language: Python
prerequisites: []
taxonomy:
  - name: Object-Relational Mapping (ORM)
    source: lightcast
    id: TBD
  - name: Query Optimization
    source: lightcast
    id: TBD
rungs: [minimal, idiomatic, excerpt, in_situ, failure]
verified: 2026-08-31
---

# Iterating over a database result set

This is the concept behind the question "what is a for loop, really." The loop
itself takes ten minutes to learn. What takes years is knowing what the loop is
standing on, because in a backend service the thing you are looping over is
usually not in memory yet.

## Rung 1 — minimal

```python
# rung: minimal
orders = [{"id": 1, "total": 40}, {"id": 2, "total": 12}]
for order in orders:
    print(order["total"])
```

The list is already in memory. Every iteration costs nothing but the loop body.
Hold on to that assumption, because the next four rungs are about it being false.

**Check:** how many times does the program touch the disk or the network? Zero.
Write down why.

## Rung 2 — idiomatic

Same loop shape, different thing underneath.

```python
# rung: idiomatic
for order in Order.objects.all():
    print(order.total)
```

This reads identically to rung 1 and behaves nothing like it. `Order.objects.all()`
is lazy: no query runs until the loop asks for the first item. Then Django runs
one `SELECT`, builds a model instance per row, and caches the whole result on the
queryset.

The naive version people write before they know that:

```python
# rung: idiomatic, variant: naive
orders = list(Order.objects.all())   # forces the query here
for order in orders:
    print(order.total)
```

Functionally the same. The explicit `list()` is what an experienced reviewer
flags, because it announces that the author expects the whole table to fit in
memory.

**Check:** at which line does the SQL actually execute in each version?

## Rung 3 — excerpt

Real shape of a nightly job, with logging, metrics, error handling and the
business rule stripped out. What remains is only the iteration strategy.

```python
# rung: excerpt
# Removed: structured logging, Prometheus counters, the retry decorator,
# and the actual settlement logic. Only the iteration survives.

def settle_pending_orders():
    qs = (
        Order.objects
        .filter(status="pending")
        .select_related("customer")          # join, do not re-query
        .iterator(chunk_size=2000)           # stream, do not cache
    )
    for order in qs:
        settle(order)
```

Two decisions carry this function, and neither is visible in the loop.

`select_related("customer")` turns what would be one query per order into a
single join. `iterator(chunk_size=2000)` tells Django to stream rows instead of
caching the entire result set on the queryset. On PostgreSQL this is backed by a
server-side cursor.

**Check:** remove `select_related` and predict the query count for 50,000 orders.
Then remove `iterator` and predict peak memory.

## Rung 4 — in situ

The behaviour lives in Django itself, in `django/db/models/query.py`. Read
`QuerySet.iterator`, then `QuerySet._iterator`, then follow into
`prefetch_related_objects`. Line numbers move between releases; the symbol names
have been stable.

What to look for while reading:

- `iterator()` does not populate `self._result_cache`. That single omission is
  the whole memory story.
- `_iterator` branches on whether prefetch lookups are present. With none, it
  yields straight through. With prefetch lookups, it accumulates a chunk, runs
  `prefetch_related_objects` on that chunk, then yields the chunk.
- That branch is why `chunk_size` became mandatory when combining `iterator()`
  with `prefetch_related()`. There is no sensible batch size to guess.

The history is worth reading alongside the code: chunk_size arrived in Django
ticket #27639, and prefetch support in ticket #29984. Both tickets are arguments
about exactly the tradeoff in this lesson.

**Check:** find the line where `_result_cache` would have been set in the
non-iterator path, and confirm it is absent in the iterator path.

## Rung 5 — failure

Three ways this breaks in production, in the order teams usually hit them.

**N+1 queries.** The loop looks like one operation and is actually 50,001. Each
`order.customer` on an unfetched relation issues its own `SELECT`. It passes
review, passes tests against a 20-row fixture, and falls over the first time it
meets real data. `select_related` for forward foreign keys, `prefetch_related`
for reverse and many-to-many.

**Memory.** Without `iterator()`, the queryset holds every model instance for the
life of the loop. A million rows of a wide model is measured in gigabytes, and
the process dies somewhere in the middle of the job with no useful traceback.

**The two fixes fight each other.** `iterator()` exists to avoid holding results;
`prefetch_related` exists by holding results. Before Django 4.1 they were simply
incompatible. They now compose, but only when you pass an explicit `chunk_size`,
and the prefetch runs once per chunk rather than once per job. Smaller chunks
mean less memory and more round trips. There is no default that is right for
every table, which is why Django refuses to pick one for you.

**A caveat that is not Django's.** Server-side cursors carry driver-level rules.
In SQLAlchemy, the equivalent is the `yield_per` execution option, and the
documentation is explicit that calling `Result.all()` defeats it, that psycopg2
raises on a server-side cursor with DML or DDL, and that MySQL drivers leave the
connection in a fragile state that will not roll back until the cursor closes.
The same concept, the same tradeoff, different sharp edges per driver.

**Check:** you have a job that must email every customer with a pending order,
where the message includes their last five orders. Write the queryset. Justify
your `chunk_size`. State what happens if the job crashes at row 400,000.

## What transfers

Nothing above is about Python. The pattern — a loop whose real cost is hidden in
what it iterates, and where the two obvious fixes trade against each other — is
the same in Go with `sql.Rows`, in Java with a JDBC fetch size, and in Rails with
`find_each`. Learn it once at this depth and the next language is a syntax
lookup.

## Sources

- Django ticket #27639, chunk_size argument for QuerySet.iterator():
  https://code.djangoproject.com/ticket/27639
- Django ticket #29984, prefetch_related() support in QuerySet.iterator():
  https://code.djangoproject.com/ticket/29984
- Django source, django.db.models.query:
  https://docs.djangoproject.com/en/5.0/_modules/django/db/models/query/
- SQLAlchemy 2.0, yield_per and server-side cursor caveats:
  https://docs.sqlalchemy.org/en/20/core/connections.html
