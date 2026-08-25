# Custom Hooks

## Concept
A custom hook is just a regular JS function whose name starts with `use` and
that calls other hooks internally. It exists purely to **extract and reuse
stateful logic** between components — without the wrapper-hell of older
patterns like render props or higher-order components.

## Why it matters
Before hooks, sharing stateful logic (e.g. "track window width," "debounce a
value," "fetch and cache data") meant HOCs or render props — both add extra
component nesting and indirection. Custom hooks let you extract that logic
into a plain function with zero extra DOM/component wrapping.

## How it works

**A real, reusable example — `useDebounce`:**

```jsx
function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timerId = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timerId); // cancel if value changes before delay ends
  }, [value, delay]);

  return debouncedValue;
}

// Usage — the component doesn't know or care HOW debouncing works internally
function SearchBox() {
  const [query, setQuery] = useState("");
  const debouncedQuery = useDebounce(query, 500);

  useEffect(() => {
    if (debouncedQuery) fetchResults(debouncedQuery);
  }, [debouncedQuery]);

  return <input value={query} onChange={(e) => setQuery(e.target.value)} />;
}
```

**Another common one — `useFetch`:**

```jsx
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const controller = new AbortController();
    setLoading(true);

    fetch(url, { signal: controller.signal })
      .then((res) => res.json())
      .then((data) => {
        setData(data);
        setLoading(false);
      })
      .catch((err) => {
        if (err.name !== "AbortError") {
          setError(err);
          setLoading(false);
        }
      });

    return () => controller.abort();
  }, [url]);

  return { data, loading, error };
}

// Any component can now do this in one line:
function UserProfile({ userId }) {
  const { data: user, loading, error } = useFetch(`/api/users/${userId}`);
  if (loading) return <Spinner />;
  if (error) return <ErrorMessage error={error} />;
  return <div>{user.name}</div>;
}
```

## The Rules of Hooks (and why they exist)

1. **Only call hooks at the top level** — never inside loops, conditions, or
   nested functions.
2. **Only call hooks from React function components or other custom hooks** —
   never from plain JS functions or class components.

```jsx
// WRONG — breaks Rule 1
function Component({ shouldTrack }) {
  if (shouldTrack) {
    useEffect(() => trackPageView()); // conditional hook call
  }
}
```

**Why this rule exists:** React tracks hooks by **call order**, not by name —
internally it's a linked list matched to each render's position. If a hook
call is skipped conditionally, every hook *after* it shifts position, and
React attaches the wrong state to the wrong hook.

```jsx
// CORRECT — keep the hook call unconditional, put the condition INSIDE
function Component({ shouldTrack }) {
  useEffect(() => {
    if (shouldTrack) trackPageView();
  }, [shouldTrack]);
}
```

## Common interview questions
- What is a custom hook, technically? (Just a function using other hooks, named `use*`.)
- What problem did custom hooks solve that HOCs/render props had?
- Why can't hooks be called conditionally? What breaks internally if you do?
- Write a custom hook for `useLocalStorage` or `useDebounce` live.
- Can a custom hook call another custom hook? (Yes — they compose freely.)

!!! tip "Gotchas / follow-ups"
    - The `use` naming convention isn't just style — the `eslint-plugin-react-hooks`
      linter uses it to know which functions to apply the Rules of Hooks to.
    - Custom hooks don't share state between components automatically — each
      call to a custom hook gets its own independent state, just like calling
      `useState` directly in each component would.
    - A very common live-coding ask: implement `useToggle`, `usePrevious`, or
      `useLocalStorage` from scratch — practice a couple of these.

## Personal example
_(Add a real case — e.g. extracting a repeated "form validation + debounced
API check" pattern across multiple forms into a single custom hook.)_

## Related
- [useEffect Deep Dive](useeffect-deep-dive.md)
- [Debounce vs Throttle](../javascript/debounce-vs-throttle.md)
