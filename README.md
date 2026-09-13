# Tekk skills

Two things live here, and most people want the first.

1. **The Tekk plugin** — 4 skills for driving your [Tekk](https://tekk.coach) board from your coding agent, bundled with the Tekk MCP server.
2. **The review methods** — **42 [Agent Skills](https://agentskills.io) across 12 domains**, from tenant isolation to webhook fulfilment to what an agent costs to run. These are what Tekk's autonomous loops apply to a codebase. They need no account.

> **Generated — do not edit here.** Every file is built from the Tekk engine and pushed on each release. A fix belongs upstream; an edit made here is overwritten by the next push. Issues and discussion are welcome.

## Drive Tekk from your agent

Installs `sweep`, `blitz`, `triage`, `spec` and connects the board in one step:

```
/plugin marketplace add guccikudo92615/tekk-skills
/plugin install tekk@tekk
```

You need a Tekk account for these — they act on your board. The MCP server authenticates in the browser; nothing is pasted and no key is stored.

For Codex and other agents, take the skills on their own and configure the MCP server separately:

```bash
npx skills add guccikudo92615/tekk-skills --skill sweep --skill blitz --skill triage --skill spec -a codex
```

## Install the review methods

No account needed. Everything, into Claude Code:

```bash
npx skills add guccikudo92615/tekk-skills
```

One domain, or one method:

```bash
npx skills add guccikudo92615/tekk-skills --list
npx skills add guccikudo92615/tekk-skills --skill tenant-isolation --skill retry-safety
```

It works the same for Codex, Cursor and 70-odd other agents — `-a codex`, `-a cursor`. In Claude Code you can instead install a whole domain as a plugin, which keeps the skills namespaced and out of your always-on context until you use one:

```
/plugin marketplace add guccikudo92615/tekk-skills
/plugin install tekk-security@tekk
```

## What is here

Each domain is a **shelf**: a `loop.md` that says what the domain is and what counts as a finding in it, plus the methods. They are written to be opened one at a time — a single method is a whole review's worth of instruction, not a checklist item.

| Domain | Methods | What it owns |
| --- | --- | --- |
| [`security`](skills/security/loop.md) | `auth-and-access` · `tenant-isolation` · `money-and-webhooks` · `secrets-and-crypto` · `injection-and-input` | Audits recent code changes for security holes — auth gaps, exposed data, injection, tenant leaks — and proposes fixes. |
| [`reliability`](skills/reliability/loop.md) | `dependency-boundary` · `retry-safety` · `resource-limits` · `failure-handling` | Audits your code for the ways it breaks under real-world failure — a dependency that times out with no deadline, a retry that runs a charge twice, a swallowed error that looks like success, an unbounded operation that exhausts memory — and proposes the fix. Sharper when Sentry is connected, but works without it. |
| [`testing`](skills/testing/loop.md) | `flakiness` · `suite-speed` · `coverage` | Finds risky changed behaviour with no tests, tests that fail at random, and tests that cost more time than the confidence they buy — and proposes the fix. |
| [`performance`](skills/performance/loop.md) | `query-latency` · `hot-path-blocking` · `infra-spend` · `algorithmic-cost` | Hunts slow queries, slow endpoints and latency spikes and proposes targeted speedups — and the infrastructure spend that buys nothing: a poll that should be a webhook, an always-on service, needless egress. |
| [`react`](skills/react/loop.md) | `state-and-effects` · `render-performance` · `markup-safety` | Checks React components for bugs and wasted renders — stale state, effect misuse, needless re-renders — and proposes fixes. |
| [`code-quality`](skills/code-quality/loop.md) | `architecture` · `propagation` · `deslop` | Cleans up what an AI-written codebase accretes — dead code and pointless wrappers, duplicate helpers, modules that grew into grab-bags, and references left behind by a rename — without changing what the app does. |
| [`ai-engineering`](skills/ai-engineering/loop.md) | `context-engineering` · `harness` · `evals` · `retrieval` · `llm-spend` | Audits how your product builds with LLMs — agent harness, prompts and tools, context, memory, retrieval, evals and what it all costs to run — and proposes the highest-leverage upgrade. |
| [`backend`](skills/backend/loop.md) | `schema-and-migrations` · `query-and-index` · `idempotency-and-jobs` · `api-correctness` | Audits everything behind your API for the bugs a founder can't see — unvalidated input, a retry that charges twice, two requests racing into a duplicate, a swallowed failure, a query with no index, a migration that could break the next deploy — and proposes the fix. |
| [`payments`](skills/payments/loop.md) | `webhooks-and-fulfillment` · `entitlements` · `billing-config` | Audits your billing code for the bugs that lose money — a paid signup that provisions nothing, a retry that double-charges, access that's never revoked on cancel, a drifted price id — and proposes the fix. |
| [`observability`](skills/observability/loop.md) | `uptime-and-heartbeats` · `crash-reporting` · `log-quality` · `tracing-and-metrics` | Finds parts of production you can't see — unwatched services, silent jobs, vanished crashes, swallowed errors, untraced critical paths — and proposes the missing check, log, or span. |
| [`alerts`](skills/alerts/loop.md) | `rule-quality` · `coverage-gaps` | Finds critical paths with no alert — payment failures, broken webhooks, auth errors — and proposes the exact Sentry rule to catch them. |
| [`product-analytics`](skills/product-analytics/loop.md) | `event-hygiene` · `event-coverage` | Checks whether analytics is even wired up and, if not, sets you up — what to track, where, and a tracking plan — so you can see your funnel. |

## The methods

Each one states what it audits, how to verify a finding before claiming it, and what does **not** belong to it — the seams matter, because several of these sound alike from the outside.

### security

Audits recent code changes for security holes — auth gaps, exposed data, injection, tenant leaks — and proposes fixes.

| Skill | What it audits |
| --- | --- |
| [`auth-and-access`](skills/security/skills/auth-and-access/SKILL.md) | Audits who is allowed to do what — route-level ownership and role checks, IDOR, TOCTOU, entitlement enforcement — and the signup/login/reset/session flows that hand out identity in the first place. Use when reviewing auth middleware, session handling, permission checks, account-lifecycle flows, or any route that reads someone else's object by id. |
| [`tenant-isolation`](skills/security/skills/tenant-isolation/SKILL.md) | Audits whether one customer can reach another customer's data — row-level security coverage and policy correctness, the service-role bypass, tenant scoping taken from session state rather than the request, and cross-tenant leaks through joins, caches, exports and admin paths. Use when reviewing migrations, database policies, tenant-scoped queries, or any shared-table schema. |
| [`money-and-webhooks`](skills/security/skills/money-and-webhooks/SKILL.md) | Audits whether someone can get paid features without paying, and whether inbound provider webhooks can be forged, replayed or pointed at another account. Use when reviewing checkout, subscription and entitlement code, or any webhook receiver that grants access or credits. |
| [`secrets-and-crypto`](skills/security/skills/secrets-and-crypto/SKILL.md) | Audits how credentials are stored, exposed and rotated — hardcoded keys, secrets in logs, error bodies, health endpoints and client bundles — plus the crypto underneath (password hashing, token randomness, hand-rolled schemes) and the supply-chain surface of the dependencies themselves. Use when reviewing config and env handling, logging, build/bundle output, auth token generation, or dependency changes. |
| [`injection-and-input`](skills/security/skills/injection-and-input/SKILL.md) | Audits untrusted input reaching a dangerous sink — SQL/NoSQL concatenation, shell execution, SSRF, path traversal, unsafe deserialization, unsafe rendering — and the agent-specific version of the same bug: prompt injection, unverified tool authorization, and model output trusted downstream. Use when reviewing route handlers, query building, file or URL handling, or any LLM/agent code path. |

### reliability

Audits your code for the ways it breaks under real-world failure — a dependency that times out with no deadline, a retry that runs a charge twice, a swallowed error that looks like success, an unbounded operation that exhausts memory — and proposes the fix. Sharper when Sentry is connected, but works without it.

| Skill | What it audits |
| --- | --- |
| [`dependency-boundary`](skills/reliability/skills/dependency-boundary/SKILL.md) | Audits every call that leaves the process — HTTP, database, cache, queue, another service — for deadlines, retry policy, backoff and blast-radius containment: calls that can hang forever, tight retry loops that amplify an outage, no jitter, no cap, and a fragile dependency with nothing between it and the rest of the system. Use when reviewing API clients, SDK wrappers, connectors or integration code. |
| [`retry-safety`](skills/reliability/skills/retry-safety/SKILL.md) | Audits state changes that can run more than once — a webhook redelivered, a job re-run after a crash, a queue that is at-least-once, a user double-submitting — for the dedupe that makes the second execution harmless: idempotency keys, unique constraints, atomic upserts, and side effects that cannot be replayed. Use when reviewing webhooks, queues, jobs, cron, or any retried write. |
| [`resource-limits`](skills/reliability/skills/resource-limits/SKILL.md) | Audits what runs out under load — connection-pool starvation, a connection or transaction held across an await, unbounded queries and in-memory accumulators, streams and uploads with no size limit or backpressure, and listeners, timers or handles that leak. Use when reviewing pooling, connection or transaction handling, streaming, uploads, or any code that accumulates. |
| [`failure-handling`](skills/reliability/skills/failure-handling/SKILL.md) | Audits what the code does once something has already failed — empty catches, errors logged and then continued past on a write, unhandled promise rejections, unawaited writes, a failure that returns success, and multi-step work left half-applied with no compensation or recovery. Use for ordinary service and handler code with no dependency, queue or resource surface in sight. |

### testing

Finds risky changed behaviour with no tests, tests that fail at random, and tests that cost more time than the confidence they buy — and proposes the fix.

| Skill | What it audits |
| --- | --- |
| [`flakiness`](skills/testing/skills/flakiness/SKILL.md) | Audits tests for non-determinism — fixed sleeps instead of waits on a condition, state shared across tests, order dependence, real clocks and timezones, unseeded randomness, real network or filesystem I/O, races and leaked handles — and proposes a stabilization or an honest quarantine. Use when reviewing e2e, integration or shared fixture/setup code, or a suite that flaps. |
| [`suite-speed`](skills/testing/skills/suite-speed/SKILL.md) | Audits the test suite for wall-clock it does not need to spend — real network, database, filesystem or timer I/O that should be faked, heavy per-test setup that could be shared, oversized fixtures, serial-but-independent cases, redundant assertions and over-broad snapshots — and proposes rewrites that keep every check. Use when test files or runner config changed, or the suite has never been reviewed for speed. |
| [`coverage`](skills/testing/skills/coverage/SKILL.md) | Audits changed behaviour for the tests it does not have — new branches and error paths, bug fixes with no regression test, new edge and boundary cases, contracts whose existing tests no longer assert them, and logic with no single expected output that needs a property-based test instead. Use when product code changed and the question is whether a regression would be caught. |

### performance

Hunts slow queries, slow endpoints and latency spikes and proposes targeted speedups — and the infrastructure spend that buys nothing: a poll that should be a webhook, an always-on service, needless egress.

| Skill | What it audits |
| --- | --- |
| [`query-latency`](skills/performance/skills/query-latency/SKILL.md) | Audits how the app reads data under load — a query inside a loop (N+1), a per-row lookup that should be one batched or joined query, unbounded or unindexed queries on a hot path, SELECT * pulling heavy columns, and pagination that scans the whole table. Use when reviewing database access, ORM calls, repositories, or any handler that reads before it responds. |
| [`hot-path-blocking`](skills/performance/skills/hot-path-blocking/SKILL.md) | Audits what a request waits on — synchronous I/O and heavy CPU in a handler, an external call with no timeout, work that should be queued or backgrounded, a response built without a cache, and payloads or assets that make the client wait. Use when reviewing routes, handlers, controllers, middleware, streaming or upload paths. |
| [`infra-spend`](skills/performance/skills/infra-spend/SKILL.md) | Audits the machine bill — a cron or poll where an event would do, a schedule far tighter than its inputs change, an always-on service that could be on-demand, work redone every tick with no cache, egress from over-fetch or missing cache headers, chatty calls to a metered API, and paid services still wired but unused. Use when reviewing cron, jobs, queues, workers, deployment or infrastructure config. |
| [`algorithmic-cost`](skills/performance/skills/algorithmic-cost/SKILL.md) | Audits the code itself for work that scales badly — a new nested loop turning O(n) into O(n²), a lookup that scans a list where a map would index it, repeated pure computation that should be memoized, a whole collection loaded to count or filter it in app, and allocation or copying that could be hoisted. Use for ordinary modules, transforms and utilities with no database or route in sight. |

### react

Checks React components for bugs and wasted renders — stale state, effect misuse, needless re-renders — and proposes fixes.

| Skill | What it audits |
| --- | --- |
| [`state-and-effects`](skills/react/skills/state-and-effects/SKILL.md) | Audits where component state lives and when effects run — array-index keys, missing or unstable effect dependencies, state that should be derived during render, effects doing work that belongs in an event handler, unhandled async races, and hooks called conditionally. Use when reviewing hooks, providers, forms, or any component holding state. |
| [`render-performance`](skills/react/skills/render-performance/SKILL.md) | Audits what a component costs to render and how widely it re-renders others — unstable props and context values, over-broad subscriptions, expensive work in render, long unvirtualised lists, and updates that block typing. Use when reviewing lists, tables, charts, modals, providers, or a component that feels slow. |
| [`markup-safety`](skills/react/skills/markup-safety/SKILL.md) | Audits what the rendered markup does to the user — unsanitised HTML injected into the DOM, and interactive elements without an accessible name, keyboard path, focus handling or correct semantics. Use when reviewing any component that renders user-supplied content or builds controls out of non-semantic elements. |

### code-quality

Cleans up what an AI-written codebase accretes — dead code and pointless wrappers, duplicate helpers, modules that grew into grab-bags, and references left behind by a rename — without changing what the app does.

| Skill | What it audits |
| --- | --- |
| [`architecture`](skills/code-quality/skills/architecture/SKILL.md) | Audits the module level — layering violations, new coupling and circular imports, logic duplicated instead of reused, responsibility creep, vendor specifics leaking through a seam, and files that grew into grab-bags. Use when reviewing services, libraries, core modules, barrels, adapters, or any change that moves logic across a boundary. |
| [`propagation`](skills/code-quality/skills/propagation/SKILL.md) | Audits what a rename left behind — every place still using the old form of a value that changed identity: env vars, exported constants and enum members, config keys, API routes, and the docs that cite them. Use when a diff renames or replaces a value, or when reviewing config, constants, env files or documentation. |
| [`deslop`](skills/code-quality/skills/deslop/SKILL.md) | Audits the statement level for AI-generated cruft — pass-through wrappers, single-use helpers that earn nothing, dead and speculative code, comments restating the code, nested ternaries, near-duplicate logic, and markup buried under redundant wrappers. Use when reviewing ordinary code churn that no sharper skill claims. |

### ai-engineering

Audits how your product builds with LLMs — agent harness, prompts and tools, context, memory, retrieval, evals and what it all costs to run — and proposes the highest-leverage upgrade.

| Skill | What it audits |
| --- | --- |
| [`context-engineering`](skills/ai-engineering/skills/context-engineering/SKILL.md) | Audits what fills a product's model context and whether any of it is dead weight or dead ends — memory written but never read, prompts demanding facts nothing supplies, blocks that grew past their usefulness. Use when reviewing prompts, memory, summarisation, compaction, or any injected context block. |
| [`harness`](skills/ai-engineering/skills/harness/SKILL.md) | Audits the agent loop itself — whether the 'agent' is a real harness with tool use, state and error recovery, and whether its prompts, tools and untrusted-input handling hold up in production. Use when reviewing agent code, tool definitions, MCP servers, model calls, or prompt-injection surface. |
| [`evals`](skills/ai-engineering/skills/evals/SKILL.md) | Audits whether a workspace can change a prompt, model or retrieval setting and KNOW it improved things, and whether production misbehaviour is diagnosable. Use when reviewing eval harnesses, golden sets, graders, judges, traces or LLM observability — or when nothing measures quality at all. |
| [`retrieval`](skills/ai-engineering/skills/retrieval/SKILL.md) | Audits how a product gets knowledge into the model's context — whether retrieval is the right shape at all, and whether it surfaces the right material or merely the nearest. Use when reviewing RAG pipelines, embeddings, chunking, vector stores, rerankers or query rewriting. Bundles a method catalog in references/. |
| [`llm-spend`](skills/ai-engineering/skills/llm-spend/SKILL.md) | Audits what the product pays to run its LLM calls — agent turn tax, context bloat, model-escalation ladders, missing prompt caching, an oversized model for a bounded task, unbatched bulk calls, retries with no ceiling, and tool surfaces paid for in every cached prefix. Use when reviewing an agent loop, a model call site, token/usage accounting, or any change whose question is "same output, less money". |

### backend

Audits everything behind your API for the bugs a founder can't see — unvalidated input, a retry that charges twice, two requests racing into a duplicate, a swallowed failure, a query with no index, a migration that could break the next deploy — and proposes the fix.

| Skill | What it audits |
| --- | --- |
| [`schema-and-migrations`](skills/backend/skills/schema-and-migrations/SKILL.md) | Audits the shape of the data and how it changes — migrations that must survive a non-atomic deploy, constraints that keep bad rows out, data types, and connection/pool configuration. Use when reviewing migration files, schema definitions, ORM models, or database client setup. |
| [`query-and-index`](skills/backend/skills/query-and-index/SKILL.md) | Audits how the app reads — N+1 queries, filters and joins on unindexed columns, unbounded or deep-paginated scans, over-fetching, and ORM calls that emit something pathological. Use when reviewing queries, repositories, ORM models, or any endpoint whose cost grows with the table. |
| [`idempotency-and-jobs`](skills/backend/skills/idempotency-and-jobs/SKILL.md) | Audits anything that can happen twice or interleave — non-idempotent state changes under retry or redelivery, check-then-act races, multi-step writes with no transaction, webhook receivers, and queue/cron consumers without retry, dead-letter or lock discipline. Use when reviewing webhooks, jobs, queues, cron, or any handler that creates, charges or provisions. |
| [`api-correctness`](skills/backend/skills/api-correctness/SKILL.md) | Audits the trust boundary of a request handler — input validated where it enters, errors that fail loudly instead of reporting success, and the defaults that decide behaviour under load (pagination caps, timeouts, body limits, rate limits, CORS). Use when reviewing routes, controllers, API handlers or middleware. |

### payments

Audits your billing code for the bugs that lose money — a paid signup that provisions nothing, a retry that double-charges, access that's never revoked on cancel, a drifted price id — and proposes the fix.

| Skill | What it audits |
| --- | --- |
| [`webhooks-and-fulfillment`](skills/payments/skills/webhooks-and-fulfillment/SKILL.md) | Audits whether the payment provider's events are all handled and whether fulfilment runs exactly once — missing money-critical events, at-least-once delivery with no dedupe, out-of-order updates, and the trial path that provisions nothing. Use when reviewing a payment webhook handler, checkout completion, or any code that provisions after money moves. |
| [`entitlements`](skills/payments/skills/entitlements/SKILL.md) | Audits what the customer can actually do after the money moves — whether every grant has a matching revoke, whether cancellation, expiry and failed renewals reach a defined state, and whether access is decided from real subscription state rather than a stale cache. Use when reviewing provisioning, plan/tier gating, credits, trials, or dunning logic. |
| [`billing-config`](skills/payments/skills/billing-config/SKILL.md) | Audits the configuration that decides what gets sold — price and plan ids split between env and code, test-mode vs live-mode keys, plan-to-feature maps that drift from the provider, currency/tax assumptions, and webhook endpoint settings. Use when reviewing pricing config, plan constants, env handling, or a price/plan change. |

### observability

Finds parts of production you can't see — unwatched services, silent jobs, vanished crashes, swallowed errors, untraced critical paths — and proposes the missing check, log, or span.

| Skill | What it audits |
| --- | --- |
| [`uptime-and-heartbeats`](skills/observability/skills/uptime-and-heartbeats/SKILL.md) | Audits unattended work for a watcher — a deployed service with no health endpoint or a shallow always-green one, a health check nothing pings, a scheduled or background job with no check-in, and a check-in carrying no monitor config so no monitor exists to go late. Use when reviewing servers, cron, queues, workers, schedulers or CI workflow files. |
| [`crash-reporting`](skills/observability/skills/crash-reporting/SKILL.md) | Audits every application entrypoint for error reporting — a server, worker, serverless function, CLI or client runtime that initializes no error-tracking SDK, an init that misses unhandled rejections, and an SDK configured without environment, release or sane sampling so its output cannot be acted on. Use when reviewing entrypoints, bootstrap and server startup code, or error-tracking setup. |
| [`log-quality`](skills/observability/skills/log-quality/SKILL.md) | Audits what the code leaves behind when it fails — an error branch or empty catch with no record, unstructured messages with no context or correlation id, log levels that make real failures invisible in noise, and PII, tokens or secrets written into a log line. Use when reviewing logging setup, loggers, redaction helpers, or any failure path. |
| [`tracing-and-metrics`](skills/observability/skills/tracing-and-metrics/SKILL.md) | Audits flight-critical paths for whether their success, failure and latency are observable — payments, auth, outbound third-party calls, queue work and hot handlers that emit no spans or metrics, spans that record no error status, and instrumentation that carries unbounded or sensitive attributes. Use for ordinary service and handler code, or when reviewing tracing, telemetry or metrics setup. |

### alerts

Finds critical paths with no alert — payment failures, broken webhooks, auth errors — and proposes the exact Sentry rule to catch them.

| Skill | What it audits |
| --- | --- |
| [`rule-quality`](skills/alerts/skills/rule-quality/SKILL.md) | Audits alert rules that already exist for whether they would actually help — thresholds too tight to survive normal traffic or too loose to ever fire, rules watching a path whose failure is never reported, duplicate and overlapping rules, alerts with no owner or route, and coverage that drifted after the code moved. Use when reviewing alerting, monitoring or notification configuration. |
| [`coverage-gaps`](skills/alerts/skills/coverage-gaps/SKILL.md) | Computes the critical paths that exist in the code minus the alert rules that exist in the project, and proposes the missing alert with a concrete condition and threshold — payment and billing failures, inbound webhook errors, auth failures, error-rate and latency breaches on hot endpoints. Use when auditing application code for failures nothing would report. |

### product-analytics

Checks whether analytics is even wired up and, if not, sets you up — what to track, where, and a tracking plan — so you can see your funnel.

| Skill | What it audits |
| --- | --- |
| [`event-hygiene`](skills/product-analytics/skills/event-hygiene/SKILL.md) | Audits whether captured analytics can actually be used — is an SDK really initialized and firing, are names consistent and machine-parseable, do events carry the properties a question needs, is PII or a token being sent as a property, and is there a tracking plan or just an accumulated pile. Use when reviewing analytics setup, tracking helpers, or event definitions. |
| [`event-coverage`](skills/product-analytics/skills/event-coverage/SKILL.md) | Audits the product's real user journeys for whether anything captures them — acquisition and signup, the activation moment where a user first gets value, the core repeated action the product exists for, and conversion or revenue steps — and proposes the specific capture calls and where they go. Use when reviewing onboarding, signup, checkout, pricing or any flow a user moves through. |

## Using one well

A skill is a method, not a linter. Point it at a real change — a diff, a pull request, a module you are about to touch — rather than at a whole repository, and read the shelf's `loop.md` first: it carries the standard of evidence the method assumes. The security shelf's rule is the general one — the bar is a traced request, not a matched pattern.

## Licence

MIT. Use them, fork them, fold them into your own review process.
