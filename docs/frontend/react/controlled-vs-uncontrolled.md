# Controlled vs Uncontrolled Components

## Concept
A **controlled** component has its value driven entirely by React state —
React is the single source of truth. An **uncontrolled** component manages
its own state internally in the DOM, and React reads it only when needed
(via a `ref`).

## Why it matters
Almost every form-handling bug (input not updating, losing cursor position,
validation timing issues) traces back to accidentally mixing the two
approaches on the same input. It's also a very common "explain the tradeoff"
interview question for anyone building forms.

## How it works

**Controlled — React state drives the input's value:**

```jsx
function ControlledInput() {
  const [value, setValue] = useState("");

  return (
    <input
      value={value}                              // React controls what's displayed
      onChange={(e) => setValue(e.target.value)} // every keystroke updates state
    />
  );
}
```

Every keystroke triggers a re-render — React is the source of truth, so you
can validate, transform, or block input on every change.

**Uncontrolled — the DOM manages its own value, React reads it via ref:**

```jsx
function UncontrolledInput() {
  const inputRef = useRef(null);

  function handleSubmit(e) {
    e.preventDefault();
    console.log(inputRef.current.value); // read the DOM's current value directly
  }

  return (
    <form onSubmit={handleSubmit}>
      <input ref={inputRef} defaultValue="" /> {/* note: defaultValue, not value */}
      <button type="submit">Submit</button>
    </form>
  );
}
```

No re-render on every keystroke — the DOM tracks its own value until you
explicitly read it.

**The classic bug — mixing the two:**

```jsx
// This warns in the console and behaves inconsistently:
// "A component is changing an uncontrolled input to be controlled"
function BuggyInput() {
  const [value, setValue] = useState(); // undefined initially!
  return <input value={value} onChange={(e) => setValue(e.target.value)} />;
  // starts as uncontrolled (value=undefined), becomes controlled after first keystroke
}

// Fix: always initialize controlled state with a defined value
const [value, setValue] = useState(""); // empty string, not undefined
```

## Common interview questions
- Difference between controlled and uncontrolled components?
- Why does React warn about switching between controlled/uncontrolled?
- When would you deliberately choose uncontrolled — isn't controlled always "more React"?
  (Uncontrolled is fine/better for simple forms, file inputs — which are
  always uncontrolled — or integrating with non-React code/libraries that
  manage their own DOM state.)
- How do file inputs (`<input type="file">`) fit into this? (Always
  uncontrolled — the browser owns the file value for security reasons; you
  can only read it via `ref`, never set it programmatically.)
- How do form libraries like React Hook Form use uncontrolled inputs for performance?
  (They use refs internally to avoid a re-render on every keystroke, only
  reading values on submit/validation — a deliberate performance tradeoff at scale.)

!!! tip "Gotchas / follow-ups"
    - Controlled components cost a re-render per keystroke — for very large
      forms, this is why libraries like React Hook Form default to an
      uncontrolled approach for performance.
    - `defaultValue`/`defaultChecked` set the *initial* value for uncontrolled
      inputs — changing them later does nothing, since React isn't tracking them.
    - Mixing controlled and uncontrolled on the SAME input (e.g. state starting
      as `undefined`) is the most common source of the console warning above.

## Personal example
_(Add a real case — e.g. choosing an uncontrolled approach with React Hook
Form for a large multi-step form to avoid re-render overhead on every field.)_

## Related
- [useEffect Deep Dive](useeffect-deep-dive.md)
