# Verification

This guide describes checked-in tests and reproduction commands. It does not claim a fresh passing run, measured throughput or production readiness.

## Backend

From `razorpay-agentic-commerce/`, with Compose services running:

```bash
docker compose exec backend pytest
```

The [flow tests](../razorpay-agentic-commerce/backend/tests/test_flow.py) cover authentication rejection, database-derived pricing, mock order creation, checkout creation retries, duplicate completion, transaction-limit rejection, suspended agents, simulated timeout recovery and catalog-result constraints.

Tests that use mocks or SQLite do not establish live Razorpay behavior or PostgreSQL concurrency correctness. Keep the database mode and provider mode with each recorded result.

## Frontend

From `razorpay-agentic-commerce/frontend/`:

```bash
npm install
npx tsc --noEmit
node --test tests/transaction-progress.test.cjs
npm run build
```

The [progress regression test](../razorpay-agentic-commerce/frontend/tests/transaction-progress.test.cjs) exercises the shared TypeScript progress module using Node's test runner. Type checking and a build are distinct from running these regressions. Google Fonts access may be required during the build.

## CI and releases

At this documentation review, the repository has no checked-in GitHub Actions workflow, returned workflow runs or published GitHub release. Do not display a passing-CI or stable-release badge until those exist.

## Evidence for stronger claims

Before reporting performance, retain the commit, machine/container limits, database/provider mode, dataset, workload, warmup, measurement duration, repetitions, latency percentiles, error categories and raw results. No benchmark number is asserted by this guide.

Concurrent completion, inventory allocation, daily-budget races, duplicate webhook races, crash recovery and captured amount/currency matching need dedicated verification before related guarantees are advertised.
