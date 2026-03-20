# Supabase / BaaS Patterns

Supabase provides Postgres database, auth, storage, realtime, and edge functions in one platform. The go-to backend for rapid prototyping and vibe coding.

## Setup

```bash
npm install @supabase/supabase-js
npx supabase init           # Local dev setup
npx supabase start          # Start local containers
npx supabase db reset        # Reset local DB + run migrations
```

```typescript
// lib/supabase/client.ts — Browser client
import { createBrowserClient } from "@supabase/ssr";

export function createClient() {
  return createBrowserClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
  );
}

// lib/supabase/server.ts — Server client (Next.js App Router)
import { createServerClient } from "@supabase/ssr";
import { cookies } from "next/headers";

export async function createClient() {
  const cookieStore = await cookies();
  return createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll() { return cookieStore.getAll(); },
        setAll(cookiesToSet) {
          cookiesToSet.forEach(({ name, value, options }) => {
            cookieStore.set(name, value, options);
          });
        },
      },
    }
  );
}

// lib/supabase/admin.ts — Service role (server-side only, bypasses RLS)
import { createClient } from "@supabase/supabase-js";

export const supabaseAdmin = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY! // NEVER expose this client-side
);
```

## Authentication

```typescript
// Sign up
const { data, error } = await supabase.auth.signUp({
  email: "user@example.com",
  password: "securepassword",
});

// Sign in
const { data, error } = await supabase.auth.signInWithPassword({
  email: "user@example.com",
  password: "securepassword",
});

// OAuth (Google, GitHub, etc.)
const { data, error } = await supabase.auth.signInWithOAuth({
  provider: "google",
  options: { redirectTo: `${origin}/auth/callback` },
});

// Auth callback route (Next.js App Router)
// app/auth/callback/route.ts
import { createClient } from "@/lib/supabase/server";
import { NextResponse } from "next/server";

export async function GET(request: Request) {
  const { searchParams, origin } = new URL(request.url);
  const code = searchParams.get("code");

  if (code) {
    const supabase = await createClient();
    const { error } = await supabase.auth.exchangeCodeForSession(code);
    if (!error) return NextResponse.redirect(`${origin}/dashboard`);
  }

  return NextResponse.redirect(`${origin}/auth/error`);
}

// Get current user (server component)
const supabase = await createClient();
const { data: { user } } = await supabase.auth.getUser();

// Sign out
await supabase.auth.signOut();

// Middleware for protected routes
// middleware.ts
import { createServerClient } from "@supabase/ssr";
import { NextResponse, type NextRequest } from "next/server";

export async function middleware(request: NextRequest) {
  let supabaseResponse = NextResponse.next({ request });
  const supabase = createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll() { return request.cookies.getAll(); },
        setAll(cookiesToSet) {
          cookiesToSet.forEach(({ name, value, options }) => {
            request.cookies.set(name, value);
            supabaseResponse.cookies.set(name, value, options);
          });
        },
      },
    }
  );
  const { data: { user } } = await supabase.auth.getUser();

  if (!user && request.nextUrl.pathname.startsWith("/dashboard")) {
    return NextResponse.redirect(new URL("/login", request.url));
  }

  return supabaseResponse;
}

export const config = { matcher: ["/dashboard/:path*", "/api/:path*"] };
```

## Database Queries

```typescript
// Select
const { data: posts, error } = await supabase
  .from("posts")
  .select("id, title, content, author:users(name, avatar_url)")
  .eq("status", "published")
  .order("created_at", { ascending: false })
  .range(0, 9); // Pagination: first 10 rows

// Insert
const { data, error } = await supabase
  .from("posts")
  .insert({ title: "New Post", content: "Hello", author_id: user.id })
  .select()
  .single();

// Update
const { data, error } = await supabase
  .from("posts")
  .update({ status: "published", published_at: new Date().toISOString() })
  .eq("id", postId)
  .select()
  .single();

// Delete
const { error } = await supabase.from("posts").delete().eq("id", postId);

// RPC (call Postgres functions)
const { data, error } = await supabase.rpc("search_posts", {
  search_query: "hello world",
});
```

## Row Level Security (RLS)

```sql
-- Enable RLS on a table
ALTER TABLE posts ENABLE ROW LEVEL SECURITY;

-- Users can read published posts
CREATE POLICY "Anyone can read published posts" ON posts
  FOR SELECT USING (status = 'published');

-- Users can read their own drafts
CREATE POLICY "Users can read own drafts" ON posts
  FOR SELECT USING (auth.uid() = author_id);

-- Users can insert their own posts
CREATE POLICY "Users can create posts" ON posts
  FOR INSERT WITH CHECK (auth.uid() = author_id);

-- Users can update their own posts
CREATE POLICY "Users can update own posts" ON posts
  FOR UPDATE USING (auth.uid() = author_id);

-- Users can delete their own posts
CREATE POLICY "Users can delete own posts" ON posts
  FOR DELETE USING (auth.uid() = author_id);

-- Admin can do anything
CREATE POLICY "Admins have full access" ON posts
  FOR ALL USING (
    EXISTS (SELECT 1 FROM users WHERE id = auth.uid() AND role = 'admin')
  );
```

