# Suspense & Lazy Loading

## Concept
`React.lazy()` splits your bundle so a component's code is only downloaded
when it's actually needed. `<Suspense>` lets you declaratively show a
fallback (like a spinner) while that component (or any async operation
it supports) is loading, instead of manually tracking loading state.

## Why it matters
Sending your entire app as one giant JS bundle means users download code for
screens they may never visit. Code-splitting with `lazy` + `Suspense` is a
standard, low-effort performance win — directly relevant to bundle-size and
load-time questions in interviews.

## How it works

**Basic code-splitting:**

```jsx
import { lazy, Suspense } from "react";

// Instead of a normal top-level import, this becomes a separate chunk,
// only fetched when Settings actually needs to render
const Settings = lazy(() => import("./Settings"));

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <Settings />
    </Suspense>
  );
}
```

**Route-based splitting — the most common real-world pattern:**

```jsx
import { lazy, Suspense } from "react";
import { Routes, Route } from "react-router-dom";

const Dashboard = lazy(() => import("./pages/Dashboard"));
const Profile = lazy(() => import("./pages/Profile"));

function App() {
  return (
    <Suspense fallback={<PageLoader />}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/profile" element={<Profile />} />
      </Routes>
    </Suspense>
  );
}
// Users visiting only /dashboard never download Profile's JS at all.
```

**Multiple Suspense boundaries — granular loading states:**

```jsx
function Dashboard() {
  return (
    <>
      <Header /> {/* renders immediately, not lazy */}
      <Suspense fallback={<ChartSkeleton />}>
        <LazyChart /> {/* shows its own skeleton independently */}
      </Suspense>
      <Suspense fallback={<TableSkeleton />}>
        <LazyTable /> {/* doesn't wait for the chart, or vice versa */}
      </Suspense>
    </>
  );
}
```

## Common interview questions
- What problem does `React.lazy` solve?
- What does `<Suspense>` actually do, mechanically?
- Why would you use multiple `<Suspense>` boundaries instead of one at the app root?
  (Granular loading — unrelated sections don't block each other; one slow
  chunk doesn't blank out the whole page.)
- Does `Suspense` work for data fetching, or only code-splitting?
  (Historically code-splitting only; React 18+ and frameworks like Next.js/
  Relay increasingly support Suspense-integrated data fetching too — worth
  a brief mention that this is evolving.)
- What happens if a lazy-loaded component's `import()` fails (e.g. network error)?
  (Throws, and needs to be caught — typically paired with an
  [Error Boundary](error-boundaries.md) around the Suspense boundary.)

```jsx
<ErrorBoundary fallback={<p>Failed to load this section.</p>}>
  <Suspense fallback={<Spinner />}>
    <LazyComponent />
  </Suspense>
</ErrorBoundary>
```

!!! tip "Gotchas / follow-ups"
    - `lazy()` only works with **default exports** — `import("./Settings")`
      expects `Settings.js` to have a `export default Settings`.
    - Overly granular code-splitting (lazy-loading every tiny component) can
      backfire — too many small chunks means too many network requests;
      split at meaningful boundaries (routes, large rarely-used features) instead.
    - Pairs naturally with Error Boundaries, since a failed dynamic import throws.

## Personal example
_(Add a real case — e.g. lazy-loading a rarely-used admin settings page to
shrink the main bundle, and measuring the before/after bundle size with a
tool like `webpack-bundle-analyzer` or Vite's equivalent.)_

## Related
- [Error Boundaries](error-boundaries.md)
- [Server-Side Rendering Basics](ssr-and-hydration.md)
