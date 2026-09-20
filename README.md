
# RawGame Stole my work

## TL;DR

Raw13G's `core.js` and `int64.js` correspond at the source-implementation level to the JavaScript implementation published in Bel's OOMfie repository, with OnePS subsequently documenting and building upon that implementation lineage.

The evidence is a file-level correspondence: the same 64-bit integer representation with the same backing-store coupling, the same overflow strings, the same module exports, the same unusual layout constants with the same values, the same `history.replaceState()` object graph, the same fake-cell/carrier architecture, and the same source-file decomposition.

Historical reference: OOMfie commit `3d7955d7b85982d5f6160adf9ba6cd56278fc300` is a public repository state in which `core.js` and `int64.js` are present alongside the associated implementation files. The comparison below is between that historical source and the current Raw13G source.

The requested remedy is narrow: attribution on `core.js` and `int64.js`, identifying their OOMfie/OnePS lineage and acknowledging the subsequent Raw13G modifications.

---

## 1. `int64.js` — the strongest source fingerprint

The clearest evidence is the 64-bit integer helper.

| Feature | OOMfie | Raw13G |
|---|---|---|
| Integer representation | `low`, `hi` | `low`, `hi` |
| Attached byte view | `.backing` (Uint8Array) | `.backing` (Uint8Array) |
| Fill helper | `zeroFill()` | `zeroFill()` |
| Arithmetic methods | `add32inplace`, `add32`, `sub32`, `sub32inplace`, `and32`, `and64` | same set |
| Backing advancement | view re-slices to follow the represented address | same mechanism |
| Overflow string | `"int64.add32inplace: overflow"` | same |
| Overflow string | `"int64.add32: overflow"` | same |
| Exports | `globalThis.int64`, `export { int64 }`, `export default int64` | same structure |

Any single row on its own is not decisive. Several of them together are.

### Why the `backing` mechanism is the strongest signal

The `.backing` property is not a generic utility. It is a `Uint8Array` view that stays coupled to the address the integer represents. When arithmetic advances the integer, the view is re-sliced so it continues to point at the same physical location.

This is not a conventional requirement of a JavaScript 64-bit integer helper. The implementation instead couples integer arithmetic to a byte-level view that follows the represented address, which directly matches the carrier design documented in the OOMfie source — the fake-cell and vector-pointer machinery needs an address-coupled byte-level view that follows pointer arithmetic.

The two implementations also retain the same combination of representation, methods, backing behavior, overflow strings, and export structure. That combination is what makes this file the strongest fingerprint in the comparison.

---

## 2. `core.js` — structural correspondence

### Constants

| Constant | OOMfie | Raw13G |
|---|---|---|
| `DUPLICATE_INDEX` | 2 | 2 |
| `CONTROL_INDEX` | 0xffff | 0xffff |
| `CONTROL_INT` | -64000 | -64000 |
| `FILLER_BIGINTS` | K - 1 | K - 1 |
| `FILLER_OBJECTS` | 0xfffe - K | 0xfffe - K |
| `EXPECTED_LENGTH` | 0x50001 | 0x50001 |
| `CELL_BYTES` | 0x30 | 0x30 |
| `FUNCTION_BYTES` | 0x20 | 0x20 |
| `NATIVE_EXECUTABLE_BYTES` | 0x38 | 0x38 |
| `HOLDER_BYTES` | 0x40 | 0x40 |

Individual constants with common values (like `2`) don't establish anything. The unusual values — `CONTROL_INT = -64000` and `EXPECTED_LENGTH = 0x50001` — are implementation-specific constants rather than generic JavaScript or `history.replaceState` requirements. Their exact correspondence is therefore more informative than the common constants.

### Function names

| Identifier | Assessment |
|---|---|
| `buildFakeHost` | descriptive, not distinctive on its own |
| `buildAndStoreGraph` | descriptive, not distinctive on its own |
| `prepareAddrof` | somewhat distinctive |
| `loadHistoryCritical` | distinctive |
| `leakScopeObject` | distinctive |
| `prepareSymbolWrapper` | distinctive |
| `fillRawCellPointers` | distinctive |
| `beginComposition` | distinctive |

