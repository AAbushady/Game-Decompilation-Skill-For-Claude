# Exact-match lever catalog

A reasoning aid, not a plan template or a menu of blind mutations. Use an entry only when retail or source
evidence licenses it, and mark the family "checked" when it does not apply.

Every lever here produces source that a programmer of the era would have written. That is what makes it a lever
and not a trick. A spelling that closes a gap only through a form in `decomp-code-rules` Section F (a dummy
local, `volatile`, an empty block, a reused variable, an offset cast) is not a closure. It points at one of the
families below, usually 3 or 4. Compiler-specific mechanisms
(inline budgets, register-allocator quirks, tail-merging regimes) belong in the project's own pattern notes; the
entries here hold across the MSVC x86 era and usually beyond it.

## 1. Recover source before steering the compiler

- **Adopt surviving source whole.** Take a complete surviving owner (engine, SDK sample, sibling game) as a
  composition: declaration, hierarchy, nested types, helpers and macros, locals, statement order, loops and
  storage. A lower first score is not a rejection.
- **Sibling evidence:**
  - Search repeated sibling implementations and sibling binaries, such as other games on the same engine.
  - Use them positively, for family, topology, widths and names; not as negative proof when revisions differ.
- **Debug builds:** where paired Debug/Release objects or a symbolized build exist, mine them for locals,
  scopes, types and source paths. Compile what you recover in the real unit.
- **Lost headroom:** recover the old source that matched better before proposing new source. Recoveries have
  turned up spurious out-parameter initialisers, a deleted frame local and deliberate structural trades.
- **Ownership and receivers:** recheck function and unit ownership, class membership, calling convention and
  static receiver type.
  - Exact callee bytes can hide a dropped `this`.
  - A dead ECX load before a call means members are missing.
  - A wrong virtual-slot displacement means a wrong static receiver type.

## 2. Types, layout, aggregates and identity

- **Widths:** propagate parameter and local widths through the whole call family:
  - `unsigned char` versus `bool` or `char`;
  - signed versus unsigned;
  - 16-bit destinations;
  - enum domains;
  - dirty upper bytes;
  - pointer versus integer;
  - real SDK types.
- **cv/ref and overloads:** const forwarding wrappers, `T&` accessors, const-reference results and by-value
  adapters can disappear from the output while still changing caller temporaries and evaluation order.
- **Aggregates:**
  - Restore them only from complete-object evidence: rectangles, points, vectors, ranges, palettes, wire
    records, paired clock fields.
  - By-value aggregate parameters, per-call return slots and whole-aggregate clears or copies are strong
    signals.
- **Repeated fields:** same-width fields are candidates, not proof. Prefer a surviving typed union or class
  family over a layout-compatible `void*` or an overlay.
- **Identity:** audit base, subobject and member identity. Identical masked bytes with a different displacement,
  vptr target or relocation name are correctness defects.
- **Forward declarations:** `class` versus `struct` changes the mangled referent even when the code is identical.

## 3. Inline, helper and macro boundaries (mandatory search)

For every function, search headers, sibling sites, surviving source, COMDATs and repeated expressions for an
authored boundary. Do not assume that emitted arithmetic was handwritten.

- **Restore tiny helpers:** accessors, setters, getters, `operator[]`, range-guarded `GetAt`, conversions,
  constructors, assignment operators, member predicates, min/max/clamp, point-containment and
  coordinate-returning accessors.
- **Preserve the helper's receiver form and full signature.** A member versus a free helper changes operand order;
  passing an object versus scalars changes load timing; by-value narrow parameters keep values narrow.
- **Preserve the helper body's store order.** An expansion can recolour caller locals before the first visible
  helper instruction.
- **An inline helper can own an out-parameter local and its reset.** A zero store between argument setup and the
  call, or a different `lea` home, shows that the local belongs inside the helper.
- **Repeat pure inline expressions when source identity matters.** Caching one in a local enables CSE and changes
  lifetimes. Conversely, remove an unjustified one-use local when retail keeps the cursor or receiver.
