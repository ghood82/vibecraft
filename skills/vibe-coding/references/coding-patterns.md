# Coding Patterns

Production-ready patterns with code examples. Reference this when generating code to ensure consistency and quality.

## React Component Patterns

### Server Component (Default in Next.js)
```tsx
// app/users/page.tsx — Server component, runs on server only
import { db } from "@/lib/db";

export default async function UsersPage() {
  const users = await db.query.users.findMany();

  return (
    <main className="container mx-auto py-8">
      <h1 className="text-2xl font-bold mb-4">Users</h1>
      <UserList users={users} />
    </main>
  );
}
```

### Client Component (Interactive)
```tsx
"use client";
// Only add "use client" when you need: useState, useEffect, event handlers, browser APIs

import { useState } from "react";

export function SearchFilter({ onSearch }: { onSearch: (q: string) => void }) {
  const [query, setQuery] = useState("");

  return (
    <input
      value={query}
      onChange={(e) => {
        setQuery(e.target.value);
        onSearch(e.target.value);
      }}
      placeholder="Search..."
      className="border rounded px-3 py-2"
    />
  );
}
```

### Composition Pattern
```tsx
// Compose complex UIs from simple, focused components
function DashboardPage() {
  return (
    <PageLayout>
      <PageHeader title="Dashboard" actions={<CreateButton />} />
      <div className="grid grid-cols-3 gap-4">
        <MetricCard title="Revenue" value="$12,345" trend="+12%" />
        <MetricCard title="Users" value="1,234" trend="+5%" />
        <MetricCard title="Orders" value="567" trend="-2%" />
      </div>
      <RecentActivity />
    </PageLayout>
  );
}
```

## API Design Patterns

### Next.js Server Actions
```tsx
"use server";

import { db } from "@/lib/db";
import { z } from "zod";
import { revalidatePath } from "next/cache";

const createUserSchema = z.object({
  name: z.string().min(1).max(100),
  email: z.string().email(),
});

export async function createUser(formData: FormData) {
  const parsed = createUserSchema.safeParse({
    name: formData.get("name"),
    email: formData.get("email"),
  });

  if (!parsed.success) {
    return { error: parsed.error.flatten().fieldErrors };
  }

  await db.insert(users).values(parsed.data);
  revalidatePath("/users");
  return { success: true };
}
```

### Hono API Route
```typescript
import { Hono } from "hono";
import { zValidator } from "@hono/zod-validator";
import { z } from "zod";

const app = new Hono();

const createUserSchema = z.object({
  name: z.string().min(1),
  email: z.string().email(),
});

app.post("/users", zValidator("json", createUserSchema), async (c) => {
  const data = c.req.valid("json");
  const user = await db.insert(users).values(data).returning();
  return c.json(user, 201);
});

app.onError((err, c) => {
  console.error(err);
  return c.json({ error: "Internal server error" }, 500);
});
```

## State Management

### Zustand Store
```typescript
import { create } from "zustand";
import { persist } from "zustand/middleware";

interface AppState {
  theme: "light" | "dark";
  sidebarOpen: boolean;
  toggleTheme: () => void;
  toggleSidebar: () => void;
}

export const useAppStore = create<AppState>()(
  persist(
    (set) => ({
      theme: "light",
      sidebarOpen: true,
      toggleTheme: () =>
        set((state) => ({ theme: state.theme === "light" ? "dark" : "light" })),
      toggleSidebar: () =>
        set((state) => ({ sidebarOpen: !state.sidebarOpen })),
    }),
    { name: "app-settings" }
  )
);
```

### TanStack Query
```typescript
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";

export function useUsers() {
  return useQuery({
    queryKey: ["users"],
    queryFn: () => fetch("/api/users").then((r) => r.json()),
  });
}

export function useCreateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (data: CreateUserInput) =>
      fetch("/api/users", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(data),
      }).then((r) => r.json()),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["users"] });
    },
  });
}
```

## Error Handling

### Result Type Pattern
```typescript
type Result<T, E = Error> =
  | { success: true; data: T }
  | { success: false; error: E };

async function fetchUser(id: string): Promise<Result<User>> {
  try {
    const user = await db.query.users.findFirst({ where: eq(users.id, id) });
    if (!user) return { success: false, error: new Error("User not found") };
    return { success: true, data: user };
  } catch (err) {
    return { success: false, error: err as Error };
  }
}

// Usage: forces caller to handle both cases
const result = await fetchUser("123");
if (!result.success) {
  console.error(result.error);
  return;
}
const user = result.data; // TypeScript knows this is User
```

