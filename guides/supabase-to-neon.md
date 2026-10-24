# Migrate an application from Supabase to Neon

Start by deciding what is moving. Both platforms provide PostgreSQL, but an application may depend on more than its tables.

This guide prepares a migration and its acceptance checks. It is not a command to migrate a live database. Keep project-specific credentials, exports and validation evidence private.

## 1. Choose the outcome

| Outcome | Scope | Main consequence |
| --- | --- | --- |
| Move the application database | PostgreSQL moves; selected Supabase services remain | Retained services still need their original backend, permissions and billing. |
| Replace Supabase completely | Database, identity, files, APIs and events move | Each used service needs an implementation and acceptance plan. |
| Rebuild a prototype | Reuse useful UI/domain code and start a new backend | A fresh seed is not a transfer of existing users or historical records. |

Choose a migration because it improves a measured requirement: cost, operations, deployment workflow or architecture. Include the engineering and verification work in the comparison.

Use the [plan template](../templates/supabase-to-neon-plan.md) to record the actual repository/commit, deployed source, destination project/branch, writer and approvals. Do not infer the current deployment from the latest local commit.

## 2. Inventory the dependencies

Inspect application code, SQL migrations, infrastructure definitions and current service configuration. Mark unknowns explicitly.

| Area | Look for | Decision to make |
| --- | --- | --- |
| Database | Schemas, tables, views, sequences, extensions, RPCs, triggers, scheduled SQL, grants and RLS | Which objects/data move, and which dependencies must exist first? |
| Auth | `supabase.auth`, user IDs, profiles, OAuth, MFA, sessions, `auth.users` and `auth.uid()` | Keep the identity provider or migrate it separately? |
| Client API | `.from()`, `.rpc()`, generated types and error handling | Keep compatible endpoints or adapt to an application API? |
| Storage | Buckets, actual objects, metadata, signed URLs and upload paths | Keep the file service or transfer bytes and permissions? |
| Realtime | Database changes, broadcast, presence and private channels | Which subscriptions and authorization semantics must be preserved? |
| Functions/jobs | Edge Functions, cron, queues, secrets, webhooks, retry and idempotency | Where will each handler run, and what can it trigger? |