- **Restore authentic macros** for standard algorithms and pixel or maths families: cipher rounds, pixel packing,
  rectangle helpers, palette maths.
- **Visibility is not eligibility.** Know the compiler's inline policy:
  - which functions expand without `inline`;
  - whether unmarked `static` helpers expand;
  - how template members behave.

  A body can be expanded at one site and called at another; inspect the actual call referents and the COMDATs
  each object emits.
- **Treat a per-caller inline budget as a population problem**, only after the candidate helpers are proved
  independently. Missing one-field helpers change expansion populations and EH states. Never keep cost padding
  or forcing devices.
- **A pure register-colour difference may still be one missing inline IL tuple.**

## 4. Local census, lifetime, scope and creation order

- **Recover the exact locals** instead of caching every member. Each local has one meaning and a name for it
  (`decomp-code-rules` R30); a decompiler's merged variable is usually two. Test:
  - distinct old/new/result variables;
  - parameter reuse;
  - one result reused through setters;
  - removing one-use call-result locals.
- **Check declaration order and first-use order separately.** Creation order can decide register preference,
  spill-slot reuse, and which call-crossing value gets a callee-saved register.
- **Initialisation placement:**
  - a declaration initialiser versus a later store;
  - initialisation in a `for` header;
  - a deliberately uninitialised local;
  - a late store after the value's first consumer.
- **Scope is emitted structure:**
  - Sibling block locals may share one home; function-scope locals cannot.
  - Distinct per-arm address-taken outputs may need distinct homes.
  - A second-half local at function scope can prevent frame sharing.
- **Compare the frame size and every address-taken `lea`.** A frame difference may be a real local, an unnamed
  temporary, the reuse of a dead parameter's home, or a spill; classify which before editing.
- **Pointer and base locals:** test them where retail folds a subobject offset into one `lea` or reuses a receiver
  across stores. Remove aliases where retail advances the parameter itself or reloads instead of caching.
- **By-value accessors can need unnamed temporaries.** Naming the temporary may let the compiler delete its
  dead stores; repeating `obj->Pos().x`-style calls can restore frame homes otherwise unreachable.
- **Named result references sequence code.** Keep them for stream and operator chains and other sequencing
  boundaries.
- **Payload locals:** keep them around varargs calls when retail homes the surrounding loop state; remove copy
  locals when they alone cause a spill.

## 5. Statement and expression shape

- **Test one expression against sequenced assignments**, and direct member stores against a result reused through
  setters. The front end keeps statement boundaries even after the machine code looks folded.
- **Put each computation in the statement where retail emits it.** Naming an outer call argument, moving `+K` into
  an initialiser, or grouping constructor arguments changes scheduling.
- **Preserve observable evaluation and operand order**, but do not infer source order from commutative
  expressions the compiler canonicalises.
- **Test the alternatives:**
  - a ternary versus a split `if`;
  - a short-circuit expression versus a materialised bool;
  - an in-place compound update versus a copy and write-back;
  - a clamp ternary versus init-then-if.
- **Order stores the same way in both symmetric arms.** Equal operations in different statement groups can
  decide cross-jumping.
- **Keep authentic idioms:** `abs()` rather than hand-expanded sign masks, cumulative numerators rather than
  recomputed products, explicit accumulation rather than one reassociable OR tree, correct parentheses around
  division.
- **Use the era's library abstraction:** `memcpy`, `memset`, `strlen`, `strcmp`, framework conversions, Win32
  helpers or the authored wrapper. Their inline forms create masks, REP setups, dead-looking scans and temporary
  homes.

## 6. Guards, exits and block placement

- **Count before calling it regalloc.** Count calls, conditional branches, unconditional jumps and returns
  before calling a wall register allocation. Match destinations and block placement, not just the multiset.
- **Test the exit styles:** separate early returns, `goto fail`, total `||`/`&&` collapse, and a positive success
  region. They are distinct tail-merging regimes; nesting the successful region under a guard can leave the
  shared failure epilogue last.
- **Result carriers:** a state/result carrier with one shared exit differs from duplicated returns. A fabricated
  `default: return` or `else` can pin the prologue and add an exit retail lacks.
