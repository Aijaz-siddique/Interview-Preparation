# React Interview Preparation

### Q1. What happens when React state changes?
<details><summary>Answer</summary>

A state update schedules a render. React calculates the next UI representation and reconciles it with the previous tree, then commits necessary DOM changes.

Modern React can prioritize and schedule work rather than treating every render as one synchronous operation.
</details>

### Q2. Props vs state?
<details><summary>Answer</summary>

Props are inputs passed from a parent. State is data owned by a component that can trigger re-rendering.

Prefer keeping state as local as possible and lifting it only when multiple components genuinely need the same source of truth.
</details>

### Q3. What is reconciliation?
<details><summary>Answer</summary>

Reconciliation is React's process of determining what changed between renders and what needs to be committed to the UI.

Keys help React identify list items across renders.
</details>

### Q4. Why are keys important?
<details><summary>Answer</summary>

Keys provide stable identity for list elements.

Using array indexes as keys can cause incorrect state association when items are inserted, removed or reordered.
</details>

### Q5. useEffect: when should you use it?
<details><summary>Answer</summary>

`useEffect` is for synchronizing a component with external systems such as subscriptions, timers, browser APIs or network interactions.

It should not be used unnecessarily for values that can be calculated during render.
</details>

### Q6. useMemo vs useCallback?
<details><summary>Answer</summary>

`useMemo` memoizes a computed value.

`useCallback` memoizes a function reference.

Neither should be added everywhere. They are useful when recomputation/reference changes have a measurable performance impact or when stabilizing dependencies for memoized children/effects.
</details>

### Q7. Controlled vs uncontrolled components?
<details><summary>Answer</summary>

Controlled inputs have their value driven by React state. Uncontrolled inputs keep state in the DOM and can be accessed via refs.

Controlled inputs are often easier for validation and dynamic behavior; uncontrolled inputs can be simpler and may reduce state management overhead.
</details>

### Q8. Context API vs global state library?
<details><summary>Answer</summary>

Context is useful for values shared across a component tree, such as theme, locale or authenticated user information.

A state library can provide stronger patterns for complex global state, selectors, debugging and updates.

Do not put every piece of application state into Context.
</details>

### Q9. How do you optimize a slow React application?
<details><summary>Answer</summary>

Measure first.

Investigate:
- unnecessary renders
- large component trees
- expensive calculations
- large bundles
- network waterfalls
- large lists
- image sizes
- state granularity

Techniques include memoization, virtualization, code splitting, lazy loading and better data fetching—but only where profiling supports them.
</details>

### Q10. How do you handle API loading/error states?
<details><summary>Answer</summary>

Model explicit states such as idle/loading/success/error and consider stale data, retries, cancellation and race conditions.

For production applications, centralized data-fetching/caching patterns can prevent duplicated requests and inconsistent server state.
</details>

### Q11. What causes stale closures?
<details><summary>Answer</summary>

A function captures values from the render in which it was created. If asynchronous code uses an old function, it may observe stale state.

Solutions include correct effect dependencies, functional state updates and refs when appropriate.
</details>

### Q12. How would you secure a React application?
<details><summary>Answer</summary>

Key areas:
- XSS prevention
- safe handling of user-generated HTML
- authentication/session strategy
- CSRF where applicable
- secure API authorization
- dependency hygiene
- Content Security Policy
- avoiding secrets in frontend bundles

Frontend authorization is not a substitute for backend authorization.
</details>

## Quick Revision Checklist

Rendering → reconciliation → keys → state/props → hooks → effects → memoization → context → state management → API states → performance → security.
