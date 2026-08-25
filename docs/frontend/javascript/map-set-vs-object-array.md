# Map/Set vs Object/Array

## Concept
`Map` and `Set` are ES6 collection types that solve specific limitations of
plain `Object` and `Array` — mainly around key types, ordering guarantees,
and lookup performance.

## Why it matters
Choosing `Object` by default (out of habit) when you actually need a `Map`
is a common code-review flag — especially for anything involving
non-string keys, frequent additions/removals, or "does this exist" checks
on large datasets.

## How it works

**Map vs Object:**

```js
const userRoles = new Map();
userRoles.set("aakash", "admin");
userRoles.set(42, "numeric key works directly"); // keys can be ANY type, not just strings
userRoles.set({ id: 1 }, "object as key works too");

console.log(userRoles.get("aakash")); // "admin"
console.log(userRoles.size);           // 3 — built-in size property
console.log(userRoles.has("aakash"));  // true

for (const [key, value] of userRoles) {
  console.log(key, value); // guaranteed insertion order
}
```

```js
// Object — keys are always coerced to strings (or Symbols)
const obj = {};
obj[42] = "value";
console.log(Object.keys(obj)); // ["42"] — number silently became a string

// No built-in size — you have to do this:
console.log(Object.keys(obj).length);

// Risk: prototype pollution / accidental collisions
const cache = {};
cache["toString"] = "uh oh"; // overwrites inherited Object.prototype.toString
```

| | `Object` | `Map` |
|---|---|---|
| Key types | String or Symbol only | Any value (object, function, primitive) |
| Key order | Not guaranteed (though mostly insertion order in practice) | Guaranteed insertion order |
| Size | `Object.keys(obj).length` | `.size` property |
| Iteration | Need `Object.entries()`/`for...in` | Directly iterable |
| Performance | Fine for small, static shapes | Better for frequent add/remove, large datasets |
| Prototype pollution risk | Yes (inherits from `Object.prototype`) | No — clean collection, no inherited keys |

**Set vs Array (for uniqueness/lookups):**

```js
const ids = [1, 2, 2, 3, 3, 3];
const uniqueIds = new Set(ids);
console.log([...uniqueIds]); // [1, 2, 3] — classic dedup pattern

const visited = new Set();
visited.add("page1");
console.log(visited.has("page1")); // O(1) lookup — vs array.includes() which is O(n)
```

```js
// Deduping an array the OLD way (before Set) — for context in interviews
const uniqueOld = ids.filter((id, index) => ids.indexOf(id) === index); // O(n²)
```

## Common interview questions
- When would you use `Map` over a plain `Object`?
- How do you remove duplicates from an array? (`Set` — shown above.)
- Why is `Set.has()` faster than `Array.includes()` for large datasets?
  (Set uses hash-based lookup, ~O(1); array requires a linear scan, O(n).)
- Can object keys in a JS `Object` be non-strings? What actually happens?
- What's `WeakMap`/`WeakSet`, and why would you use one?
  (Keys must be objects, and they're garbage-collected when no other
  reference exists — useful for caching metadata about DOM nodes/objects
  without causing memory leaks.)

!!! tip "Gotchas / follow-ups"
    - `JSON.stringify()` does **not** work on `Map`/`Set` directly — you need to
      convert first (`Object.fromEntries(map)` or `[...set]`).
    - `Map` preserves insertion order reliably; plain objects mostly do too in
      modern engines but with quirky exceptions for numeric-like keys (they get
      sorted first) — `Map` avoids that ambiguity entirely.
    - Default rule of thumb: use `Object` for fixed, known-shape data
      (like a config or a single record); use `Map` for dynamic key-value
      collections that grow/shrink at runtime.

## Personal example
_(Add a real case — e.g. switching a lookup table keyed by user IDs from a
plain object to a `Map` to avoid prototype pollution issues, or using `Set`
to dedupe API response IDs before rendering a list.)_

## Related
- [Higher-Order Functions](higher-order-functions.md)
