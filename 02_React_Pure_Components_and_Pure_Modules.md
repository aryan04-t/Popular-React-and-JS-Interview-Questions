## ✨ Pure Components & Pure Modules — Compact Version

### Pure Component (React)
A pure component in React adheres to the principle of "purity" in functional programming:

- **Same Input, Same Output:** Given the same props and state, it always renders the exact same output.  
- **No Side Effects:** Does not modify external variables, perform network requests, or interact with the DOM outside rendering.

✅ Example:
```js
const NameBadge = React.memo(function NameBadge({ name }) {
  return <span>{name}</span>;
});
```
- `React.memo` prevents unnecessary re-renders if props don't change.

---

### Pure Module
A module is pure if:

- Importing it does not trigger side effects (no network calls, DOM mutations, or global state changes).  
- It only exports functions/values that execute when explicitly called.

✅ Example:
```js
// utils/format.js
export function formatName(s) { return s.trim(); }
```

❌ Bad Example:
```js
// badModule.js
console.log('module loaded'); // side effect
fetch('/init');               // side effect on import
export const x = 1;
```

**Summary:**  
Pure components + pure modules = predictable, tree-shakeable, and safe builds.