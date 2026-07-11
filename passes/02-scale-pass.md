# The Scale Pass

Your app works great with one user: you. Then 50 people show up at once, the database locks, requests queue behind it, and everyone gets the blank screen. You do not need a rewrite to survive your first hundred users. You need three unglamorous things.

## 1. Connection pooling

The default AI-generated data layer opens a fresh database connection per request. Databases have a connection ceiling. Fifty simultaneous requests can hit it; when they do, the app does not slow down, it dies.

Pooling shares a small set of connections across all requests. Managed platforms (Supabase, Neon, RDS Proxy) have it built in or one toggle away. Self-hosting Postgres: PgBouncer. It is a config change, not a project.

**Work order:** "Show me how database connections are created per request. If there is no pool, add one, set sane limits against the database's max_connections, and show me the config. Then explain what happens at the limit: requests should wait briefly, not crash."

## 2. A caching layer

If the same data gets requested a hundred times a minute, do not compute it a hundred times a minute. Cache anything that does not change every second. A read-heavy endpoint going from 800ms to 50ms and database load dropping by most of its read traffic is a normal result, not a best case.

Three layers, stack them:

- **Memory or Redis at the API:** short TTL (30 to 60 seconds) on read-heavy endpoints. One middleware.
- **CDN at the edge:** static assets and any response that is identical for every user. The request never reaches your server at all.
- **Query-level cache:** that dashboard doing eight joins across four tables on every page load is the single most expensive thing you own. Cache the result, invalidate when the underlying data writes.

**Work order:** "List my ten highest-traffic read endpoints and the most expensive queries behind them. Add a short-TTL cache to the ones that can tolerate staleness, and write-invalidation to the ones that cannot. Show me before/after response times."

## 3. Load test before launch, not after

k6 and Artillery are free and take about half an hour. Simulate 100 users hitting your real flows: sign in, load the main list, open a record, trigger the expensive action. The bottleneck you find in a test costs you an afternoon. The one your users find costs you customers.

**Work order:** "Write a k6 script covering my core user flows at 100 concurrent users. Run it against staging. Report the p95 per endpoint and the first thing that breaks."

If it breaks in the test, it broke quietly. That was the whole point.
