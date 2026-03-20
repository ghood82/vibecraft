# Drizzle ORM Patterns

## Config Setup
```typescript
// drizzle.config.ts
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  schema: "./src/db/schema.ts",
  out: "./drizzle",
  dialect: "postgresql", // or "sqlite" | "mysql"
  dbCredentials: {
    url: process.env.DATABASE_URL!,
  },
});
```

## Schema Definition (Postgres)
```typescript
// src/db/schema.ts
import {
  pgTable, text, integer, timestamp, boolean, uuid, varchar,
  primaryKey, index, uniqueIndex, serial, jsonb, pgEnum,
} from "drizzle-orm/pg-core";
import { relations } from "drizzle-orm";

// Enums
export const roleEnum = pgEnum("role", ["admin", "member", "viewer"]);
export const statusEnum = pgEnum("status", ["draft", "published", "archived"]);

// Users
export const users = pgTable("users", {
  id: uuid("id").defaultRandom().primaryKey(),
  email: varchar("email", { length: 255 }).notNull().unique(),
  name: varchar("name", { length: 255 }).notNull(),
  role: roleEnum("role").default("member").notNull(),
  avatarUrl: text("avatar_url"),
  createdAt: timestamp("created_at").defaultNow().notNull(),
  updatedAt: timestamp("updated_at").defaultNow().notNull(),
}, (table) => ({
  emailIdx: uniqueIndex("users_email_idx").on(table.email),
}));

// Posts
export const posts = pgTable("posts", {
  id: uuid("id").defaultRandom().primaryKey(),
  title: varchar("title", { length: 500 }).notNull(),
  slug: varchar("slug", { length: 500 }).notNull().unique(),
  content: text("content"),
  status: statusEnum("status").default("draft").notNull(),
  authorId: uuid("author_id").notNull().references(() => users.id, { onDelete: "cascade" }),
  metadata: jsonb("metadata").$type<{ tags: string[]; readTime: number }>(),
  publishedAt: timestamp("published_at"),
  createdAt: timestamp("created_at").defaultNow().notNull(),
  updatedAt: timestamp("updated_at").defaultNow().notNull(),
}, (table) => ({
  authorIdx: index("posts_author_idx").on(table.authorId),
  statusIdx: index("posts_status_idx").on(table.status),
  slugIdx: uniqueIndex("posts_slug_idx").on(table.slug),
}));

// Comments
export const comments = pgTable("comments", {
  id: uuid("id").defaultRandom().primaryKey(),
  body: text("body").notNull(),
  postId: uuid("post_id").notNull().references(() => posts.id, { onDelete: "cascade" }),
  authorId: uuid("author_id").notNull().references(() => users.id, { onDelete: "cascade" }),
  parentId: uuid("parent_id"),
  createdAt: timestamp("created_at").defaultNow().notNull(),
}, (table) => ({
  postIdx: index("comments_post_idx").on(table.postId),
}));

// Relations (for query builder)
export const usersRelations = relations(users, ({ many }) => ({
  posts: many(posts),
  comments: many(comments),
}));

export const postsRelations = relations(posts, ({ one, many }) => ({
  author: one(users, { fields: [posts.authorId], references: [users.id] }),
  comments: many(comments),
}));

export const commentsRelations = relations(comments, ({ one }) => ({
  post: one(posts, { fields: [comments.postId], references: [posts.id] }),
  author: one(users, { fields: [comments.authorId], references: [users.id] }),
  parent: one(comments, { fields: [comments.parentId], references: [comments.id] }),
}));
```

## Schema Definition (SQLite / D1)
```typescript
import { sqliteTable, text, integer } from "drizzle-orm/sqlite-core";

export const users = sqliteTable("users", {
  id: integer("id").primaryKey({ autoIncrement: true }),
  email: text("email").notNull().unique(),
  name: text("name").notNull(),
  createdAt: text("created_at").default(sql`CURRENT_TIMESTAMP`),
});
```

## Database Client Setup
```typescript
// Postgres (Neon serverless)
import { neon } from "@neondatabase/serverless";
import { drizzle } from "drizzle-orm/neon-http";
import * as schema from "./schema";

const sql = neon(process.env.DATABASE_URL!);
export const db = drizzle(sql, { schema });

// SQLite (Cloudflare D1)
import { drizzle } from "drizzle-orm/d1";
import * as schema from "./schema";

export function getDb(d1: D1Database) {
  return drizzle(d1, { schema });
}

// Postgres (node-postgres)
import { Pool } from "pg";
import { drizzle } from "drizzle-orm/node-postgres";
import * as schema from "./schema";

const pool = new Pool({ connectionString: process.env.DATABASE_URL });
export const db = drizzle(pool, { schema });
```

## Common Queries
```typescript
import { eq, and, gt, like, desc, sql, count } from "drizzle-orm";

// Basic CRUD
const user = await db.query.users.findFirst({
  where: eq(users.email, "alice@example.com"),
});

const allPosts = await db.query.posts.findMany({
  where: eq(posts.status, "published"),
  orderBy: desc(posts.publishedAt),
  limit: 10,
  with: { author: true, comments: { limit: 5 } },
});

// Insert
const [newUser] = await db.insert(users).values({
  email: "bob@example.com",
  name: "Bob",
}).returning();

// Update
await db.update(posts)
  .set({ status: "published", publishedAt: new Date() })
  .where(eq(posts.id, postId));

// Delete
await db.delete(comments).where(eq(comments.id, commentId));

// Complex query
const stats = await db
  .select({
    authorName: users.name,
    postCount: count(posts.id),
  })
  .from(users)
  .leftJoin(posts, eq(users.id, posts.authorId))
  .groupBy(users.name)
  .orderBy(desc(count(posts.id)));

// Transactions
await db.transaction(async (tx) => {
  const [order] = await tx.insert(orders).values({ userId, total }).returning();
  await tx.insert(orderItems).values(
    items.map(item => ({ orderId: order.id, ...item }))
  );
  await tx.update(users).set({ orderCount: sql`order_count + 1` }).where(eq(users.id, userId));
});
```

## Seeding
```typescript
// src/db/seed.ts
import { db } from "./index";
import { users, posts } from "./schema";

async function seed() {
  console.log("Seeding database...");
  await db.delete(posts);
  await db.delete(users);

  const [alice, bob] = await db.insert(users).values([
    { email: "alice@example.com", name: "Alice" },
    { email: "bob@example.com", name: "Bob" },
  ]).returning();

  await db.insert(posts).values([
    { title: "First Post", slug: "first-post", content: "Hello world", authorId: alice.id, status: "published" },
    { title: "Draft Post", slug: "draft-post", content: "WIP", authorId: bob.id },
  ]);

  console.log("Seeded successfully!");
}

seed().catch(console.error);
```

```json
// package.json
{ "scripts": { "db:seed": "npx tsx src/db/seed.ts" } }
```

## Prisma Alternative
```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(uuid())
  email     String   @unique
  name      String
  posts     Post[]
  createdAt DateTime @default(now()) @map("created_at")
  @@map("users")
}

model Post {
  id        String   @id @default(uuid())
  title     String
  content   String?
  author    User     @relation(fields: [authorId], references: [id])
  authorId  String   @map("author_id")
  createdAt DateTime @default(now()) @map("created_at")
  @@index([authorId])
  @@map("posts")
}
```
