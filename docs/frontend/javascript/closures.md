# Closures

## Concept
A closure is a function that **remembers the variables from its outer (lexical)
scope**, even after that outer function has finished executing. The inner
function "closes over" those variables instead of losing access to them.

## Why we need it (real-world example)
Without closures, you cannot create **private state** in JavaScript the way
`private` fields work in Java. Closures are the mechanism behind:

- Data privacy / encapsulation (before ES2022 private class fields existed)
- Function factories (functions that generate customized functions)
- Memoization / caching
- Maintaining state in callbacks (e.g. event handlers, `setTimeout`, React hooks)

**Real-world example — a bank account module (private balance):**

```js
function createBankAccount(initialBalance) {
  let balance = initialBalance; // private — no external code can touch this directly

  return {
    deposit(amount) {
      balance += amount;
      return balance;
    },
    withdraw(amount) {
      if (amount > balance) {
        console.log("Insufficient funds");
        return balance;
      }
      balance -= amount;
      return balance;
    },
    getBalance() {
      return balance;
    },
  };
}

const account = createBankAccount(1000);
account.deposit(500);      // 1500
account.withdraw(200);     // 1300
console.log(account.balance); // undefined — can't access it directly!
console.log(account.getBalance()); // 1300 — only through the closure
```

`balance` lives inside `createBankAccount`'s scope. The three returned methods
form closures over that same `balance` variable — they share it, and it stays
alive in memory as long as `account` exists, even though `createBankAccount()`
already finished running.

**A second everyday example — React (why this matters for your stack):**

```js
function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    // this callback "closes over" the current 'count' from this render
    console.log("Current count is:", count);
  }

  return <button onClick={handleClick}>Click</button>;
}
```

This is exactly why "stale closure" bugs happen in React — `handleClick`
captures the `count` value from the render it was created in, not the latest one.

## How it works

```js
function outer() {
  let counter = 0; // lives in outer's scope

  function inner() {
    counter++; // inner closes over 'counter'
    console.log(counter);
  }

  return inner;
}

const increment = outer(); // outer() runs and finishes
increment(); // 1
increment(); // 2
increment(); // 3 — 'counter' was NOT garbage collected, because 'increment' still references it
```

Normally, local variables are garbage collected once a function returns. But
if an inner function referencing them escapes the outer function (by being
returned, passed as a callback, assigned to a variable, etc.), the JS engine
keeps that scope alive — that's the closure.

## Common interview questions
- What is a closure, in your own words?
- Write a function that returns a counter using closures.
- What's the classic "loop + `var` + `setTimeout`" closure bug, and how do you fix it?
- How do closures enable data privacy in JS?
- What's a "stale closure" in React, and when have you hit one?

**The classic loop bug (very commonly asked):**

```js
for (var i = 1; i <= 3; i++) {
  setTimeout(() => console.log(i), 1000);
}
// Prints: 4, 4, 4  (not 1, 2, 3!)
// Because 'var' is function-scoped — all three callbacks close over the SAME 'i',
// and by the time setTimeout fires, the loop has already finished with i = 4.

// Fix 1: use 'let' (block-scoped — a new 'i' per iteration)
for (let i = 1; i <= 3; i++) {
  setTimeout(() => console.log(i), 1000);
}
// Prints: 1, 2, 3

// Fix 2 (pre-ES6 way): create a new scope manually with an IIFE
for (var i = 1; i <= 3; i++) {
  (function (j) {
    setTimeout(() => console.log(j), 1000);
  })(i);
}
```

!!! tip "Gotchas / follow-ups"
    - Closures can cause **memory leaks** if large objects are captured and never
      released (e.g. a closure holding a reference to a huge array that's never needed again).
    - Every function in JS technically forms a closure over its defining scope —
      even if it doesn't use any outer variables.
    - Interviewers love combining closures with hoisting/TDZ — expect a mixed snippet.

## Personal example
_(Fill this in with a real bug/feature from your own projects — e.g. a debounced
search input in React where you used a closure to preserve a timer ID between
renders, or a Spring Boot equivalent discussion contrasting Java's explicit
`final` captured variables in lambdas vs JS's implicit closures.)_

## Related
- [Hoisting](hoisting.md)
- [Temporal Dead Zone](tdz.md)
- [var vs let vs const](var-let-const.md)
