# 🔍 JavaScript Equality Operators — Quick Revision

## 1. Types of Equality
| Operator | Checks | Type Conversion | Example |
|-----------|--------|-----------------|----------|
| `===` (Strict) | Value **and** Type | ❌ No | `5 === '5'` → `false` |
| `==` (Loose) | Value only | ✅ Yes | `5 == '5'` → `true` |

**Preferred:** Always use `===` (predictable, avoids coercion).  
**Use `==` only** when intentional coercion is desired — e.g. `x == null` (matches `null` or `undefined`).

---

## 2. ⚠️ Weird `==` Comparisons (Interview Favorites)

| # | Expression | Result | Simple Explanation |
|---|------------|--------:|--------------------|
| 1 | `null == undefined` | ✅ `true` | Special case: equal only to each other |
| 2 | `0 == false` | ✅ `true` | `false` → `0` |
| 3 | `'' == 0` | ✅ `true` | `''` → `0` |
| 4 | `'' == false` | ✅ `true` | Both → `0` |
| 5 | `'0' == 0` | ✅ `true` | `'0'` → `0` |
| 6 | `'0' == false` | ✅ `true` | `'0'` → `0`, `false` → `0` |
| 7 | `[] == false` | ✅ `true` | `[]` → `''` → `0`, `false` → `0` |
| 8 | `[] == ''` | ✅ `true` | `[]` → `''` |
| 9 | `[0] == 0` | ✅ `true` | `[0]` → `'0'` → `0` |
|10 | `[1] == '1'` | ✅ `true` | `[1]` → `'1'` |
|11 | `[1,2] == '1,2'` | ✅ `true` | `[1,2]` → `'1,2'` |
|12 | `' \t\n' == 0` | ✅ `true` | whitespace string → `0` |
|13 | `NaN == NaN` | ❌ `false` | `NaN` never equals anything |
|14 | `null == 0` | ❌ `false` | `null` only equals `undefined` |
|15 | `[] == ![]` | ✅ `true` | `![]` → `false`, then `[] == false` → true |
|16 | `[0] == false` | ✅ `true` | `[0]` → `'0'` → `0`, `false` → `0` |
|17 | `'0' == [0]` | ✅ `true` | `[0]` → `'0'`, `'0' == '0'` |
|18 | `{ } == { }` | ❌ `false` | Different object references |

---

## 3. 🧠 Rules Behind Loose Equality (`==`)
1. **`null` & `undefined` are BFFs** → only equal to each other.
2. **Booleans → Numbers:**  
   `false → 0`, `true → 1`
3. **Strings ↔ Numbers:**  
   When comparing string & number → string converted to number.
4. **Objects (incl. arrays) → Primitive:**  
   Use `toString()` or `valueOf()`  
   - `[] → ''`  
   - `[5] → '5'`  
   - `[1,2] → '1,2'`  
   - `{}` → `'[object Object]'`
5. **Same-type values compare normally.**
6. **`NaN` is special:** not equal to anything, even itself.

---

## 4. 🧩 Mnemonics
- **B→N:** Booleans become Numbers.  
- **O→P:** Objects become Primitives (`toString`/`valueOf`).  
- **Empty array → empty string → 0.**  
- **`null` ↔ `undefined` only.**  
- **`NaN` = never equal.**

---

## 5. ✅ When to Use `==` Intentionally
- `if (x == null)` → checks for both `null` and `undefined`
- When *explicitly* wanting JS coercion (rare)

---

## 6. 🧰 Tips for Interviews
- Always explain coercion **step-by-step:**
  1. Compare types  
  2. Convert objects → primitive (if any)  
  3. Convert booleans → number  
  4. Convert strings → number  
  5. Compare final primitives  
- Mention `===` is the standard safe operator.
- Use `Number.isNaN()` instead of `NaN == NaN`.

---

**Quick rule to live by:**  
> “If you don’t *need* coercion, don’t use `==` — stick with `===`.”