# TL;DR: Where the Raw13G Code Came From

If you're not familiar with PS4 exploit development, the simplest way to understand this is:

**Earlier PS4 research → OOMfie / OnePS → Raw13G**

The important part is that these are not just three projects that happen to solve the same problem. The evidence concerns **specific implementation choices that were carried over.**

## 1. The work started with earlier PS4 research

The underlying SSV research did not originate entirely with Bel. Bel's own OnePS documentation credits the earlier researcher responsible for the original SSV shape.

That distinction matters:

**The underlying research has its own history and deserves its own credit.**

Bel then developed that research into a concrete PS4 exploitation implementation.

## 2. Bel turned that research into OOMfie / OnePS

OOMfie and OnePS contain my actual implementation work: the JavaScript exploitation framework, the fake-cell/carrier system, the grooming system, and supporting `int64.js` implementation.

One particularly important historical point is that the OOMfie implementation was already publicly present in commit:

`3d7955d7b85982d5f6160adf9ba6cd56278fc300`

That commit introduced `core.js`, `int64.js`, `fn_leak.js`, `index.html`, and `ps4_offsets.js`.

So this isn't just a claim based on a README or a retrospective explanation. The implementation exists in the Git history itself.

## 3. Raw13G contains extremely specific pieces of that implementation

This is where the issue becomes significant.

Raw13G's code does not merely use the same general idea.

Its `int64.js` contains the same highly specific implementation fingerprint found in OOMfie:

* the `low`, `hi`, and `backing` properties
* the `zeroFill()` helper
* the same method names
* the same arithmetic structure
* the same technique where the backing `Uint8Array` moves along with the integer
* the same overflow error strings
* the same ES-module exports

Those are implementation decisions, not requirements imposed by the PS4 or JavaScript.

The same pattern appears in `core.js`.

Raw13G contains the same unusually specific identifiers and structures, including:

* `DUPLICATE_INDEX`
* `CONTROL_INDEX`
* `CONTROL_INT`
* `FILLER_BIGINTS`
* `FILLER_OBJECTS`
* `EXPECTED_LENGTH`
* `CELL_BYTES`
* `FUNCTION_BYTES`
* `NATIVE_EXECUTABLE_BYTES`
* `HOLDER_BYTES`

It also contains the same distinctive function structure, including:

* `leakScopeObject`
* `prepareSymbolWrapper`
* `buildFakeHost`
* `buildAndStoreGraph`
* `prepareAddrof`
* `fillRawCellPointers`
* `loadHistoryCritical`
* `runGroomAndLoad`
* `beginComposition`
* `reportComposition`

And it retains the same unusual exploitation architecture, including the `history.replaceState()` object graph construction, fake-cell/carrier design, and the same named grooming stages.

That combination is the evidence.

## 4. This isn't claiming Raw13G did nothing original

Raw13G does contain modifications.

For example, it changes things such as grooming values, carrier geometry, firmware-specific values, and adds surrounding components such as `mem.js` and `jb.js`.

The claim is narrower:

> **Raw13G's `core.js` and `int64.js` are derivatives of my OOMfie/OnePS implementation, rather than independent implementations.**

## 5. Why the specific names and values matter

Someone independently implementing the same general exploit could obviously arrive at:

> "I need a 64-bit integer class."

That alone means nothing.

But independently arriving at:

> the same unusual property names,
> the same helper function,
> the same method names,
> the same backing-store behavior,
> the same error messages,
> the same export structure,
> the same fake-cell architecture,
> the same function names,
> the same object layout constants,
> and the same grooming terminology

is a completely different kind of overlap.

The evidence is therefore not simply:

**"Raw13G uses the same exploit idea."**

It is:

**"Raw13G contains implementation decisions that were already present together in MY's published source."**

## 6. The chain in one picture

