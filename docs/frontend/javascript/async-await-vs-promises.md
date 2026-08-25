# Async/Await vs Promises

## Concept
`async/await` is **syntax sugar built on top of Promises** — not a different
mechanism. `async` functions always return a Promise, and `await` pauses
execution of that function until the awaited promise settles, without
blocking the rest of the JS thread.

## Why it matters
Same underlying behavior, very different readability — this is a common
"which do you prefer and why" interview discussion, and mixing them
incorrectly is a frequent source of real bugs (unhandled rejections, missing
`await`s, accidental sequential calls that should've been parallel).

## How it works — side by side

```js
// Promise chain version
function getUserOrders(userId) {
  return fetch(`/api/user/${userId}`)
    .then((res) => res.json())
    .then((user) => fetch(`/api/orders/${user.id}`))
    .then((res) => res.json())
    .catch((err) => {
      console.error("Failed:", err);
      throw err;
    });
}
```

```js
// async/await version — same behavior, reads top-to-bottom like sync code
async function getUserOrders(userId) {
  try {
    const userRes = await fetch(`/api/user/${userId}`);
    const user = await userRes.json();
    const ordersRes = await fetch(`/api/orders/${user.id}`);
    const orders = await ordersRes.json();
    return orders;
  } catch (err) {
    console.error("Failed:", err);
    throw err;
  }
}
```

**The common mistake — accidentally serializing parallel work:**

```js
// BAD: these run one after another (slow) even though they don't depend on each other
async function loadDashboard() {
  const user = await fetchUser();       // waits
  const orders = await fetchOrders();   // then waits again
  const payments = await fetchPayments(); // then waits again
  return { user, orders, payments };
}

// GOOD: fire all three at once, await together
async function loadDashboard() {
  const [user, orders, payments] = await Promise.all([
    fetchUser(),
    fetchOrders(),
    fetchPayments(),
  ]);
  return { user, orders, payments };
}
```

## Common interview questions
- Is `async/await` a replacement for Promises, or built on top of them?
- What does an `async` function return if you `return 5` inside it?
  (Answer: a Promise that resolves to `5`.)
- How do you handle errors with `async/await`? (`try/catch`, vs `.catch()` for Promises)
- What's the bug in awaiting three independent calls sequentially instead of
  using `Promise.all`? (Shown above — unnecessary serialization.)
- What happens if you forget to `await` an async function call?

```js
async function save() {
  return "saved";
}

function run() {
  const result = save(); // missing 'await'
  console.log(result); // Promise { <pending> } or Promise {'saved'} — NOT "saved" directly
}
```

!!! tip "Gotchas / follow-ups"
    - `await` only pauses the `async` function it's inside — it does **not**
      block the rest of the JS thread; other code keeps running via the event loop.
    - An `async` function with no explicit `return` still returns a Promise
      that resolves to `undefined`.
    - Unhandled promise rejections (missing `.catch()` or `try/catch`) can crash
      Node processes or silently fail in the browser — always handle errors explicitly.
    - `await` inside a loop runs sequentially — use `Promise.all` with `.map()`
      if the iterations are independent and can run in parallel.

## Personal example
_(Add a real case — e.g. refactoring a `.then()` chain into `async/await` in a
Spring Boot-backed React app for readability, or catching a bug where
sequential `await`s in a loop slowed down a data table's initial load.)_

## Related
- [Promises](promises.md)
- [Event Loop](event-loop.md)
