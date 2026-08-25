# Call, Apply, Bind

## Concept
Three methods available on every function that let you explicitly control
what `this` refers to when the function runs.

## Why it matters
They're the classic tool for borrowing methods between unrelated objects,
and for locking in `this` (or preset arguments) ahead of time — a very common
whiteboard/live-coding request ("implement your own `bind`").

## How it works

```js
const person = { name: "Aakash" };

function introduce(greeting, punctuation) {
  console.log(`${greeting}, I'm ${this.name}${punctuation}`);
}

// call — invokes immediately, arguments passed individually
introduce.call(person, "Hi", "!"); // "Hi, I'm Aakash!"

// apply — invokes immediately, arguments passed as an array
introduce.apply(person, ["Hello", "."]); // "Hello, I'm Aakash."

// bind — does NOT invoke; returns a NEW function with 'this' locked in
const boundIntroduce = introduce.bind(person, "Hey");
boundIntroduce("?"); // "Hey, I'm Aakash?" — called later, 'this' is still 'person'
```

**Real use — method borrowing:**

```js
function ArrayLike() {
  this[0] = "a";
  this[1] = "b";
  this.length = 2;
}

// Array.prototype.slice doesn't care that this isn't a real array
const arr = Array.prototype.slice.call(new ArrayLike());
console.log(arr); // ["a", "b"] — now a real array
```

**Implementing your own `bind` (common interview ask):**

```js
Function.prototype.myBind = function (context, ...boundArgs) {
  const originalFn = this;
  return function (...callArgs) {
    return originalFn.apply(context, [...boundArgs, ...callArgs]);
  };
};

function greet(greeting) {
  console.log(`${greeting}, ${this.name}`);
}
const bound = greet.myBind({ name: "Aakash" }, "Hi");
bound(); // "Hi, Aakash"
```

## Common interview questions
- Difference between `call`, `apply`, and `bind`?
- Implement your own version of `bind`.
- When would you use `apply` over `call`? (When arguments are already in array form.)
- Why is `bind` useful for React class component event handlers?

## Comparison

| Method | Invokes immediately? | Argument format | Returns |
|---|---|---|---|
| `call` | Yes | Comma-separated | Function's return value |
| `apply` | Yes | Array | Function's return value |
| `bind` | No | Comma-separated | A new bound function |

!!! tip "Gotchas / follow-ups"
    - `bind` is commonly used to preset arguments too (partial application), not just `this`.
    - Modern JS/React code rarely needs these directly — arrow functions and
      hooks eliminated most `this`-binding needs. Still very much asked in interviews though.
    - Arrow functions **ignore** `call`/`apply`/`bind` for `this` — they can't
      have their `this` reassigned since they don't have their own.

## Personal example
_(Add a real case — e.g. using `.apply()` with `Math.max` on an array of
numbers before spread syntax was common, or `bind` in a legacy class component.)_

## Related
- [The this Keyword](this-keyword.md)