```text
Earlier SSV research
        │
        │ underlying research
        ▼
   Bel / OOMfie
        │
        │ concrete implementation
        │
        ├── core.js
        ├── int64.js
        ├── fn_leak.js
        └── ps4_offsets.js
        │
        ▼
     OnePS
        │
        │ implementation lineage
        ▼
     Raw13G
        │
        ├── core.js   ← substantial source-level overlap
        ├── int64.js  ← highly specific matching implementation
        ├── mem.js
        ├── jb.js
        └── other files
```

## Bottom line

You do **not** need to understand JavaScript exploitation to understand the argument.

The argument is not:

> "They made something similar."

It is:

> **"Code published in OOMfie/OnePS contains a very specific collection of implementation choices. Raw13G contains that same collection of choices, including unusual names, structures, behaviors, and even exact strings."**

The historical OOMfie commit establishes that this implementation existed publicly in source before the later documentation changes. The current Raw13G source contains the matching implementation patterns.



# Provenance: Raw13G's `core.js` and `int64.js` are derivatives of OnePS/OOMfie.

**The claim, stated once:** The `core.js` and `int64.js` currently published at `raw13g/raw13g.github.io` are source-level derivatives of the implementation published by bel (`thebelx/OnePS`, `thebelx/oomfietest`).

---

## 1. The `int64.js` fingerprint

`int64.js` is the cleanest evidence because the implementation contains a combination of details that no independent implementation would reproduce by chance.

The historical OOMfie `int64.js`, introduced in commit `3d7955d7b85982d5f6160adf9ba6cd56278fc300`, contains:

```js
function zeroFill(number, width) { ... }

function int64(low, hi) {
    this.low  = (low >>> 0);
    this.hi   = (hi  >>> 0);
    this.backing = null;

    this.add32inplace = function (val) { ... }
    this.add32        = function (val) { ... }
    this.sub32        = function (val) { ... }
    this.sub32inplace = function (val) { ... }
    this.and32        = function (val) { ... }
    this.and64        = function (vallo, valhi) { ... }
    this.toString     = function (radix = 16) { ... }
    return this;
}

globalThis.int64 = int64;
export { int64 };
export default int64;
```

Two properties of this implementation are distinctive on their own:

1. **The `backing` property.** It is a `Uint8Array` view that advances when the integer value advances. `add32inplace` re-slices `this.backing` so arithmetic on the address also repositions a byte-level view into the same buffer. That pattern is not a standard 64-bit integer helper. It is specific to a carrier design where the address and the byte window must move together.

2. **The exact error strings.** `"int64.add32inplace: overflow"` and `"int64.add32: overflow"`. These are not conventional names. They are specific strings written by a specific author.

Both properties are present, unchanged, in Raw13G's current `int64.js`.

The full fingerprint — identical property names (`low`, `hi`, `backing`), identical method names, identical helper (`zeroFill`), identical backing-store technique, identical overflow strings, identical ES-module export structure (`globalThis.int64 = int64; export { int64 }; export default int64;`) — is not coincidence. It is a copy.

**Source links:**
- OOMfie at `3d7955d`: https://github.com/thebelx/oomfietest/blob/3d7955d7b85982d5f6160adf9ba6cd56278fc300/int64.js
- Raw13G current: https://github.com/raw13g/raw13g.github.io/blob/main/int64.js

---

## 2. The `core.js` overlap

`core.js` is 1,416 lines in the OOMfie implementation at `3d7955d`. The overlap with Raw13G's `core.js` is architectural, not cosmetic.

The following identifiers appear in both implementations, in the same roles:

```
K
DUPLICATE_INDEX
CONTROL_INDEX
CONTROL_INT
FILLER_BIGINTS
FILLER_OBJECTS
EXPECTED_LENGTH
CELL_BYTES
FUNCTION_BYTES
NATIVE_EXECUTABLE_BYTES
HOLDER_BYTES
```

The following function names appear in both implementations, in the same call sequence:

```
leakScopeObject
prepareSymbolWrapper
buildFakeHost
buildAndStoreGraph
prepareAddrof
fillRawCellPointers
loadHistoryCritical
runGroomAndLoad
beginComposition
reportComposition
```

