# Hoisting

## Concept
JavaScript moves variable and function **declarations** to the top of their scope
during the compile phase, before code executes. Only the declaration is hoisted,
not the assignment/initialization.

## Why it matters
Explains bugs like getting `undefined` instead of `ReferenceError` when a `var`
is used before its declaration, or why a function can be called before its
definition in code — but doing the same with a `const`/arrow function crashes.

## How it works

```js
console.log(a); // undefined (not an error)
var a = 5;

// What JS actually does under the hood:
var a;
console.log(a); // undefined
a = 5;
```

```js
console.log(b); // ReferenceError: Cannot access 'b' before initialization
let b = 5;
// let/const ARE hoisted, but land in the "Temporal Dead Zone" (TDZ)
// until their line executes — that's why it throws instead of returning undefined.
```

```js
sayHi(); // works fine
function sayHi() { console.log("hi"); }
// Function declarations are fully hoisted — name AND body.

sayBye(); // TypeError: sayBye is not a function
var sayBye = function () { console.log("bye"); };
// Only the 'var sayBye' declaration hoists, not the function assignment.
```

## Common interview questions
- What's the difference between `var`, `let`, and `const` hoisting?
- What is the Temporal Dead Zone?
- Why does calling a function expression before its definition throw, but a
  function declaration doesn't?
- Is `class` hoisted in JS?
- What's the output of a tricky snippet mixing `var`/`let`/functions?

!!! tip "Gotchas / follow-ups"
    - `class` declarations ARE hoisted but stay in the TDZ (like `let`) — commonly missed.
    - Hoisting happens per-scope, not just globally — function scope resets it.
    - Interviewers often follow up with "how would you avoid hoisting bugs?" →
      prefer `let`/`const`, declare functions before use, enable strict mode/linting.

## Personal example
Hit a bug in a React form component where a `var isValid` inside a conditional
block leaked and got hoisted to function scope, causing a validation check to
silently pass with `undefined`. Switched to `let` scoped inside the block — bug
gone. Good story for "tell me about a subtle bug you fixed."

## Related
- [Closures](../javascript/closures.md)
- [Temporal Dead Zone](../javascript/tdz.md)
- var vs let vs const
