---
name: database
description: "Database migrations, schema design, ORM setup (Drizzle, Prisma, D1). Use for database work. Do NOT use for API design, deployment, or Supabase auth."
---

# Database Migrations & Schema Management

Safe schema changes, ORM setup, data modeling, and migration workflows. Covers Drizzle ORM (preferred), Prisma, and raw SQL.

## Drizzle ORM (Recommended)

```bash
npm install drizzle-orm && npm install -D drizzle-kit
```

### Migration Workflow
```
1. Edit schema.ts (add/modify tables, columns, indexes)
2. Run `npx drizzle-kit generate` → creates SQL migration
3. Review the generated SQL (ALWAYS review before applying!)
4. Run `npx drizzle-kit migrate` → applies to database
5. Commit both schema.ts changes AND migration files
```

### Key Commands
```bash
npx drizzle-kit generate    # Generate migration from schema changes
npx drizzle-kit push        # Push schema directly (dev only)
npx drizzle-kit migrate     # Apply pending migrations
npx drizzle-kit studio      # Visual DB browser
npx drizzle-kit drop        # Drop a migration (before applying)
```

## Cloudflare D1 Migrations
```bash
npx wrangler d1 migrations create my-db add-users-table
npx wrangler d1 migrations apply my-db --local    # Local
npx wrangler d1 migrations apply my-db --remote   # Production
```

## Prisma (Alternative)
```bash
npx prisma migrate dev --name add-posts    # Dev migration
npx prisma migrate deploy                   # Production migration
npx prisma generate                         # Generate client
npx prisma studio                           # Visual browser
npx prisma db seed                          # Run seed script
```

## Schema Design Rules

1. **Always use UUIDs** for public-facing IDs (not auto-increment integers)
2. **Always add `createdAt` and `updatedAt`** timestamps
3. **Always add indexes** on foreign keys and frequently filtered columns
4. **Use enums** for columns with fixed value sets
5. **Use `onDelete: "cascade"`** for child records
6. **Soft delete** (add `deletedAt`) for recoverable data
7. **Name tables plural** (`users`, `posts`), columns singular (`email`, `name`)

## Migration Safety Checklist

- [ ] Reviewed generated SQL before applying
- [ ] Tested migration locally first
- [ ] No data loss (DROP COLUMN only if backed up or unused)
- [ ] New columns have defaults (or are nullable)
- [ ] Indexes added for new foreign keys
- [ ] Rollback plan exists
- [ ] Migration is backwards-compatible with running code
- [ ] Seed data updated for new schema

## Reference Docs

Read `references/drizzle-patterns.md` for schema definitions, client setup, queries, relations, transactions, and seeding.
Read `references/supabase-patterns.md` for Supabase auth, RLS, storage, realtime, and edge functions.
