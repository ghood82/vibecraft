# Advanced Performance Patterns

## Font Optimization

```html
<!-- Preload critical fonts -->
<link rel="preload" href="/fonts/inter-var.woff2" as="font" type="font/woff2" crossorigin />

<!-- Font-display swap to prevent FOIT -->
<style>
@font-face {
  font-family: 'Inter';
  src: url('/fonts/inter-var.woff2') format('woff2');
  font-display: swap;
  font-weight: 100 900;
}
</style>
```

```typescript
// Next.js font optimization (automatic)
import { Inter } from "next/font/google";
const inter = Inter({ subsets: ["latin"], display: "swap" });

// Self-hosted with next/font
import localFont from "next/font/local";
const myFont = localFont({ src: "./fonts/MyFont.woff2" });
```

## Prefetching & Preloading

```typescript
// Next.js Link automatically prefetches on hover/viewport
import Link from "next/link";
<Link href="/dashboard">Dashboard</Link>

// Prefetch data for likely next navigation
function ProductList() {
  const queryClient = useQueryClient();

  function handleHover(productId: string) {
    queryClient.prefetchQuery({
      queryKey: ["product", productId],
      queryFn: () => fetchProduct(productId),
      staleTime: 60_000,
    });
  }

  return products.map(p => (
    <Link href={`/products/${p.id}`} onMouseEnter={() => handleHover(p.id)}>
      {p.name}
    </Link>
  ));
}

// DNS prefetch for external domains
<link rel="dns-prefetch" href="https://api.stripe.com" />
<link rel="preconnect" href="https://fonts.googleapis.com" crossorigin />
```

## Service Worker Caching

```typescript
// sw.js — Cache-first for static, network-first for API
const STATIC_CACHE = "static-v1";
const API_CACHE = "api-v1";

self.addEventListener("fetch", (event) => {
  const { request } = event;
  const url = new URL(request.url);

  if (url.pathname.startsWith("/api/")) {
    // Network-first for API calls
    event.respondWith(
      fetch(request)
        .then(response => {
          const clone = response.clone();
          caches.open(API_CACHE).then(cache => cache.put(request, clone));
          return response;
        })
        .catch(() => caches.match(request))
    );
  } else {
    // Cache-first for static assets
    event.respondWith(
      caches.match(request).then(cached => cached || fetch(request))
    );
  }
});
```

## Streaming & Suspense (React 18+)

```typescript
// Stream HTML with Suspense boundaries
// app/dashboard/page.tsx (Next.js App Router)
import { Suspense } from "react";

export default function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>
      {/* This renders immediately */}
      <Suspense fallback={<StatsSkeleton />}>
        <StatsPanel />  {/* Async component, streams when ready */}
      </Suspense>
      <Suspense fallback={<ChartSkeleton />}>
        <RevenueChart /> {/* Streams independently */}
      </Suspense>
    </div>
  );
}

// Async server component
async function StatsPanel() {
  const stats = await fetchStats(); // Doesn't block the page
  return <div>{stats.revenue}</div>;
}
```

## Web Workers for Heavy Computation

```typescript
// worker.ts
self.onmessage = (event) => {
  const { data, type } = event.data;

  switch (type) {
    case "sort":
      const sorted = data.sort((a, b) => a.value - b.value);
      self.postMessage({ type: "sorted", data: sorted });
      break;
    case "filter":
      const filtered = data.filter(item => item.active);
      self.postMessage({ type: "filtered", data: filtered });
      break;
  }
};

// main.ts — use from React
function useWorker() {
  const workerRef = useRef<Worker>();

  useEffect(() => {
    workerRef.current = new Worker(new URL("./worker.ts", import.meta.url));
    return () => workerRef.current?.terminate();
  }, []);

  const sortInWorker = useCallback((data: Item[]) => {
    return new Promise<Item[]>((resolve) => {
      workerRef.current!.onmessage = (e) => resolve(e.data.data);
      workerRef.current!.postMessage({ type: "sort", data });
    });
  }, []);

  return { sortInWorker };
}
```

## Optimistic Updates

```typescript
// TanStack Query optimistic update
const mutation = useMutation({
  mutationFn: updateTodo,
  onMutate: async (newTodo) => {
    await queryClient.cancelQueries({ queryKey: ["todos"] });
    const previous = queryClient.getQueryData(["todos"]);
    queryClient.setQueryData(["todos"], (old: Todo[]) =>
      old.map(t => t.id === newTodo.id ? { ...t, ...newTodo } : t)
    );
    return { previous };
  },
  onError: (err, newTodo, context) => {
    queryClient.setQueryData(["todos"], context?.previous); // Rollback
  },
  onSettled: () => {
    queryClient.invalidateQueries({ queryKey: ["todos"] }); // Refetch truth
  },
});
```

## Monitoring Performance in Production

```typescript
// Report Core Web Vitals
import { onCLS, onINP, onLCP } from "web-vitals";

function sendMetric(metric: { name: string; value: number; id: string }) {
  fetch("/api/vitals", {
    method: "POST",
    body: JSON.stringify(metric),
    keepalive: true, // Ensure delivery even on page unload
  });
}

onCLS(sendMetric);
onINP(sendMetric);
onLCP(sendMetric);

// Next.js built-in
// app/layout.tsx
export function reportWebVitals(metric: NextWebVitalsMetric) {
  console.log(metric); // Or send to analytics
}
```

## Database Advanced Patterns

### Materialized Views for Complex Queries
```sql
-- Pre-compute expensive aggregations
CREATE MATERIALIZED VIEW daily_revenue AS
SELECT
  date_trunc('day', created_at) AS day,
  SUM(amount) AS total,
  COUNT(*) AS order_count,
  AVG(amount) AS avg_order
FROM orders
WHERE status = 'completed'
GROUP BY 1;

-- Refresh on schedule
REFRESH MATERIALIZED VIEW CONCURRENTLY daily_revenue;
```

### Query Caching with Redis
```typescript
import { Redis } from "@upstash/redis";
const redis = Redis.fromEnv();

async function getCachedQuery<T>(
  key: string,
  queryFn: () => Promise<T>,
  ttl = 300
): Promise<T> {
  const cached = await redis.get<T>(key);
  if (cached) return cached;

  const result = await queryFn();
  await redis.set(key, result, { ex: ttl });
  return result;
}

// Usage
const stats = await getCachedQuery(
  `dashboard:stats:${userId}`,
  () => db.select().from(dashboardStats).where(eq(dashboardStats.userId, userId)),
  60
);
```

## Performance Checklist

### Before Launch
- [ ] Lighthouse score > 90 on mobile
- [ ] LCP < 2.5s on 3G throttled
- [ ] INP < 200ms
- [ ] CLS < 0.1
- [ ] Initial JS bundle < 100KB gzipped
- [ ] Images in WebP/AVIF with proper sizing
- [ ] Fonts preloaded with display:swap
- [ ] Third-party scripts loaded async/defer
- [ ] No layout shifts from dynamic content
- [ ] Database queries have appropriate indexes

### Ongoing
- [ ] Monitor Core Web Vitals in production (web-vitals library)
- [ ] Bundle size check in CI
- [ ] Lighthouse CI on PRs
- [ ] Database slow query log review (weekly)
- [ ] CDN cache hit rate > 90%
