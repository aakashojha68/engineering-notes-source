# Higher-Order Functions (map/filter/reduce internals)

## Concept
A higher-order function either **takes a function as an argument**, **returns
a function**, or both. `map`, `filter`, and `reduce` are the most common
built-in examples — they abstract away manual loops for transforming, selecting,
and accumulating array data.

## Why it matters
This is the foundation of functional, declarative JS — the style React itself
encourages (`array.map()` to render lists, `.filter()` to derive visible items,
`.reduce()` to compute derived state). Interviewers use "implement your own
`map`" to check whether you actually understand what's happening under the
hood, not just that you can call it.

## How it works

```js
const nums = [1, 2, 3, 4, 5];

const doubled = nums.map((n) => n * 2);         // [2, 4, 6, 8, 10]
const evens = nums.filter((n) => n % 2 === 0);   // [2, 4]
const sum = nums.reduce((acc, n) => acc + n, 0); // 15
```

**Write your own `map`:**

```js
Array.prototype.myMap = function (callback) {
  const result = [];
  for (let i = 0; i < this.length; i++) {
    if (i in this) { // skip holes in sparse arrays, like the real map does
      result.push(callback(this[i], i, this));
    }
  }
  return result;
};

console.log([1, 2, 3].myMap((n) => n * 2)); // [2, 4, 6]
```

**Write your own `filter`:**

```js
Array.prototype.myFilter = function (callback) {
  const result = [];
  for (let i = 0; i < this.length; i++) {
    if (i in this && callback(this[i], i, this)) {
      result.push(this[i]);
    }
  }
  return result;
};

console.log([1, 2, 3, 4].myFilter((n) => n % 2 === 0)); // [2, 4]
```

**Write your own `reduce` (the one most people fumble):**

```js
Array.prototype.myReduce = function (callback, initialValue) {
  let acc = initialValue;
  let startIndex = 0;

  if (acc === undefined) {
    // No initial value: start acc as the first element, skip it in the loop
    acc = this[0];
    startIndex = 1;
  }

  for (let i = startIndex; i < this.length; i++) {
    acc = callback(acc, this[i], i, this);
  }
  return acc;
};

console.log([1, 2, 3, 4].myReduce((acc, n) => acc + n, 0)); // 10
console.log([1, 2, 3, 4].myReduce((acc, n) => acc + n));     // 10 — no initial value
```

**`reduce` can implement `map` and `filter` too — a common follow-up:**

```js
const mapViaReduce = (arr, fn) =>
  arr.reduce((acc, item) => [...acc, fn(item)], []);

const filterViaReduce = (arr, fn) =>
  arr.reduce((acc, item) => (fn(item) ? [...acc, item] : acc), []);
```

## Common interview questions
- Implement `map`, `filter`, or `reduce` from scratch.
- What happens if `reduce` is called on an empty array with no initial value?
  (Throws `TypeError: Reduce of empty array with no initial value`.)
- Can you implement `map` using `reduce`? (Shown above.)
- Difference between `forEach` and `map`? (`forEach` returns `undefined`,
  doesn't build a new array — used purely for side effects.)
- Why prefer `map`/`filter`/`reduce` over a `for` loop in modern JS?

!!! tip "Gotchas / follow-ups"
    - `map`/`filter` always return a **new array** — they don't mutate the original.
    - Chaining `.filter().map()` reads clean but iterates the array twice;
      a single `.reduce()` can do both in one pass if performance matters on large datasets.
    - `reduce` with no initial value on a single-element array just returns
      that element without ever calling the callback — an edge case worth knowing.
    - In React, `.map()` over a list to render JSX is the most common real-world
      use — pairs directly with the `key` prop discussion in
      [React re-renders & performance](../react/rerenders-and-performance.md).

## Personal example
_(Add a real case — e.g. replacing a manual `for` loop with `.reduce()` to
group an API response by category, or a performance fix where chained
`.filter().map()` was collapsed into one `.reduce()` on a large dataset.)_

## Related
- [Closures](closures.md) — the accumulator in your own `reduce` implementation relies on closure-like scoping.