- **Preserve staged failure ownership** around allocation, initialisation and insertion; early-failure
  partitioning shapes the later cleanup tails.
- **Check guard polarity**, compare-operand order, and positive versus negative nesting. Cross-jumped call arms
  reveal their source order through the different constants they push.
- **Scan and retry shapes:**
  - a scan may null its cursor rather than `break`;
  - a retry may reuse one variable and one return;
  - branch-local null checks may share only the success tail.
- **Constructors, destructors and EH arms are special:** unwind targets, the called base constructor, the
  cleanup owner and the EH-state count can prove the authored layer.

## 7. Loops and cursor ownership

- **Loop form:** test `while`, guarded `do/while`, `for`, infinite loop plus `break`, and duplicated `continue`
  paths against retail's entry and latch shape.
- **Count-down guards:** test post-decrement forms; `while (n-- > 0)` materialises `n-1`.
- **Reproduce update order**, including comma increments, for example counter before pointer in nested loops.
- **Decide who owns the cursor:**
  - the parameter itself;
  - an interior pointer local;
  - an array cursor that carries the member bias;
  - indexed access.
- **An entry trampoline is a readout of value ownership or pressure**, not a request to respell the loop: check
  hoisted invariants, one-use locals and cross-arm aliases first.

## 8. Allocation, destruction and storage operations

- **Prefer source `new`/`delete` (and the array forms)** when object ownership supports them. Explicit allocator
  or destructor calls need evidence: their cleanup, EH and scratch registers differ even when the final frees
  match.
- **Audit constructor chain depth** and per-site inline cuts with EH states and call referents.
- **Test the alternatives:**
  - a guarded delete plus a null store;
  - a direct member out-parameter versus a local plus copy;
  - `memset`, a fixed loop, or manual stores.
- **Keep typed teardown adapters** and container seams that surviving source proves.

## 9. Data, constants, referents and unit state

- **Audit ordered relocations**, not just masked instructions. A wrong callee, pooled literal, member or global
  identity, FP constant slot or DIR32 addend can be the whole residue.
- **Literals:** use bare string and FP literals when string pooling covers them. Duplicate file-static char arrays
  are wrong under pooling.
- **Named constants:** a `#define` or enum folds to the same immediate. In C, a `const int` is an object, and
  unoptimized builds load it from memory. A `const float` may get its own storage instead of the pooled literal.
  `sizeof` division is unsigned (`decomp-code-rules` R11).
- **Static ownership:** check local static, unit global or header COMMON ownership, and unit order for
  first-function EH or dynamic-initialiser effects.
- **Data layout:** some compilers order `.bss` or `.data` by a hash of the identifier, so names and declaration
  order can be evidence too. Check the project's notes.
- **Masked diffs:** a masked diff can hide wrong branch destinations; inspect raw displacements and the normalised
  CFG.
- **Compiler-state probes come last.** Only after every structural family is disposed of may a classified
  register/schedule wall use deterministic source variants or disposable declaration- or unit-state probes.
  Bank exact unchanged-source states, remove the probes, and keep only the proof.

## 10. Stop signals and negative controls

- **Pure register colour:** a pure destination-register colour difference, with identical IL and no source
  entity, may be cursor phase. Do not manufacture a local to request a register.
- **Proven-flat controls:** commutative term order, local names under optimisation, some declaration-order
  changes and some cursor spellings have proven flat in specific signatures. Treat them as controls, not
  universal laws. Verify each on your compiler; some compilers, for example, assign stack slots by name.
- **Inline library expansions** (`strcmp` and the like) can pick a longer form from register availability
  alone; fix the pressure source elsewhere.
- **Downstream counts:** REP counts and branch totals can be consequences of tail merging or allocation; confirm
  the earlier divergence before changing counts or CFG.
- **Dips:** a current dip below a banked best is no reason to distort correct source. A source change that lost a
  historical exact is, however, a mandatory history investigation.

An exact closure does not need a new catalogue entry. Add only distinct reusable mechanisms, with bounded A/B
evidence; function-specific experiment history belongs in the handoff and version control.
