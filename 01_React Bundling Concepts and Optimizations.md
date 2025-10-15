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
| **Runtime / Manifest** | Bootstraps app, maps chunk IDs → filenames, handles dynamic imports. Lets the browser know how the chunks are connected to each other in the web-app |
| **Vendors** | Third-party libraries grouped for caching efficiency. |
| **Main / App** | Core app logic loaded on first visit. |
| **Lazy / Route / Feature** | Created by `import()` or `React.lazy`, loaded on demand. |
| **Shared / Common** | Code reused by multiple lazy chunks, extracted to avoid duplication. |
| **Asset** | Non-JS files (CSS, images, fonts) emitted separately with hashes. |

---

## 🔹 How Chunks Are Served

#### When you open a web page:

- HTML file loads in the browser.
- The HTML references initial JS bundles (usually runtime + vendors + main/app).
- Browser downloads **runtime** first, then **vendors**, then **main/app**.

#### Why this order?

- **Runtime** must come first — it knows how to load and connect other chunks.
- **Vendors** are usually next — core libraries like React, lodash, etc., needed by main/app.
- **Main / App** contains your actual app logic — depends on runtime + vendors.
- **Lazy / Route / Feature Chunks** are loaded on demand when a user navigates to a route or triggers a dynamic import.
- **Shared / Common Chunks** are loaded before any lazy chunk that depends on them to avoid duplication and improve caching.
- **Assets for non-lazy chunks** are usually loaded immediately as part of the initial page load, and **assets for lazy chunks** are loaded alongside their corresponding chunk when requested.

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
// ❌ Bad: Creating a barrel file which re-exports everything 

// components/index.ts
export * from './BigWidget';   // heavy charting lib
export * from './HeavyChart';  // runs setup code on import

// main.ts
import { BigWidget } from './components'; // bundler may pull both files into bundle
```

```js
// ✅ Good: Don't create barrel files like components/index.ts and prefer direct imports from each module

// main.ts
import BigWidget from './components/BigWidget';
```

- Barrel files (index.ts) that use export * can block tree-shaking.
- Bundlers may include all re-exported modules, even unused ones.
- Use explicit exports or direct imports for better tree-shaking.

---

### 5. Keep Modules Small & Side-Effect-Free
- Avoid top-level API calls, event listeners, or computations in modules. No code should *run* at import-time.  
- Modules become predictable and easily tree-shakeable.


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

---

### 6. Avoid Circular Dependancies 

#### What Are Circular Dependencies?

A circular dependency happens when **two or more modules import each other**, directly or indirectly.

❌ Example:
```js
// a.js 

import { bFunc } from "./b.js"; 

export function aFunc() { console.log("aFunc"); } 
export function cFunc() { bFunc() } 

// b.js 

import { aFunc } from "./a.js"; 

export function bFunc() { aFunc(); } 
```

❓ What's the Issue: 
- When Node loads a.js: It sees import "./b.js" Then loads b.js, which imports a.js But since a.js is already loading, Node gives b.js a temporary export object that doesn’t yet have aFunc. 
- So bFunc has a reference to an uninitialized aFunc, and this is the issue which circular dependancies can cause.

🌀 Causes:
- `undefined` imports or runtime crashes.  
- Harder tree-shaking and slower builds.  

✅ Fixes:
- Keep dependency flow one-way (utils → components → pages → app).  
- Move shared logic to neutral `shared/helpers.js`.  
- Never import UI components into logic layers.  
- Use ESLint rule `import/no-cycle` to detect cycles.  

---

### 7. Use Latest Build Tools
- Prefer **Vite**, **esbuild**, or **Rollup** for ESM-first builds.  
- Use `vite build --analyze` or `webpack-bundle-analyzer` to inspect heavy deps.  
- Enable gzip/brotli compression and long-term caching.  
- Use `"module": "ESNext"` in `tsconfig.json` for optimal tree-shaking.

---

### 8. Split Routes by Feature Folders
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

### 9. Extract Shared Utilities Intentionally
- Only place truly reused logic in `shared/` folder.
- Avoid bloated `shared/index.ts` which imports everything, instead use dependancy injection approach whenever needed.

---

### 10. Other Small Wins
- Remove dead code, unused exports and imports.
- Memoize expensive functions (only where measurable).
- Compress assets (gzip, brotli) on build.
- Use `React.memo`, `useCallback`, and `useMemo` smartly.

---