The argument does not depend on any single identifier being unique. The identifiers occur alongside the same values, object structures, and corresponding implementation roles. The convergence of structure and naming is the evidentiary point — not either one in isolation.

### The `history.replaceState()` object graph

The technique is expressed through a specific object graph:

```js
outerGraph[1] = referenceTarget;
outerGraph[2] = referenceTarget;
...
history.replaceState(outerGraph, "");
```

The same target is inserted at two indices in the object graph, which is then submitted through history.replaceState(). This is a specific exploitation strategy, not a naming convention. It appears in both implementations with the same structure.

Fake-cell / carrier architecture

The OOMfie implementation uses a coupling between the int64.backing view and the carrier machinery:

```
int64.backing
      ↓
   carrier
      ↓
fake-cell / vector manipulation
      ↓
read/write primitives
```

Raw13G retains both sides of that coupling. The int64.js behavior and the core.js carrier construction are not two separate design decisions — they fit together, and both are retained.

Grooming terminology

The grooming terminology itself is not treated as primary evidence. Terms such as drain, slab, guard, and predecessor can occur independently in exploit-development code. The provenance argument instead rests on the combination of implementation structure, constants, identifiers, and the int64 design.

---

# 3. Evidence hierarchy

The evidence is cumulative, but it is not equally weighted.

The strongest correspondence is int64.js, because the unusual .backing design couples a 64-bit representation to a moving byte-level view.

The core.js constants, function structure, object graph, and carrier architecture provide independent corroboration.

Taken together, these correspondences support the conclusion that the Raw13G versions are source-level derivatives rather than independent implementations of the same underlying research.

---

# 4. Git history

The OOMfie repository commit 3d7955d7b85982d5f6160adf9ba6cd56278fc300 changed five files together:

```
core.js
fn_leak.js
index.html
int64.js
ps4_offsets.js
```

The commit contains 2,368 additions and 1,607 deletions.

The later stealth push commit modified only readme.md; it did not introduce the implementation files.

This establishes a public, independently verifiable point in the repository history at which these implementation files existed. The later stealth push changed only readme.md; it did not introduce core.js or int64.js.

---

# 5. Scope

The claim concerns core.js and int64.js. It does not extend to:

· mem.js
· jb.js
· rpc_worker.js
· payload2.bin
· Every firmware offset used in Raw13G
· Every part of the final jailbreak

The original SSV research concept is credited to the earlier researcher identified in Bel's own OnePS documentation — this document is not claiming invention of the underlying technique. It concerns the JavaScript implementation of that technique, which is the implementation lineage documented in OOMfie and reflected in the corresponding Raw13G source.

Raw13G contains modifications and additional components. The claim is not that Raw13G contributed nothing original. It is that the specific implementation lineage of core.js and int64.js is derivative.

---

# 6. A separate note on private provenance

Separately from the public technical record, Bel provided the OOMfie graph and related research material directly to 0xMansoor before the Raw13G release. This exchange is not required to establish the public source correspondence and is not offered as evidence for it.

---

# 7. The ask

Attribution on core.js and int64.js, indicating their OOMfie/OnePS lineage and acknowledging the subsequent Raw13G modifications.

A reasonable form for the header:

```javascript
// Source lineage: Bel's OnePS / OOMfie implementation.
// Original repositories:
// https://github.com/thebelx/OnePS
// https://github.com/thebelx/oomfietest
// Subsequent modifications by the Raw13G authors.
```

The ask is narrow. It concerns the two files whose source lineage is documented above.

---

**Evidence:**

· OOMfie historical commit 3d7955d — https://github.com/thebelx/oomfietest/commit/3d7955d7b85982d5f6160adf9ba6cd56278fc300
· OOMfie historical core.js
· OOMfie historical int64.js
· OOMfie stealth push commit
· OnePS — https://github.com/thebelx/OnePS
· Raw13G repository
· Raw13G core.js
· Raw13G int64.js
