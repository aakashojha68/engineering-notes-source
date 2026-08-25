# The `this` Keyword

## Concept
`this` refers to the object that is currently "executing" the function. Unlike
most languages, its value in JS is determined by **how a function is called**,
not where it's defined (except for arrow functions).

## Why it matters
It's the source of a huge number of real bugs in React class components,
event handlers, and callbacks — and it's one of the most-asked JS fundamentals
questions because it has several distinct rules depending on call style.

## How it works — the 4 binding rules

```js
// 1. Default binding — plain function call → 'this' is undefined (strict mode) or window/global
function show() {
  console.log(this);
}
show(); // undefined (strict mode)

// 2. Implicit binding — called as a method → 'this' is the object before the dot
const user = {
  name: "Aakash",
  greet() {
    console.log(this.name);
  },
};
user.greet(); // "Aakash"

// 3. Explicit binding — call/apply/bind force 'this'
function greet() {
  console.log(this.name);
}
greet.call({ name: "Explicit" }); // "Explicit"

// 4. new binding — 'this' is the newly created object
function Person(name) {
  this.name = name;
}
const p = new Person("Aakash");
console.log(p.name); // "Aakash"
```

**Arrow functions don't have their own `this`** — they inherit it lexically
from the surrounding scope at the time they're defined:

```js
const user = {
  name: "Aakash",
  greetNormal: function () {
    setTimeout(function () {
      console.log(this.name); // undefined — 'this' is NOT 'user' here (default binding kicks in)
    }, 100);
  },
  greetArrow: function () {
    setTimeout(() => {
      console.log(this.name); // "Aakash" — arrow inherits 'this' from greetArrow's scope
    }, 100);
  },
};
user.greetNormal();
user.greetArrow();
```

## Common interview questions
- What determines the value of `this` in a regular function?
- Why don't arrow functions have their own `this`?
- Why did class components in React need `this` bound in constructors
  (`this.handleClick = this.handleClick.bind(this)`)?
- What's `this` inside a `setTimeout` callback, and why?
- Predict `this` across several call styles (method call, standalone call, arrow, `new`).

!!! tip "Gotchas / follow-ups"
    - This is exactly why React moved toward function components + hooks —
      arrow function event handlers sidestep the entire `this`-binding problem class components had.
    - `call`/`apply`/`bind` (see [Call, Apply, Bind](call-apply-bind.md)) exist
      specifically to control `this` explicitly.
    - In strict mode, default binding gives `undefined`; in non-strict mode it
      silently falls back to the global object — a subtle difference interviewers may probe.

## Personal example
_(Add a real case — e.g. a bug in an old class-based React component where a
handler lost its `this` context because it wasn't bound, causing
`this.setState` to throw.)_

## Related
- [Call, Apply, Bind](call-apply-bind.md)
- [Closures](closures.md)
