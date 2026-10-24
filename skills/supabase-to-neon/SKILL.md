---
name: supabase-to-neon
description: "Audit and migrate an existing Supabase application to Neon while preserving data, identities and access rules. Use for Supabase-to-Neon migration plans, dependency inventories, database rehearsals, application adapters, validation and cutover preparation. Distinguish database migration from replacing Auth, Storage, Realtime, Edge Functions and browser SDK calls."
---

# Supabase to Neon

Turn an existing application's dependencies into an explicit, verified migration. Begin with a read-only audit; implement or execute only within the user's authorized scope. Reuse existing authorization instead of asking again. Creating this plan does not authorize a production cutover.

## Establish the actual application

1. Read local instructions and the current handoff. Verify source repository, branch, HEAD, dirty work, current writer, deployed version and the exact target environment. Preserve unrelated changes.
2. If a supplied repository is empty or inaccessible, report that limitation. Request the actual source or use an explicitly identified local source. Never invent its schema or treat a case study as its implementation.
3. Identify the outcome: move only Postgres, replace the entire backend, or start a new backend with reused UI. An empty new database with seed data is not a migration of existing users and records.
4. Inventory code and migration files before touching infrastructure. Search filenames first; exclude environment files, credentials, dumps, generated bundles and dependency directories. Report paths and dependency categories rather than printing keys or user content.

Produce one compact inventory:

| Dependency | Source evidence | Keep / replace / unused / unknown | Target and validation |
| --- | --- | --- | --- |
| Postgres | Tables, extensions, routines, triggers, sequences, RLS, grants, jobs | | |
| Auth | SDK calls, providers, IDs, sessions, JWT verification, `auth.users` / `auth.uid()` references | | |
| Data API | Browser `.from()`, `.rpc()`, generated types and authorization assumptions | | |
| Storage | Buckets, object bytes, metadata, permissions, signed URLs, uploads | | |
| Realtime | Subscriptions, broadcast, presence and per-account authorization | | |
| Edge Functions | Handlers, secrets, webhooks, schedules, external APIs and retries | | |

## Choose the smallest viable migration

- Preserve a working identity provider unless replacing it is part of the requested outcome. If retaining Supabase Auth, verify its existing JWTs server-side and preserve subject IDs and account relationships; explicitly document the remaining Supabase dependency.
- If moving Auth, plan identity/password/OAuth/session compatibility separately. Verify export/import support for the actual providers. Do not assume copying `auth.users` makes login work or that sessions survive. Obtain a decision on resets or re-login if required.
- Browser Supabase SDK calls need an appropriate target API or an application backend. Never expose a database connection string, owner role or service credential in the client.
- Map authorization deliberately. Copying an RLS policy does not provide its missing auth functions, roles or verified claims. Test using the real non-owner application role; an owner query does not prove isolation. Prevent identity context from leaking through pooled connections.
- Choose replacements for Storage, Realtime and Functions only where used. Check current Neon features, region availability and compatibility; neither assume Neon only provides Postgres nor assume Supabase APIs are interchangeable with Neon APIs.
- Compare cost and latency of the whole route: database compute/storage/egress, API hosting, object storage, auth, jobs, observability and external providers. Account for cold starts and background polling that prevents suspension. Do not promise production uptime or free operation from a project allowance.

Consult the current [official references](references/migration-notes.md#official-references) before selecting commands, compatibility assumptions or prices. Use the available Neon/Supabase skills when applicable. Avoid provisioning or switching providers during an audit.

## Prepare and rehearse

1. State source and destination identity, required access, data scope, size, tolerated downtime, budget, recovery point and recovery time. Do not request credentials in chat; use the user's approved secret mechanism without exposing values in arguments, logs or Git.
2. For a small database with an acceptable write pause, prepare `pg_dump` / `pg_restore`. Verify client/server versions, direct or otherwise officially supported migration connections, extensions and a reviewed schema/object list. A `public`-only export is not a complete Supabase backend export.
3. When write downtime must be minimized, evaluate supported logical replication/CDC. Verify source permissions, data types, replica identity, slot/WAL impact, schema changes and any separately copied objects. Define a measurable lag gate and final write pause. Do not promise zero downtime.
4. Restore into an isolated, explicitly identified test destination or Neon branch within authorization. A branch containing production data is still sensitive. Disable outbound email, webhooks, paid providers and scheduled jobs during rehearsal.
5. If owner/ACL restoration is omitted, recreate reviewed application ownership and least-privilege grants explicitly. Do not suppress restore errors or remove security policies just to obtain a passing import.
6. Reconcile per-table counts, key relationships, stable IDs, sequence/identity positions and suitable deterministic checksums. Verify constraints, routines, triggers and required extensions. For files, validate object bytes/checksums and permissions separately from storage metadata.
7. Exercise login/logout/refresh, anonymous denial, account A versus B, representative read/write/delete, uploads, subscription isolation and retry/idempotency where relevant. Distinguish fixture tests, isolated database tests and real provider/runtime tests.
8. Rehearse recovery. Preserve the source and migration artifacts according to the agreed retention plan. Never describe changing a connection string back as sufficient recovery after new writes have reached the target.

## Cut over only within applicable authorization

Prepare a concrete cutover and recovery runbook before requesting any missing production authorization. Include the tested artifact and migration set, write freeze, final transfer/catch-up, verification gates, configuration switch, client checks, abort thresholds and named owner. Preserve existing explicit authorization for those exact actions.

Once the target accepts new writes, define how to reconcile those writes before returning to the source. Avoid unplanned dual writes. Do not delete, pause or decommission the old provider until its remaining dependencies and the acceptance window have been resolved.

## Return a reviewable result

Keep one concise migration record with inventory, selected strategy, actual changes, source/target versions, validation results, cost assumptions, recovery limits and remaining gates. Link detailed implementation evidence rather than duplicating it. Commit coherent local work when repository policy requires it; push and publish only within applicable authorization.

Label the outcome precisely: **AUDITED → PREPARED → REHEARSED → CUT OVER → LIVE VERIFIED**. Report which stage was reached and which users/data/features remain unmigrated. Read [migration notes](references/migration-notes.md) for official sources and acceptance scenarios.