## Storage

```typescript
// Upload file
const { data, error } = await supabase.storage
  .from("avatars")
  .upload(`${userId}/avatar.png`, file, {
    cacheControl: "3600",
    upsert: true,
    contentType: file.type,
  });

// Get public URL
const { data: { publicUrl } } = supabase.storage
  .from("avatars")
  .getPublicUrl(`${userId}/avatar.png`);

// Download
const { data, error } = await supabase.storage
  .from("documents")
  .download("report.pdf");

// List files
const { data, error } = await supabase.storage
  .from("uploads")
  .list(userId, { limit: 100, sortBy: { column: "created_at", order: "desc" } });

// Delete
const { error } = await supabase.storage
  .from("avatars")
  .remove([`${userId}/avatar.png`]);
```

Storage RLS:
```sql
-- Storage policies (in storage.objects table)
CREATE POLICY "Users can upload own avatar" ON storage.objects
  FOR INSERT WITH CHECK (
    bucket_id = 'avatars' AND
    auth.uid()::text = (storage.foldername(name))[1]
  );

CREATE POLICY "Avatars are publicly readable" ON storage.objects
  FOR SELECT USING (bucket_id = 'avatars');
```

## Realtime

```typescript
// Subscribe to changes
const channel = supabase
  .channel("posts-changes")
  .on(
    "postgres_changes",
    { event: "*", schema: "public", table: "posts", filter: `author_id=eq.${userId}` },
    (payload) => {
      console.log("Change:", payload.eventType, payload.new);
      // Update local state
    }
  )
  .subscribe();

// Presence (who's online)
const room = supabase.channel("room-1");
room
  .on("presence", { event: "sync" }, () => {
    const state = room.presenceState();
    console.log("Online:", Object.keys(state));
  })
  .subscribe(async (status) => {
    if (status === "SUBSCRIBED") {
      await room.track({ user_id: userId, name: userName });
    }
  });

// Cleanup
supabase.removeChannel(channel);
```

## Edge Functions

```bash
npx supabase functions new my-function
npx supabase functions serve   # Local dev
npx supabase functions deploy my-function
```

```typescript
// supabase/functions/my-function/index.ts
import { serve } from "https://deno.land/std@0.168.0/http/server.ts";
import { createClient } from "https://esm.sh/@supabase/supabase-js@2";

serve(async (req) => {
  const supabase = createClient(
    Deno.env.get("SUPABASE_URL")!,
    Deno.env.get("SUPABASE_SERVICE_ROLE_KEY")!
  );

  const { data, error } = await supabase.from("posts").select("*").limit(10);
  return new Response(JSON.stringify(data), {
    headers: { "Content-Type": "application/json" },
  });
});
```

## Migrations

```bash
npx supabase migration new create-posts-table

# Creates: supabase/migrations/20240101000000_create-posts-table.sql
# Write your SQL, then:
npx supabase db reset  # Apply locally
npx supabase db push   # Push to remote (⚠️ review first)
```

## Type Generation

```bash
npx supabase gen types typescript --local > src/types/database.ts
# or from remote:
npx supabase gen types typescript --project-id $PROJECT_ID > src/types/database.ts
```

```typescript
// Use generated types
import { Database } from "@/types/database";

type Post = Database["public"]["Tables"]["posts"]["Row"];
type InsertPost = Database["public"]["Tables"]["posts"]["Insert"];
type UpdatePost = Database["public"]["Tables"]["posts"]["Update"];

// Typed client
const supabase = createClient<Database>(url, key);
```

## Supabase + Stripe Pattern

```typescript
// Webhook handler for Stripe events
// app/api/webhooks/stripe/route.ts
export async function POST(req: Request) {
  const body = await req.text();
  const sig = req.headers.get("stripe-signature")!;
  const event = stripe.webhooks.constructEvent(body, sig, webhookSecret);

  switch (event.type) {
    case "checkout.session.completed": {
      const session = event.data.object;
      await supabaseAdmin
        .from("subscriptions")
        .upsert({
          user_id: session.metadata.userId,
          stripe_customer_id: session.customer,
          stripe_subscription_id: session.subscription,
          status: "active",
          plan: session.metadata.plan,
        });
      break;
    }
    case "customer.subscription.deleted": {
      const subscription = event.data.object;
      await supabaseAdmin
        .from("subscriptions")
        .update({ status: "canceled" })
        .eq("stripe_subscription_id", subscription.id);
      break;
    }
  }

  return new Response("OK");
}
```
