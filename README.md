# Zero to 100

Practical guides, workflows and AI skills for building products solo.

By [Dante Galeazzi](https://dantegaleazzi.com). More experiments and write-ups at [dantegaleazzi.com](https://dantegaleazzi.com).

## Start here: Supabase → Neon

A guide and reusable agent skill for moving an existing application from Supabase to Neon while preserving its data and access rules.

**Moving PostgreSQL tables and replacing the whole Supabase backend are different projects.** Auth, Storage, Realtime, Edge Functions and browser SDK calls need their own decisions. A successful SQL import does not prove the application still works.

| Resource | Use it for |
| --- | --- |
| [Migration guide](guides/supabase-to-neon.md) | Understand the trade-offs, choose a strategy, rehearse, validate and prepare a cutover. |
| [Agent skill](skills/supabase-to-neon/SKILL.md) | Give a coding assistant the migration workflow and its boundaries. |
| [Migration plan template](templates/supabase-to-neon-plan.md) | Record the actual source, destination, dependencies, acceptance checks and recovery plan. |

## Use the skill with your AI

Clone this repository and give your coding assistant a prompt like:

```text
Read skills/supabase-to-neon/SKILL.md from this repository.
Audit my application at <application path> in read-only mode.
Identify every Supabase dependency, what can stay, what must change,
and the smallest migration that preserves users, data and permissions.
Produce a plan using templates/supabase-to-neon-plan.md.
Do not access credentials, create cloud resources, run a migration,
deploy or change production during this audit.
```

When implementation is authorized, use the resulting plan to define the exact environments, allowed operations, budget and acceptance criteria. The skill preserves authorization already given; it does not grant itself production access.

## What this includes

- Database-only migration and full-backend replacement paths.
- A reviewed dump/restore rehearsal example without inline passwords.
- Auth, stable user IDs, RLS, grants and tenant isolation checks.
- File migration, background jobs and external side-effect considerations.
- Downtime, replication, cost and rollback trade-offs.

This is a documentation and agent-instruction toolkit. The commands are templates to review for your application. It does not contain an automatic migration executable, customer data, database dumps or credentials, and it has not been verified against every Supabase configuration.

## Repository map

```text
guides/       Human-readable workflows
skills/       Instructions and references for coding assistants
templates/    Reusable planning documents
```

Provider capabilities and prices change. Check the official documentation linked in the guide before executing a migration. Documentation reviewed: **2026-10-24**.
