# Supabase → Neon migration plan

Keep the completed plan private if it contains infrastructure or customer information. Use credential references, never values.

## Starting point

- Goal and reason for migrating:
- Scope: database only / full backend / fresh backend:
- Source repository, branch, commit and current writer:
- Deployed source version:
- Source database/project/environment and snapshot boundary:
- Destination project/branch/database/environment:
- PostgreSQL/client versions and required extensions:
- Data size, write rate and client platforms:
- Existing authorization, budget and excluded actions:
- Required access still missing:

## Dependencies

| Service or object group | Source evidence | Keep / move / replace / unused / unknown | Destination | Acceptance check |
| --- | --- | --- | --- | --- |
| PostgreSQL | | | | |
| Auth and stable user IDs | | | | |
| RLS, roles, grants and RPCs | | | | |
| Browser API/SDK | | | | |
| Storage bytes and metadata | | | | |
| Realtime | | | | |
| Edge Functions, jobs and external APIs | | | | |

## Transfer and rehearsal

- Method and measured duration:
- Data/schema allowlist and unresolved dependencies:
- Write-freeze or replication/catch-up strategy:
- Private recovery/export location and retention policy:
- Rehearsal destination; outbound side effects disabled:
- Counts/IDs/relationships/checksums/sequence validation:
- Anonymous/A/B/privileged-role checks:
- Login, files, jobs and actual-client checks:
- Failures and checks not run:

## Cost and cutover

- Ongoing cost assumptions and overlap cost:
- Tolerated write downtime and data loss:
- Cutover owner, steps, authorization and abort thresholds:
- Recovery method before target writes:
- Reconciliation/recovery after target writes:
- Observation window and old-service retirement conditions:

## Result

- Stage: AUDITED / PREPARED / REHEARSED / CUT OVER / LIVE VERIFIED
- Tested commit/artifact, environment, date and evidence:
- Actual changes and retained services:
- Unmigrated users/data/features:
- Remaining decisions or approvals:
