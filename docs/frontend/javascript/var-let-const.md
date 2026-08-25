# var vs let vs const

## Concept
Three ways to declare variables in JS, differing in **scope**, **hoisting
behavior**, **re-declaration/re-assignment rules**, and whether they attach to
the global object.

## Why it matters
Choosing the wrong one is a top source of subtle bugs (loop closures, leaked
globals, accidental reassignment). Modern JS/React codebases almost never use
`var` — knowing exactly why is a very common interview filter question.

## How it works

| | `var` | `let` | `const` |
|---|---|---|---|
| Scope | Function-scoped | Block-scoped | Block-scoped |
| Hoisting | Hoisted, initialized as `undefined` | Hoisted, in TDZ until declared | Hoisted, in TDZ until declared |
| Re-declaration | Allowed | Not allowed (same scope) | Not allowed (same scope) |
| Re-assignment | Allowed | Allowed | **Not allowed** |
| Attaches to `window`/`global` | Yes (in non-module scripts) | No | No |

```js
// Function scope vs block scope
function test() {
  if (true) {
    var a = 1;
    let b = 2;
  }
  console.log(a); // 1 — 'var' leaked out of the if-block
  console.log(b); // ReferenceError — 'b' stayed inside the block
}
```

```js
// const doesn't mean "immutable" — it means "can't reassign the binding"
const arr = [1, 2, 3];
arr.push(4);       // fine — mutating the object, not reassigning the variable
console.log(arr);  // [1, 2, 3, 4]

arr = [5, 6];       // TypeError: Assignment to constant variable.
```

```js
// Re-declaration
var x = 1;
var x = 2; // fine

let y = 1;
let y = 2; // SyntaxError: Identifier 'y' has already been declared
```

## Common interview questions
- Difference between `var`, `let`, and `const`?
- Does `const` make an object immutable? (No — see mutation example above.)
- Why did the community move away from `var` in modern codebases?
- What's the loop-closure bug with `var`, and how does `let` fix it?
  (Covered in detail in [Closures](closures.md).)
- Can you use `let`/`const` before declaration if wrapped in `typeof`? Why not?
  (Covered in [TDZ](tdz.md).)

!!! tip "Gotchas / follow-ups"
    - `const` on an object/array still allows mutating its contents — only
      reassignment of the variable itself is blocked. Use `Object.freeze()`
      for true immutability (shallow only).
    - In non-strict-mode scripts, `var` at the top level attaches to `window` —
      a common source of accidental global pollution in older codebases.
    - Default recommendation: use `const` everywhere by default, `let` only
      when you know the variable will be reassigned, and avoid `var` entirely.

## Personal example
_(Add a real refactor story — e.g. migrating a legacy file full of `var` to
`let`/`const` and the loop bug it uncovered.)_

## Related
- [Hoisting](hoisting.md)
- [Temporal Dead Zone](tdz.md)
- [Closures](closures.md)
