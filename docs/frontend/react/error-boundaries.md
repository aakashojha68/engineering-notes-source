# Error Boundaries

## Concept
An Error Boundary is a component that catches JS errors thrown anywhere in
its child component tree during rendering, in lifecycle methods, and in
constructors — and shows a fallback UI instead of crashing the entire app.

## Why it matters
Without one, a single unhandled error in a deeply nested component **unmounts
the entire React tree**, showing a blank white screen to the user. Error
boundaries contain the blast radius to just the broken section.

## How it works

Error boundaries **must be class components** — there's no hook equivalent
(as of React 18) because the two lifecycle methods they rely on
(`getDerivedStateFromError`, `componentDidCatch`) don't have hook versions.

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // Called during the render phase — update state to show fallback UI
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // Called during the commit phase — good place to LOG the error
    console.error("Caught by ErrorBoundary:", error, errorInfo);
    // e.g. send to Sentry/logging service here
  }

  render() {
    if (this.state.hasError) {
      return <h2>Something went wrong. Please refresh.</h2>;
    }
    return this.props.children;
  }
}

// Usage — wrap risky sections, not necessarily the whole app
function App() {
  return (
    <ErrorBoundary>
      <Dashboard />
    </ErrorBoundary>
  );
}
```

**Placing boundaries strategically — isolate failures per section:**

```jsx
function App() {
  return (
    <>
      <ErrorBoundary fallback={<p>Sidebar failed to load</p>}>
        <Sidebar />
      </ErrorBoundary>
      <ErrorBoundary fallback={<p>Feed failed to load</p>}>
        <Feed /> {/* if this crashes, Sidebar keeps working */}
      </ErrorBoundary>
    </>
  );
}
```

## What Error Boundaries do NOT catch

```jsx
// 1. Errors in event handlers — use a regular try/catch instead
function Button() {
  function handleClick() {
    try {
      riskyOperation();
    } catch (err) {
      console.error(err); // Error Boundary won't catch this
    }
  }
  return <button onClick={handleClick}>Click</button>;
}

// 2. Errors in async code (setTimeout, promises, fetch callbacks)
useEffect(() => {
  fetchData().catch((err) => console.error(err)); // handle explicitly, boundary won't catch it
}, []);

// 3. Errors during server-side rendering
// 4. Errors thrown in the Error Boundary's own render method
```

## Common interview questions
- What is an Error Boundary, and what lifecycle methods does it use?
- Why must Error Boundaries be class components?
- What kinds of errors does an Error Boundary NOT catch? (Event handlers,
  async code, SSR, errors in the boundary itself.)
- Where would you place Error Boundaries in a real app — one at the root, or many?
  (Usually both — one top-level catch-all, plus targeted boundaries around
  independent, potentially-flaky sections like a third-party widget or a
  data-heavy dashboard panel.)
- Difference between `getDerivedStateFromError` and `componentDidCatch`?
  (First updates state for the fallback UI during render; second is for side
  effects like logging, runs during commit.)

!!! tip "Gotchas / follow-ups"
    - Libraries like `react-error-boundary` provide a hook-friendly wrapper
      API around the same underlying class-based mechanism — worth mentioning
      if asked "is there a hooks way to do this."
    - Error boundaries reset only when you force a remount (e.g. changing a
      `key` on the boundary, or a manual "Try again" button that resets state).
    - Pair this with real error tracking (Sentry, LogRocket) in
      `componentDidCatch` — catching the error without reporting it just
      hides bugs from your team.

## Personal example
_(Add a real case — e.g. wrapping a third-party chart/widget in its own
Error Boundary so a rendering bug in that library didn't take down the whole dashboard.)_

## Related
- [useEffect Deep Dive](useeffect-deep-dive.md)
