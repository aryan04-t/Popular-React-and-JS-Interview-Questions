# React Bundling — Ultra-Compact Revision Sheet

## 🔹 Bundling Pipeline
1. **Entry → Graph** — Bundler starts from entry files, resolves imports, builds dependency graph.  
2. **Transform** — Transpile (Babel/TS/esbuild) - convert jsx and ts to js, process CSS/assets.  
3. **Optimize** — Tree-shake unused exports, remove dev-only code, minify, scope-hoist.  
4. **Chunking** — Split into runtime, vendor, app, and lazy chunks via `import()` or route splits.  
5. **Emit** — Output hashed JS/CSS/assets + manifest + optional source maps.  
6. **Serve** — Deploy to CDN/object store; long-cache hashed files, short-cache `index.html`, enable gzip/Brotli.

---

## 🔹 Types of Chunks
| Chunk Type | Description |
|-------------|--------------|
| **Runtime / Manifest** | Bootstraps app, maps chunk IDs → filenames, handles dynamic imports. - lets the browser know how the chunks are connected to each other in the web-app |
| **Vendors** | Third-party libraries grouped for caching efficiency. |
| **Main / App** | Core app logic loaded on first visit. |
| **Lazy / Route / Feature** | Created by `import()` or `React.lazy`, loaded on demand. |
| **Shared / Common** | Code reused by multiple lazy chunks, extracted to avoid duplication. |
| **Asset** | Non-JS files (CSS, images, fonts) emitted separately with hashes. |

---

## 🔹 Code-Level Optimizations That Actually Move Bytes

### 1. Use ESM + Named Exports (Enables Tree-Shaking)
```js
// ✅ Tree-shakeable
import { debounce } from 'lodash-es';

// ❌ Pulls whole library (CommonJS)
import _ from 'lodash';
const v = _.debounce(...);
```
- Always prefer ESM versions (`lodash-es`, `date-fns`, etc.)
- Avoid `require()` in frontend code.

---

### 2. Import Subpaths for Large Libraries
```js
// ✅ Only loads the specific module
import format from 'date-fns/format';
```
- Prevents bundler from including unnecessary parts of the library.

---

### 3. Lazy Loading / Dynamic Imports for Routes & Heavy Components
```js
// ✅ Route-level code splitting
const Dashboard = React.lazy(() => import('./Dashboard'));
return <Suspense fallback={<Loader/>}><Dashboard/></Suspense>;
```
- Creates separate lazy chunks loaded only when the route/component is visited.

---

### 4. Avoid Huge Barrel Files (That Break Tree-Shaking)
```js
// ❌ Bad: importing from a big barrel pulls everything
export * from './BigWidget';
export * from './HeavyChart';

// ✅ Good: import only what’s needed
import BigWidget from 'components/BigWidget';
```
- Barrel files (`index.ts`) can disable tree-shaking if they import side-effectful modules.

---


### 5️. `sideEffects: false` → Declare Side-Effect Freedom
```json
// package.json
{
  "sideEffects": false
}
```
- Tells bundler unused exports can be safely removed (tree-shaking).  
- If you import files with global effects (like CSS), mark them explicitly:
```json
{ 
  "sideEffects": ["*.css"] 
}
```

✅ Example:
```js
// utils/math.js
export const add = (a, b) => a + b;
export const sub = (a, b) => a - b;

// main.js
import { add } from './utils/math.js';
// Unused exports like `sub` will be dropped automatically.
```

---

### 6️. Keep Modules Small & Side-Effect-Free
- No code should *run* at import-time.  
- Avoid top-level API calls, event listeners, or computations.
- Export pure functions or lightweight React components.

❌ Bad:
```js
// userService.js
const users = await fetch('/api/users'); // runs on import
export default users;
```

✅ Good:
```js
export const fetchUsers = () => fetch('/api/users');
```

→ Modules become predictable and easily tree-shakeable.

---

### 7. Avoid Circular Dependancies 

#### What Are Circular Dependencies?

A circular dependency happens when **two or more modules import each other**, directly or indirectly.

❌ Example:
```js
// A.js
import { foo } from './B.js';
export const bar = () => foo();

// B.js
import { bar } from './A.js';
export const foo = () => bar();
```
🌀 Causes:
- `undefined` imports or runtime crashes.  
- Harder tree-shaking and slower builds.  

✅ Fixes:
- Keep dependency flow one-way (utils → components → pages → app).  
- Move shared logic to neutral `shared/helpers.js`.  
- Never import UI components into logic layers.  
- Use ESLint rule `import/no-cycle` to detect cycles.  

---

### 8. Use Latest Build Tools
- Prefer **Vite**, **esbuild**, or **Rollup** for ESM-first builds.  
- Use `vite build --analyze` or `webpack-bundle-analyzer` to inspect heavy deps.  
- Enable gzip/brotli compression and long-term caching.  
- Use `"module": "ESNext"` in `tsconfig.json` for optimal tree-shaking.

---

### 9. Split Routes by Feature Folders
```
src/
 ├─ features/
 │   ├─ auth/
 │   │   ├─ routes/
 │   │   └─ components/
 │   ├─ dashboard/
 │   │   ├─ routes/
 │   │   └─ components/
 ├─ shared/
 └─ app/
```
- Enables isolated builds and better chunk caching.

---

### 10. Extract Shared Utilities Intentionally
- Only place truly reused logic in `shared/` folder.
- Avoid bloated `shared/index.ts` importing everything.

---

### 11. Other Small Wins
- Remove dead code and unused exports.
- Memoize expensive functions (only where measurable).
- Compress assets (gzip, brotli) on build.
- Use `React.memo` and `useCallback` smartly.

---