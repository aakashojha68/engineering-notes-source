# useEffect Deep Dive

## Concept
`useEffect` lets function components run side effects (data fetching,
subscriptions, manual DOM work, timers) after render. Its behavior is
entirely governed by the **dependency array** — this is where most real
bugs live.

## Why it matters
`useEffect` bugs are probably the single most common React interview and
real-world debugging topic — infinite loops, stale closures, and missing
cleanup all trace back to misunderstanding when effects re-run and what they
capture.

## How it works — the three dependency array modes

```jsx
useEffect(() => {
  console.log("runs after EVERY render");
}); // no dependency array

useEffect(() => {
  console.log("runs ONCE, after initial mount only");
}, []); // empty array

useEffect(() => {
  console.log("runs after mount, and again whenever 'count' changes");
}, [count]); // dependency array with values
```

**Cleanup function — runs before the next effect, and on unmount:**

```jsx
useEffect(() => {
  const timerId = setInterval(() => console.log("tick"), 1000);

  return () => {
    clearInterval(timerId); // cleanup — prevents leaked timers/subscriptions
  };
}, []);
```

**Stale closures — the classic `useEffect` bug (ties directly to
[Closures](../javascript/closures.md)):**

```jsx
function Timer() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      console.log(count);      // BUG: always logs 0
      setCount(count + 1);     // BUG: always sets to 1, never increments further
    }, 1000);
    return () => clearInterval(id);
  }, []); // empty deps — effect closes over 'count' from the FIRST render only

  return <div>{count}</div>;
}
```

**Why:** the effect function is created once (deps `[]`), so it closes over
whatever `count` was during the render it was defined in — `0`, forever.

**Fix 1 — functional update (doesn't need to read `count` from the closure):**

```jsx
useEffect(() => {
  const id = setInterval(() => {
    setCount((prev) => prev + 1); // reads the LATEST state, not the closed-over one
  }, 1000);
  return () => clearInterval(id);
}, []);
```

**Fix 2 — add `count` to deps (effect re-runs, gets a fresh closure each time):**

```jsx
useEffect(() => {
  const id = setInterval(() => {
    setCount(count + 1);
  }, 1000);
  return () => clearInterval(id);
}, [count]); // works, but resets the interval every second — often not ideal
```

## Common interview questions
- What are the three forms of the dependency array, and when does each run?
- What is a stale closure in `useEffect`, and how do you fix it?
- Why does `useEffect`'s cleanup function run before the *next* effect, not just on unmount?
- What happens if you fetch data in `useEffect` without a cleanup/cancellation
  check, and the component unmounts mid-fetch? (Causes a "state update on an
  unmounted component" warning/memory leak — need an `isMounted`/`AbortController` guard.)
- Why does React (in Strict Mode, dev only) run effects twice on mount?

**Fetch with cleanup (avoiding the unmounted-component bug):**

```jsx
useEffect(() => {
  const controller = new AbortController();

  fetch(`/api/user/${userId}`, { signal: controller.signal })
    .then((res) => res.json())
    .then((data) => setUser(data))
    .catch((err) => {
      if (err.name !== "AbortError") console.error(err);
    });

  return () => controller.abort(); // cancels the fetch if the component unmounts first
}, [userId]);
```

!!! tip "Gotchas / follow-ups"
    - The ESLint `exhaustive-deps` rule exists specifically to catch missing
      dependencies before they become stale-closure bugs — don't casually disable it.
    - Effects with object/array/function dependencies re-run every render
      unless those values are memoized (`useMemo`/`useCallback`) — a frequent
      cause of "infinite loop" bugs.
    - React 18 Strict Mode intentionally double-invokes effects in
      development to surface missing cleanup — it's not a bug, it's a
      diagnostic (production only runs once).

## Personal example
_(Add a real case — e.g. a stale closure bug in a polling/interval feature,
or an unmounted-component warning traced back to a missing `AbortController`.)_

## Related
- [Closures](../javascript/closures.md)
- [useMemo vs useCallback](usememo-vs-usecallback.md)
- [Custom Hooks](custom-hooks.md)
