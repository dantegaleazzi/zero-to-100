# Migration notes

Read the current official documentation before executing. Documentation reviewed: 2026-10-24.

## Official references

- [Neon documentation index](https://neon.com/docs/llms.txt) and [migration index](https://neon.com/docs/import/llms.txt): discover current compatibility, connection, replication and region guidance.
- [Supabase database migration to Neon](https://neon.com/docs/import/migrate-from-supabase): database export/restore; the example is not a complete backend migration.
- [PostgreSQL pg_dump](https://www.postgresql.org/docs/current/app-pgdump.html), [pg_restore](https://www.postgresql.org/docs/current/app-pgrestore.html) and [service files](https://www.postgresql.org/docs/current/libpq-pgservice.html): review compatible clients, private connections, schema coverage and restore behavior.
- [Logical replication restrictions](https://www.postgresql.org/docs/current/logical-replication-restrictions.html): account for DDL, sequence state and large objects separately.
- [Supabase users](https://supabase.com/docs/guides/auth/users), [RLS](https://supabase.com/docs/guides/database/postgres/row-level-security) and [Storage schema](https://supabase.com/docs/guides/storage/schema/design): preserve identities, access semantics and actual file bytes.
- [Supabase changelog](https://supabase.com/changelog), [Neon pricing](https://neon.com/pricing) and [usage calculations](https://neon.com/docs/introduction/usage-calculations): recheck current behavior and costs.

## Acceptance scenarios

- **Database only, Auth retained:** verify existing JWTs, stable subject IDs, user lifecycle synchronization and account A/B isolation against the new database. Record retained Supabase services.
- **Full backend replacement:** give every used service an owner, implementation and test. Include file bytes, auth and jobs; SQL restore alone is insufficient.
- **Fresh backend with reused UI:** validate requested behavior and seed provenance. State explicitly which historical users/data were not transferred.

For a worked command template and planning worksheet, see the repository's `guides/supabase-to-neon.md` and `templates/supabase-to-neon-plan.md`. Those are human-facing resources; the skill remains usable independently with these references.
