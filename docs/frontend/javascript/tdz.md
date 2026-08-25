# Temporal Dead Zone (TDZ)

## Concept
The TDZ is the time span between entering a scope (where a `let`/`const`/`class`
variable is hoisted) and the line where it's actually declared. During this
window, the variable exists but is **uninitialized** — accessing it throws a
`ReferenceError` instead of returning `undefined`.

## Why it matters
It's the reason `let`/`const` feel "safer" than `var` — TDZ turns a silent bug
(`undefined` sneaking through your logic) into a loud, immediate error at the
exact line where the mistake happened, making bugs easier to catch early.

## How it works

```js
console.log(typeof x); // "undefined" — var, no TDZ
var x = 10;

console.log(typeof y); // ReferenceError: Cannot access 'y' before initialization
let y = 10;
```

```js
{
  // TDZ for 'msg' starts here (block begins)
  console.log(msg); // ReferenceError
  let msg = "hello"; // TDZ ends here — 'msg' is now initialized
  console.log(msg); // "hello" — safe from here on
}
```

```js
// TDZ applies per-scope, and classes are affected too:
class Foo {}
function test() {
  console.log(Foo); // ReferenceError — TDZ, even though an outer 'Foo' exists!
  class Foo {}
}
test();
// The inner 'class Foo' hoists to the top of test()'s scope and shadows the
// outer Foo immediately — so the outer one isn't visible either.
```

## Common interview questions
- What is the Temporal Dead Zone?
- Why does `let`/`const` throw a `ReferenceError` but `var` returns `undefined`?
- Is TDZ related to hoisting, or separate from it?
- Does `typeof` behave differently for a TDZ variable vs an undeclared one?

```js
console.log(typeof neverDeclared); // "undefined" — safe, no error
console.log(typeof inTDZ);         // ReferenceError — TDZ variable
let inTDZ = 1;
```

!!! tip "Gotchas / follow-ups"
    - TDZ variables ARE hoisted (their existence is known upfront) — they're just
      not *initialized* yet. That's the subtle but important distinction from "no hoisting at all."
    - Function parameters can also have their own mini-TDZ in edge cases
      (default parameter values referencing later parameters).
    - This pairs almost every time with a hoisting question in interviews — expect both together.

## Personal example
_(Add a real instance — e.g. a case where destructuring `const { a, b } = obj`
early in a component and referencing it before its actual declaration line
threw a TDZ error during a refactor.)_

## Related
- [Hoisting](hoisting.md)
- [var vs let vs const](var-let-const.md)