### Error Boundary
```tsx
"use client";

import { Component, type ReactNode } from "react";

interface Props { children: ReactNode; fallback?: ReactNode; }
interface State { hasError: boolean; error?: Error; }

export class ErrorBoundary extends Component<Props, State> {
  state: State = { hasError: false };

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback ?? (
        <div className="p-4 bg-red-50 text-red-800 rounded">
          <h2 className="font-bold">Something went wrong</h2>
          <p className="text-sm">{this.state.error?.message}</p>
        </div>
      );
    }
    return this.props.children;
  }
}
```

## Database Patterns (Drizzle)

### Schema Definition
```typescript
import { sqliteTable, text, integer } from "drizzle-orm/sqlite-core";

export const users = sqliteTable("users", {
  id: text("id").primaryKey().$defaultFn(() => crypto.randomUUID()),
  name: text("name").notNull(),
  email: text("email").notNull().unique(),
  role: text("role", { enum: ["admin", "user"] }).default("user"),
  createdAt: integer("created_at", { mode: "timestamp" }).$defaultFn(() => new Date()),
});

export const posts = sqliteTable("posts", {
  id: text("id").primaryKey().$defaultFn(() => crypto.randomUUID()),
  title: text("title").notNull(),
  content: text("content"),
  authorId: text("author_id").references(() => users.id),
  published: integer("published", { mode: "boolean" }).default(false),
});
```

### Type-Safe Queries
```typescript
import { db } from "./index";
import { eq, desc, like } from "drizzle-orm";
import { users, posts } from "./schema";

// Select with relations
const usersWithPosts = await db.query.users.findMany({
  with: { posts: { where: eq(posts.published, true) } },
  orderBy: desc(users.createdAt),
  limit: 20,
});

// Insert
const [newUser] = await db.insert(users).values({ name: "Alice", email: "alice@example.com" }).returning();

// Update
await db.update(users).set({ role: "admin" }).where(eq(users.id, userId));

// Delete
await db.delete(users).where(eq(users.id, userId));
```

## Authentication Patterns

### JWT + Cookie Auth (Next.js)
```typescript
import { SignJWT, jwtVerify } from "jose";
import { cookies } from "next/headers";

const secret = new TextEncoder().encode(process.env.JWT_SECRET);

export async function createSession(userId: string) {
  const token = await new SignJWT({ userId })
    .setProtectedHeader({ alg: "HS256" })
    .setExpirationTime("7d")
    .sign(secret);

  cookies().set("session", token, {
    httpOnly: true,
    secure: process.env.NODE_ENV === "production",
    sameSite: "lax",
    maxAge: 60 * 60 * 24 * 7,
  });
}

export async function getSession() {
  const token = cookies().get("session")?.value;
  if (!token) return null;

  try {
    const { payload } = await jwtVerify(token, secret);
    return payload as { userId: string };
  } catch {
    return null;
  }
}
```

### Middleware Auth Guard
```typescript
// middleware.ts
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

const publicPaths = ["/", "/login", "/signup", "/api/auth"];

export function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl;
  if (publicPaths.some((p) => pathname.startsWith(p))) return NextResponse.next();

  const session = request.cookies.get("session");
  if (!session) {
    return NextResponse.redirect(new URL("/login", request.url));
  }

  return NextResponse.next();
}

export const config = { matcher: ["/((?!_next/static|_next/image|favicon.ico).*)"] };
```

## Real-Time Patterns

### Server-Sent Events (SSE)
```typescript
// API route (Next.js or Hono)
export async function GET() {
  const encoder = new TextEncoder();
  const stream = new ReadableStream({
    start(controller) {
      const interval = setInterval(() => {
        const data = JSON.stringify({ time: Date.now(), value: Math.random() });
        controller.enqueue(encoder.encode(`data: ${data}\n\n`));
      }, 1000);

      // Cleanup on close
      return () => clearInterval(interval);
    },
  });

  return new Response(stream, {
    headers: {
      "Content-Type": "text/event-stream",
      "Cache-Control": "no-cache",
      Connection: "keep-alive",
    },
  });
}

// Client hook
function useSSE<T>(url: string) {
  const [data, setData] = useState<T | null>(null);

  useEffect(() => {
    const source = new EventSource(url);
    source.onmessage = (e) => setData(JSON.parse(e.data));
    return () => source.close();
  }, [url]);

  return data;
}
```