A `public`-only export does not include every service or dependency. Copying all internal schemas blindly is not a replacement for this inventory. Neon publishes a [database migration guide](https://neon.com/docs/import/migrate-from-supabase); use its [import index](https://neon.com/docs/import/llms.txt) to locate the current version if the page moves.

## 3. Preserve identity and authorization

**Keeping Supabase Auth:** keep its sign-in/refresh flow and verify its tokens on the target API. Validate signature, issuer, audience and expiry using the provider's current guidance. Preserve subject IDs and the relationships from those IDs to application rows. Document how user lifecycle changes reach the new database; an old cross-database foreign key or trigger will not continue to enforce that relationship by itself.

**Replacing Auth:** plan users, linked identities, password compatibility, OAuth callbacks, MFA, sessions and account recovery separately. Decide whether re-login or password reset is required. Never assume importing a user table recreates an authentication service. [Supabase user and identity model](https://supabase.com/docs/guides/auth/users).

**Database access:** review both grants and RLS. Functions such as `auth.uid()` need a supported identity context at the destination. Test through the actual non-owner runtime role and real request path. Do not fix a failed import by granting everyone broad access or disabling isolation. Keep migration-owner credentials separate from application credentials. [Supabase RLS](https://supabase.com/docs/guides/database/postgres/row-level-security).

The browser must not receive a database password or privileged service key. If the client calls Supabase's Data API directly, choose a compatible target API or adapt those calls. An API that accepts an arbitrary `user_id` from the browser is not an authorization boundary.

## 4. Choose the data-transfer method

### Small database with an acceptable write pause

Use a rehearsed dump/restore. Measure export, restore, validation and client-switch time. A consistent export represents one point in time; later writes are not included. Define how writers and background jobs stop for the final transfer.

### Larger or continuously written database

Evaluate supported logical replication or another CDC mechanism. Confirm source permissions, table coverage, replica identity, initial synchronization, lag monitoring and WAL/slot retention. Keep the final switch measurable and bounded.

PostgreSQL logical replication does not automatically replicate DDL, sequence state or large objects. Plan those separately, and verify restrictions for your actual versions. Low lag is not proof that every required object moved. [PostgreSQL replication restrictions](https://www.postgresql.org/docs/current/logical-replication-restrictions.html).

Avoid introducing dual writes without a design for failures, ordering and reconciliation. The migration can often use a short final write pause instead.

## 5. Rehearse in an isolated destination

Use a clearly identified empty test database or an approved Neon branch. A branch holding production data is still sensitive. Disable jobs, email, webhooks and paid-provider calls so the rehearsal cannot contact customers or repeat business actions.

Check source/target PostgreSQL versions, dump/restore client compatibility, extension availability, network reachability and allowed roles. Use direct migration connections rather than assuming an application transaction-pooler is suitable.

The example below covers a **reviewed application-owned `public` schema**. Resolve its external dependencies first. Add other schemas or large objects only after reviewing their ownership and purpose. This is not an Auth or Storage migration.

Configure private libpq services named `supabase_source` and `neon_rehearsal` outside the checkout, with host/database/user/TLS settings and the approved password mechanism. A [connection service file](https://www.postgresql.org/docs/current/libpq-pgservice.html) lets the commands refer to names instead of exposing connection strings. Keep credential files permission-restricted; do not paste them into an AI chat.

```sh
# Confirm compatible PostgreSQL client versions first.
pg_dump --version
pg_restore --version

# Check both service mappings against the approved project/branch endpoints.
# Database names alone are not enough to identify the destination.
psql 'service=supabase_source' -X -v ON_ERROR_STOP=1 \
  -c 'SELECT current_database(), current_user, version();'
psql 'service=neon_rehearsal' -X -v ON_ERROR_STOP=1 \
  -c 'SELECT current_database(), current_user, version();'
```

After confirming the destination is the intended empty rehearsal database:

```sh
umask 077
MIGRATION_WORKDIR="$(mktemp -d)"

pg_dump --dbname='service=supabase_source' \
  --format=custom --schema=public \
  --file="$MIGRATION_WORKDIR/application.dump"

# Review the archive's object list before executing it at the destination.
pg_restore --list "$MIGRATION_WORKDIR/application.dump" \
  > "$MIGRATION_WORKDIR/objects.txt"

pg_restore --dbname='service=neon_rehearsal' \
  --no-owner --no-acl --single-transaction --exit-on-error \
  "$MIGRATION_WORKDIR/application.dump"
```

Run each operation only after the preceding one succeeds. Keep errors visible and investigate them. A dump can contain executable routines and triggers; review its source and contents before restoring. This example uses a single transaction for an atomic rehearsal restore, without parallel jobs. Larger imports may need a different reviewed approach. [pg_dump](https://www.postgresql.org/docs/current/app-pgdump.html), [pg_restore](https://www.postgresql.org/docs/current/app-pgrestore.html).

`--no-owner` and `--no-acl` intentionally omit ownership/grant restoration. Recreate reviewed least-privilege ownership and grants before testing the application. A successful import as the owner does not prove that the runtime can access only permitted rows.

Store artifacts according to the agreed private retention/encryption policy. Do not commit the dump or logs containing data. Before cutover, repeat the measured process with a final consistent transfer; do not promote a stale rehearsal copy accidentally.

## 6. Handle services outside the SQL dump

**Files:** Supabase stores object metadata in PostgreSQL and object bytes separately. Transfer bytes through supported storage APIs/tools if moving them; reconcile object keys, sizes/checksums, content types and access rules. Test private downloads, new uploads and signed URL behavior. Do not edit Supabase's internal Storage tables to copy or delete files. [Storage schema](https://supabase.com/docs/guides/storage/schema/design).

**Functions and jobs:** preserve authentication, schedules, webhook verification, retries and idempotency. Inventory provider secrets by reference, never by copying values into the plan. Prevent both old and new workers from executing the same side effect during transition.

**Realtime:** reproduce the events the application needs, including private-channel authorization, reconnect behavior and resynchronization after a lost signal. Switching a database does not reconnect existing clients automatically.

Check current Neon capabilities and regional availability before choosing replacements. Retaining a working service for the first stage is a valid option; record its remaining operational dependency.

## 7. Validate data and actual behavior

Build acceptance checks from the application's contracts, including failure cases. Record the source commit, target environment, date and whether a check used fixtures, an isolated database or a real deployed service.

- Reconcile table counts at the same snapshot/write boundary; investigate differences.
- Check stable IDs, foreign keys, sequences/identity values and suitable deterministic checksums. Counts alone can miss corrupted or mismatched rows.
- Exercise inserts, updates and deletes through the intended runtime role.
- Test anonymous rejection, account A/B isolation and every privileged role.
- Test login, refresh, logout and account recovery under the selected identity strategy.
- Test files, required subscriptions, background jobs and retry/replay behavior.
- Exercise the actual web/mobile/extension clients that use the API.

Do not publish a validation report containing user rows, emails, tokens or private object URLs. Keep a sanitized summary and private evidence separately.

## 8. Review the cost of the whole system

Compare database compute, data/history storage, branches, egress, application hosting, identity, object storage, queues and observability. Include migration rehearsal and temporary overlap of both providers.

Neon compute usage is weighted by compute size and active time. Background queries may prevent suspension. A free allowance is not a guarantee of free continuous production operation, and a spending notification is not necessarily a hard cap. Recheck [current pricing](https://neon.com/pricing) and [usage calculations](https://neon.com/docs/introduction/usage-calculations) rather than baking a price into the migration script.

## 9. Cut over and define recovery

Prepare a reviewed runbook before any production change. Confirm applicable authorization for its exact target and actions; preserve permission already given.

1. Confirm the current source/target, recovery artifact, application version and write owners.
2. Pause writers/jobs as planned; finish the final transfer or measured catch-up.
3. Run the agreed data/security checks and resolve sequence/object gaps.
4. Switch the intended runtime configuration; keep secrets server-side.
5. Verify real client journeys and monitored error/latency thresholds.
6. Re-enable one set of jobs and observe the agreed acceptance window.

Define the abort conditions, recovery owner and tolerated data loss/downtime in advance. Once the target accepts new writes, switching the connection back can lose those writes. Recovery needs reconciliation or another tested mechanism. A code rollback also does not automatically reverse schema or external side effects.

Keep the old services until all remaining dependencies and the acceptance period are resolved. Do not delete the source as part of a successful import.

## Completion record

Use precise stages: **AUDITED → PREPARED → REHEARSED → CUT OVER → LIVE VERIFIED**. List services deliberately retained, unmigrated data, failed/unrun checks and remaining approvals. Keep detailed evidence with the implementation; use the [plan template](../templates/supabase-to-neon-plan.md) for a concise summary.

Documentation reviewed: **2026-10-24**. No production migration was executed to publish this guide.
