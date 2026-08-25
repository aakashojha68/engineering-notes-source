# Context API vs Redux/Zustand

## Concept
All three solve the same underlying problem — **avoiding prop drilling** by
sharing state across components without passing props down every level — but
they differ in scale, performance characteristics, and tooling.

## Why it matters
"Should we use Context or Redux?" is a real architecture decision you'll be
asked to defend at 3+ YOE — picking the wrong one for the scale of an app is
a common root cause of either boilerplate bloat (Redux for a tiny app) or
performance/maintainability problems (Context for a large, frequently-updating app).

## How it works

**Context API — built into React, best for low-frequency, "global-ish" data:**

```jsx
const ThemeContext = createContext();

function App() {
  const [theme, setTheme] = useState("dark");
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <Toolbar />
    </ThemeContext.Provider>
  );
}

function Toolbar() {
  const { theme, setTheme } = useContext(ThemeContext); // no prop drilling needed
  return <button onClick={() => setTheme(theme === "dark" ? "light" : "dark")}>
    Current: {theme}
  </button>;
}
```

**The core Context performance problem:**

```jsx
// EVERY consumer of ThemeContext re-renders whenever ANY value in the
// context object changes — even consumers that only care about 'theme'
// will re-render when 'user' changes, if both live in the same context.
<ThemeContext.Provider value={{ theme, user, cart }}>
```

Context has no built-in mechanism to let a component subscribe to just a
*slice* of the value — this is the main reason large apps outgrow it.

**Zustand — minimal external store, subscribes to slices directly:**

```jsx
import { create } from "zustand";

const useCartStore = create((set) => ({
  items: [],
  addItem: (item) => set((state) => ({ items: [...state.items, item] })),
}));

// Component only re-renders when 'items' changes — NOT on unrelated store changes,
// because Zustand tracks subscriptions at the selector level, not the whole store.
function CartBadge() {
  const items = useCartStore((state) => state.items);
  return <span>{items.length}</span>;
}
```

**Redux — the traditional/enterprise-scale option, explicit and structured:**

```jsx
// Action + reducer + store (classic pattern)
const cartSlice = createSlice({
  name: "cart",
  initialState: { items: [] },
  reducers: {
    addItem: (state, action) => {
      state.items.push(action.payload); // Redux Toolkit uses Immer internally
    },
  },
});

function CartBadge() {
  const items = useSelector((state) => state.cart.items); // subscribes to a slice
  const dispatch = useDispatch();
  return <span>{items.length}</span>;
}
```

## Comparison

| | Context API | Zustand | Redux (Toolkit) |
|---|---|---|---|
| Setup | Built-in, zero deps | Tiny library, minimal boilerplate | More boilerplate, but structured |
| Re-render behavior | Any change re-renders ALL consumers of that context | Selector-based — only relevant components re-render | Selector-based — only relevant components re-render |
| DevTools | None built-in | Basic | Excellent (time-travel debugging, action log) |
| Best for | Rarely-changing global data: theme, auth user, locale | Small-to-mid apps needing real state management without ceremony | Large apps, complex state logic, teams needing strict conventions/traceability |

## Common interview questions
- Why does Context re-render more components than Redux/Zustand for the same state change?
- When would you choose Context over a state management library, and vice versa?
- What's "prop drilling," and how does each of these solve it?
- Can you use Context AND Redux/Zustand together in the same app? (Yes —
  common pattern: Context for rarely-changing app-wide values like theme/auth,
  a store for frequently-updated domain state like cart/data.)
- How would you optimize Context to avoid the "re-renders everyone" problem?
  (Split into multiple smaller contexts, or memoize the provider value.)

!!! tip "Gotchas / follow-ups"
    - A common Context performance mistake: passing a new object literal as
      the `value` prop every render (`value={{ theme, setTheme }}`) — this
      creates a new reference every time, defeating any memoization downstream.
      Wrap it in `useMemo`.
    - Context isn't a "state management library" — it's a dependency-injection
      mechanism. It has no actions, reducers, or built-in update batching logic of its own.
    - Redux Toolkit (`@reduxjs/toolkit`) is the modern standard — plain Redux
      with manual boilerplate is rarely written from scratch anymore; mention
      Toolkit specifically if asked about Redux.

## Personal example
_(Add a real case — e.g. migrating a growing app from Context-only state to
Zustand/Redux after noticing unrelated components re-rendering on every
unrelated context update.)_

## Related
- [React Re-renders & Performance](rerenders-and-performance.md)
- [useMemo vs useCallback](usememo-vs-usecallback.md)