The following structural choices appear in both implementations:

- Duplicate-reference graph assembled via `history.replaceState(...)`
- Same reference target placed into multiple array positions before deserialization
- The same fake-cell / carrier architecture with a `backing` view
- The same parameterized grooming system with the same named components: `drain`, `slab`, `butterfly hole`, `separator`, `early hole`, `guard`, `predecessor`, `final hole`

None of these are required to exploit SSV. They are decisions made by a specific implementation. They are present in both.

**Source links:**
- OOMfie at `3d7955d`: https://github.com/thebelx/oomfietest/blob/3d7955d7b85982d5f6160adf9ba6cd56278fc300/core.js
- Raw13G current: https://github.com/raw13g/raw13g.github.io/blob/main/core.js

---

## 3. The commit timeline

The implementation is not a reconstruction from a later README. It is in the source history.

**`3d7955d7b85982d5f6160adf9ba6cd56278fc300`** — parent `6d5912b`. Adds five files: `core.js`, `fn_leak.js`, `index.html`, `int64.js`, `ps4_offsets.js`. 2,368 additions, 1,607 deletions.

**`2a13faa071b244e287c250685b8a9224443a9272`** — parent `eb214df`. Titled "stealth push." Changes exactly one file: `readme.md`. 322 additions, one deletion.

The "stealth push" is not the introduction of the implementation. It is a documentation commit. The implementation was already in the ancestry.

This distinction matters because it forecloses the "he wrote it himself after seeing the README" defense. The source was public before the README existed.

**Source links:**
- Introduction commit: https://github.com/thebelx/oomfietest/commit/3d7955d7b85982d5f6160adf9ba6cd56278fc300
- "Stealth push": https://github.com/thebelx/oomfietest/commit/2a13faa071b244e287c250685b8a9224443a9272

---

## 4. The `int64` / carrier relationship

The `int64` class is not a standalone utility in the OOMfie implementation. It is wired into the carrier.

The historical OOMfie `int64.js` documents this directly:

> The backing-store trick: int64 can hold a Uint8Array in `.backing`. When it does, arithmetic that moves the numeric value also advances the backing view...

That comment is describing the carrier design in `core.js`. The two files are coupled. Copying one without the other would break the mechanism. Raw13G's files retain both the mechanism and the coupling.

---

## 5. What Raw13G modified

The claim is not byte-for-byte duplication. Raw13G modified:

- Heap-grooming values (`DRAIN_COUNT`, slab sizes, hole sizes)
- Carrier geometry
- Firmware offsets
- The surrounding memory-management layer (`mem.js`)
- The surrounding jailbreak layer (`jb.js`)
- Payload handling

Those modifications are real. They are also exactly what a derivative looks like when it is retuned for a different target — the substrate is inherited, the tuning is local. The existence of local modifications does not break the provenance claim; it is the expected shape of the derivative.

The files that are not claimed as derivative: `mem.js`, `jb.js`, `ps4_offsets.js`, `rpc_worker.js`, `payload2.bin`. Those may contain original work. The provenance claim covers `core.js` and `int64.js`.

---

## 6. What is not being claimed

This document does not claim:

- Bel invented the underlying SSV research. OnePS credits Hassan. That credit stands.
- Every file in Raw13G was copied from Bel.
- `mem.js` or `jb.js` were copied.
- The firmware offsets were copied.
- Raw13G contributed no original work.

Those would each require separate evidence. This document is narrow.

---

## 7. What the evidence cumulatively establishes

Any single overlap is dismissible. `low`/`hi` is a common 64-bit representation. `add32` is a common method name. A `Uint8Array` is a common buffer. None of these alone proves anything.

The claim rests on the combination:

