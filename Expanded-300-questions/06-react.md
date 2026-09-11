# React Interview Preparation — Experienced Level

Focus: rendering, hooks, performance, architecture, data fetching, testing and security.

## React Fundamentals

### Q1. What happens after a state update?
<details><summary>Answer</summary>

React schedules a render, computes the next UI tree and commits the necessary changes. Rendering and committing are conceptually distinct.

</details>

### Q2. Props vs state?
<details><summary>Answer</summary>

Props are inputs from a parent; state is component-owned data that can trigger renders. Keep state close to where it is needed.

</details>

### Q3. What is reconciliation?
<details><summary>Answer</summary>

React compares rendered trees and determines which UI changes need to be committed.

</details>

### Q4. Why are keys important?
<details><summary>Answer</summary>

Keys provide stable identity for list elements. Unstable keys can cause incorrect state preservation and inefficient reconciliation.

</details>

### Q5. Why avoid array index as key?
<details><summary>Answer</summary>

Insertion/reordering changes item identity, potentially causing component state to follow the wrong item.

</details>

### Q6. Controlled vs uncontrolled input?
<details><summary>Answer</summary>

Controlled inputs use React state as the source of truth. Uncontrolled inputs keep value in the DOM and can be accessed via refs.

</details>

## Hooks

### Q7. Rules of Hooks?
<details><summary>Answer</summary>

Call hooks only at the top level of React functions and only from React components/custom hooks so hook call order remains stable.

</details>

### Q8. useEffect purpose?
<details><summary>Answer</summary>

Use it to synchronize with external systems such as subscriptions, timers or browser APIs. Avoid using it for values that can be derived during render.

</details>

### Q9. What causes stale closures?
<details><summary>Answer</summary>

Functions capture values from their render. Async callbacks may therefore observe old state if dependencies or update patterns are incorrect.

</details>

### Q10. Functional state update?
<details><summary>Answer</summary>

Use the previous-state form when the next state depends on prior state, avoiding stale-value races between updates.

</details>

### Q11. useMemo vs useCallback?
<details><summary>Answer</summary>

useMemo caches a computed value; useCallback caches a function reference. Both should be justified by actual render/dependency costs.

</details>

### Q12. useRef vs state?
<details><summary>Answer</summary>

A ref stores mutable data without causing a render when changed. State represents UI-relevant data that should trigger rendering.

</details>

## Performance

### Q13. How do you find unnecessary renders?
<details><summary>Answer</summary>

Use React DevTools Profiler, inspect component boundaries and state/context propagation, then optimize based on measured hot paths.

</details>

### Q14. What is memoization?
<details><summary>Answer</summary>

Caching a computation or component result to avoid repeated work when inputs have not meaningfully changed.

</details>

### Q15. Why can Context hurt performance?
<details><summary>Answer</summary>

Consumers can re-render when provider values change. Split contexts, stabilize values and use more targeted state subscriptions for high-frequency updates.

</details>

### Q16. What is list virtualization?
<details><summary>Answer</summary>

Render only visible rows of a large list rather than thousands of DOM nodes. It can dramatically reduce rendering and layout cost.

</details>

### Q17. Code splitting?
<details><summary>Answer</summary>

Load JavaScript chunks on demand, often by route or feature, reducing initial bundle cost.

</details>

### Q18. How do you optimize data fetching?
<details><summary>Answer</summary>

Avoid waterfalls, cache server state, deduplicate requests, cancel stale requests and model loading/error/empty states explicitly.

</details>

## Architecture & Security

### Q19. Local vs global state?
<details><summary>Answer</summary>

Keep state local when possible. Promote it when multiple distant components genuinely share ownership. Server state is often better handled by dedicated data-fetching/cache mechanisms.

</details>

### Q20. Context vs Redux-like state library?
<details><summary>Answer</summary>

Context distributes values; a state library can provide selectors, structured updates, middleware and debugging for complex state.

</details>

### Q21. How do you prevent XSS?
<details><summary>Answer</summary>

Avoid unsafe HTML injection, sanitize trusted HTML when required, encode output, use secure CSP and never rely on React alone for backend authorization.

</details>

### Q22. Where should authorization be enforced?
<details><summary>Answer</summary>

On the backend/API. Frontend checks improve UX but cannot be a security boundary.

</details>

### Q23. How do you handle authentication tokens?
<details><summary>Answer</summary>

Prefer secure, appropriately scoped session/token mechanisms; avoid exposing long-lived secrets to JavaScript unnecessarily and design for expiration/rotation.

</details>

### Q24. How do you test React?
<details><summary>Answer</summary>

Unit/component tests for behavior and integration tests for user flows. Prefer user-visible behavior over implementation details.

</details>

## Question Count

**24 experienced-level questions** in this file.

## Quick Revision Checklist

