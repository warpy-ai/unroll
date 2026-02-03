# Future Work: Lifetime System Extensions

**Status:** Planned
**Prerequisites:** Lifetime implementation complete (Phases 1-7)
**Date:** 2026-02-01

---

## Overview

This document describes future extensions to Oite's lifetime system. The core lifetime implementation is complete in the self-hosted compiler. These features extend the system for ecosystem integration and advanced use cases.

---

## 1. Unroll Ecosystem Integration

### 1.1 Import Classification by Lifetime Capability

Unroll should classify import sources and apply appropriate lifetime checking.

**Architecture:**

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         USER APPLICATION                                 │
│   import { serve } from "@rolls/http";     ← Full lifetime support       │
│   import { connect } from "@rolls/db";     ← Full lifetime support       │
│   import lodash from "lodash";             ← .nroll (no lifetimes)       │
│   import { utils } from "./helpers.ts";    ← TS compat (scope-based)     │
└───────────────────────────────────────────────────────────────────────────┘
                                    │
          ┌─────────────────────────┼─────────────────────────┐
          │                         │                         │
          ▼                         ▼                         ▼
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────────────┐
│     ROLLS       │     │   NPM (.nroll)  │     │   JS/TS FILES           │
│  @rolls/http    │     │   lodash.nroll  │     │   *.js, *.ts            │
│  @rolls/fs      │     │   uuid.nroll    │     │                         │
│                 │     │                 │     │                         │
│ Full lifetime   │     │ No lifetimes    │     │ Scope-based only        │
│ Zero-copy APIs  │     │ Copy at boundary│     │ Backwards compat        │
└─────────────────┘     └─────────────────┘     └─────────────────────────┘
```

**Import Source Classification:**

| Import Source          | Format   | Lifetime Support | Memory Model                |
| ---------------------- | -------- | ---------------- | --------------------------- |
| `@rolls/*`             | `.ot`    | Full             | Zero-copy, lifetime-tracked |
| `npm:*` / bare imports | `.nroll` | None             | Value semantics, copies     |
| `./file.ot`            | `.ot`    | Full             | Zero-copy, lifetime-tracked |
| `./file.ts`            | `.ts`    | Scope-only       | Scope-based borrows         |
| `./file.js`            | `.js`    | None             | Dynamic, no tracking        |

**Tasks:**

- [ ] Implement import source detection in Unroll resolver
- [ ] `.ot` imports → Full lifetime tracking
- [ ] `.ts` imports → Scope-based checking only
- [ ] `.js` imports → No static checking (runtime ownership)
- [ ] `.nroll` imports → Copy at boundary

**Configuration (unroll.toml):**

```toml
[build]
strict_lifetimes = false  # Error if .js/.ts returns borrows
```

---

### 1.2 Declaration Files for NPM Packages

Allow users to provide lifetime-annotated declarations for npm packages via `.d.ot` files.

**Example:**

```typescript
// lodash.d.ot
declare module "lodash" {
    export function first<'a, T>(arr: &'a T[]): &'a T | null;
    export function find<'a, T>(arr: &'a T[], pred: fn(&T) -> bool): &'a T | null;
}
```

**Tasks:**

- [ ] Load `.d.ot` files alongside `.nroll` packages
- [ ] Override inferred types with declaration types
- [ ] Validate declarations match actual signatures
- [ ] Document declaration file format

---

### 1.3 NPM Interop Strategy

NPM packages use copy-at-boundary semantics:

```javascript
import _ from "lodash";

let items = [{ name: "b" }, { name: "a" }];
let sorted = _.sortBy(items, "name"); // COPIES items
console.log(items[0].name); // OK - items wasn't moved
```

**Rationale:**

1. JS libraries assume GC - may hold references indefinitely
2. Can't retrofit lifetime annotations onto npm code
3. Copy-at-boundary is safe and predictable
4. Oite has no GC - ownership model only

---

## 2. Cross-File Lifetime Tracking

Track lifetimes across `.ot` module boundaries.

**Example:**

```javascript
// collections.ot
export function iter<'a, T>(arr: &'a T[]): Iterator<'a, T> {
    return new ArrayIterator(arr);
}

export class ArrayIterator<'a, T> {
    #data: &'a T[];
    #index: number;

    constructor(data: &'a T[]) {
        this.#data = data;
        this.#index = 0;
    }

    next(): &'a T | null {
        if (this.#index >= this.#data.length) return null;
        return &this.#data[this.#index++];
    }
}

// main.ot
import { iter } from "./collections.ot";

let items = [1, 2, 3];
let it = iter(&items);  // it borrows items

while (let item = it.next()) {
    console.log(item);
}
// it goes out of scope, borrow released
```

**Tasks:**

- [ ] Export lifetime parameter info in module metadata
- [ ] Import and instantiate lifetime params across modules
- [ ] Track borrowed references across module calls
- [ ] Add cross-module lifetime tests

---

## 3. Advanced Lifetime Features

### 3.1 Lifetime Bounds

Express that one lifetime must outlive another:

```javascript
// 'a must outlive 'b
function extend<'a, 'b: 'a>(short: &'b T, long: &'a T): &'a T
```

**Tasks:**

- [ ] Parse `'a: 'b` bound syntax
- [ ] Add bounds to LifetimeParam
- [ ] Generate outlives constraints from bounds
- [ ] Validate bounds during constraint solving

---

### 3.2 Higher-Ranked Lifetimes

Enable functions that work for any lifetime:

```javascript
// For any lifetime 'a, this function works
function apply<F>(f: for<'a> fn(&'a T) -> &'a U, x: &T): &U
```

**Tasks:**

- [ ] Parse `for<'a>` syntax
- [ ] Implement higher-ranked trait bounds
- [ ] Instantiate fresh lifetimes at call sites
- [ ] Add tests for higher-ranked patterns

---

### 3.3 Advanced Variance

Full covariance/contravariance for lifetime parameters in generic types.

Current implementation handles basic variance:

- `&'a T` is covariant in `'a`
- `&'a mut T` is invariant in `'a`

**Future:**

- [ ] Infer variance for user-defined generic types
- [ ] Support contravariant positions
- [ ] Bivariance for phantom lifetimes

---

## 4. Open Design Questions

1. **Syntax for lifetime bounds**: Use `'a: 'b` (Rust-style) or `'a extends 'b` (TS-style)?

2. **Anonymous lifetimes**: Support `&'_ T` for explicit "infer this lifetime"?

3. **Lifetime in closures**: How to handle captured references in closures?
   - Option A: Closures capture by reference with inferred lifetime
   - Option B: Require explicit lifetime annotation on capturing closures
   - Option C: Closures always capture by value (clone)

4. **Error message verbosity**: How detailed should lifetime errors be?
   - Minimal: "lifetime mismatch"
   - Verbose: Full explanation with suggestions (like Rust)

---

## 5. File Type Behavior Summary

| File Type | Borrow Checking | Lifetime Params | Return Borrows | Memory Model            |
| --------- | --------------- | --------------- | -------------- | ----------------------- |
| `.ot`     | Full            | Yes             | Yes            | Ownership, compile-time |
| `.ts`     | Scope-based     | No              | No (copies)    | Ownership, compile-time |
| `.js`     | None            | No              | No             | Ownership, runtime only |
| `.nroll`  | N/A             | No              | No (copies)    | Copy at boundary        |

---

## References

- [Rust RFC 0141: Lifetime Elision](https://rust-lang.github.io/rfcs/0141-lifetime-elision.html)
- [Rust Nomicon: Lifetimes](https://doc.rust-lang.org/nomicon/lifetimes.html)
- [Polonius: Next-gen Rust borrow checker](https://github.com/rust-lang/polonius)
- [Cyclone: Region-based Memory Management](https://cyclone.thelanguage.org/)