1. Same object representation (`low`, `hi`, `backing`)
2. Same helper (`zeroFill`)
3. Same method names
4. Same arithmetic structure
5. Same backing-store technique
6. Same overflow strings
7. Same export structure (`globalThis.int64 = int64; export { int64 }; export default int64;`)
8. Same carrier assumptions (backing view advances with address)
9. Same SSV graph architecture (`history.replaceState` with duplicate references)
10. Same unusual object layouts (`K`, `DUPLICATE_INDEX`, `CONTROL_INDEX`, `CONTROL_INT`, `FILLER_BIGINTS`, `FILLER_OBJECTS`)
11. Same grooming architecture (`drain`, `slab`, `butterfly hole`, `separator`, `early hole`, `guard`, `predecessor`, `final hole`)
12. Same file decomposition (`core.js`, `int64.js`, `fn_leak.js`, `ps4_offsets.js`)

The probability that two independent implementations reproduce all twelve is not small. It is negligible.

---

## 8. The provenance chain

```
Hassan
  │   original SSV research
  ▼
OnePS / OOMfie — bel
  ├── core.js       (SSV primitive, carrier, grooming)
  ├── int64.js      (backing-store integer)
  ├── fn_leak.js
  └── ps4_offsets.js
  │
  ▼
Raw13G
  ├── core.js       (derivative, retuned)
  ├── int64.js      (derivative, unchanged structure)
  ├── mem.js        (not claimed)
  ├── jb.js         (not claimed)
  └── ...
```

First arrow: research attribution.

Second arrow: source lineage.

The two are distinct. Conflating them is a category error, and this document does not.

---

## 9. Evidence table

| Artifact | Repository | Evidence |
|---|---|---|
| `int64.js` | OOMfie | Introduced at `3d7955d7b85982d5f6160adf9ba6cd56278fc300` |
| `core.js` | OOMfie | Introduced at `3d7955d7b85982d5f6160adf9ba6cd56278fc300` |
| `int64.js` | Raw13G | Present at `main`; same structure, strings, exports |
| `core.js` | Raw13G | Present at `main`; same identifiers, functions, architecture |
| `int64.js` | OnePS | Present at `main` |
| `core.js` | OnePS | Present at `main` |
| `2a13faa` | OOMfie | Later commit; modifies only `readme.md` |
| SSV research | OnePS README | Credits Hassan |

**Primary references:**

- https://github.com/thebelx/oomfietest/commit/3d7955d7b85982d5f6160adf9ba6cd56278fc300
- https://github.com/thebelx/oomfietest/blob/3d7955d7b85982d5f6160adf9ba6cd56278fc300/core.js
- https://github.com/thebelx/oomfietest/blob/3d7955d7b85982d5f6160adf9ba6cd56278fc300/int64.js
- https://github.com/thebelx/oomfietest/commit/2a13faa071b244e287c250685b8a9224443a9272
- https://github.com/thebelx/OnePS
- https://github.com/raw13g/raw13g.github.io
- https://github.com/raw13g/raw13g.github.io/blob/main/core.js
- https://github.com/raw13g/raw13g.github.io/blob/main/int64.js

---

## 10. What attribution would look like

At minimum, a header in each derived file:

```js
// Derived from the OnePS / OOMfie implementation published by bel.
// https://github.com/thebelx/OnePS
// https://github.com/thebelx/oomfietest
```

For `int64.js`, this is not optional. The file is a copy with the same structure, same helper, same method names, same overflow strings, same export lines.

For `core.js`, the attribution should name the lineage and note that Raw13G layered modifications on top.


---

## 11. Conclusion

Raw13G's `int64.js` is a source-level derivative of the OOMfie/OnePS implementation. Raw13G's `core.js` is a source-level derivative of the same implementation, with modifications.

Neither file can be characterized as an independent reimplementation of the same research. The overlap is in specific implementation details that are not required by the problem being solved — they are decisions of a specific author, and they are present in both projects.

The historical record is unambiguous: the implementation was public at commit `3d7955d7b85982d5f6160adf9ba6cd56278fc300`, before the later `readme.md`-only commit that carries the misleading "stealth push" title.

Fucking credit me.
