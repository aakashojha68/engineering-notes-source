# useMemo vs useCallback

## Concept
Both memoize something between renders to avoid unnecessary work — `useMemo`
memoizes a **computed value**, `useCallback` memoizes a **function reference**.
`useCallback(fn, deps)` is actually just `useMemo(() => fn, deps)` under the hood.

## Why it matters
Both are commonly **overused** — wrapping everything in `useMemo`/`useCallback`
"just in case" adds complexity and its own overhead without measurable
benefit. Knowing exactly when they help (and when they're noise) is a strong
signal of real production experience, not just tutorial knowledge.

## How it works

**`useMemo` — skip an expensive recalculation:**

```jsx
function ProductList({ products, filterText }) {
  const filteredProducts = useMemo(() => {
    console.log("Filtering..."); // only logs when 'products' or 'filterText' change
    return products.filter((p) => p.name.includes(filterText));
  }, [products, filterText]);

  return <ul>{filteredProducts.map((p) => <li key={p.id}>{p.name}</li>)}</ul>;
}
```

**`useCallback` — keep a stable function reference:**

```jsx
function Parent() {
  const [count, setCount] = useState(0);

  // WITHOUT useCallback: a brand-new function is created every render,
  // which breaks React.memo on Child (it re-renders every time regardless)
  const handleClick = useCallback(() => {
    console.log("clicked");
  }, []); // stable reference across renders

  return <Child onClick={handleClick} />;
}

const Child = React.memo(function Child({ onClick }) {
  console.log("Child rendered");
  return <button onClick={onClick}>Click</button>;
});
```

**Why `useCallback` matters specifically WITH `React.memo`:**

```jsx
// Without useCallback, this breaks the memoization entirely:
function Parent() {
  const [count, setCount] = useState(0);
  const handleClick = () => console.log("clicked"); // NEW function every render

  return (
    <>
      <button onClick={() => setCount(count + 1)}>Increment: {count}</button>
      <Child onClick={handleClick} /> {/* re-renders every time — memo is useless */}
    </>
  );
}
```

## Common interview questions
- Difference between `useMemo` and `useCallback`?
- When does `useCallback` actually help — and when is it pointless?
  (Only helps if the function is passed to a memoized child, or used as a
  dependency elsewhere that would otherwise cause an infinite effect loop.)
- Does wrapping everything in `useMemo`/`useCallback` always improve performance?
  (No — memoization itself has a cost: comparing dependencies each render.
  For cheap computations/components, it can be net negative.)
- What's the relationship between `useCallback` and `useMemo`?
  (`useCallback(fn, deps) === useMemo(() => fn, deps)`)

## When each actually helps

| Use it when... | Skip it when... |
|---|---|
| The computation is genuinely expensive (large array processing, sorting, filtering big lists) | The computation is trivial (simple arithmetic, short string concat) |
| The function/value is passed to a `React.memo`-wrapped child | The child isn't memoized — the reference stability buys nothing |
| The value is a dependency of another `useEffect`/`useMemo` and you need referential stability to avoid re-triggering it | Nothing downstream cares about reference equality |

!!! tip "Gotchas / follow-ups"
    - Premature memoization is a real anti-pattern — profile first
      (React DevTools Profiler), then memoize the specific bottleneck.
    - `useMemo` is **not a guarantee** — React may discard the cached value
      and recompute in some cases (e.g. under memory pressure); never use it
      for correctness, only performance.
    - This question is almost always followed by "how would you profile
      this to confirm it actually helped?" — mention React DevTools Profiler.

## Personal example
_(Add a real case — e.g. wrapping a large table's row click handler in
`useCallback` to stop 500 memoized rows from re-rendering on every keystroke
in an unrelated search box.)_

## Related
- [Virtual DOM & Reconciliation](virtual-dom-reconciliation.md)
- [React Re-renders & Performance](rerenders-and-performance.md)
