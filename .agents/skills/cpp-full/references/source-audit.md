# Source audit

## Contents

- [Coverage and selection](#coverage-and-selection)
- [Corrections](#corrections)
- [Technical sources](#technical-sources)

## Coverage and selection

The user-selected C++ Core Guidelines are the design authority, specifically [R.1](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#rr-raii), [CP.4](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#rconc-task), and [CP.3](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#rconc-data). The project bans raw-pointer creation/ownership and manual allocation. Prefer references for borrowed objects and bounded views for buffers; borrowed raw pointers remain permitted at required interfaces, consistent with R.2/R.3. Task-first async examples supersede the previous thread/queue/publication demonstrations. Standard wording and implementation documentation check version/support details.

The consolidation reviewed five source skills and all 24 supporting Markdown references. The original folders remain unchanged. Paths below describe source coverage, not runtime dependencies of cpp-full.

| Source skill | Files reviewed | Retained material |
|---|---|---|
| cpp | SKILL.md | C++20 interfaces, ownership, error policy, concurrency and build scope |
| cpp-coding-standards | SKILL.md; classes-and-resources, concurrency-and-templates, expressions-and-errors, interfaces-and-functions, library-and-style, performance-checklist | Relevant invariant, lifetime, semantic constraint, exception and measurement decisions |
| cpp-modern-features | SKILL.md; cpp11-14, cpp17, cpp20, cpp23, ownership-and-constexpr, review-checklist | C++20/23 features and useful lifetime/evaluation pitfalls; older tutorials excluded |
| cpp-pro | SKILL.md; build-tooling, concurrency, memory-performance, modern-cpp, templates | Feature probes, measurement, task lifecycle, allocator/SIMD decision criteria, constrained algorithms |
| modern-cpp | SKILL.md; anti-patterns, compiler-hardening, cpp20-features, cpp23-features, cpp26-features, safe-idioms | C++20/23/26 selection, checked arithmetic, hardening, migration and draft adoption |

Common naming, syntax tutorials, generic “write small functions” rules, persona text, mandatory blanket replacements, stale tool/version tables, and unsupported speedup claims were removed. Established RAII, borrowing, and result utilities remain where they are necessary for C++20-and-newer safety; they are identified as baseline guidance rather than newly introduced features.

## Corrections

Expert-agent reevaluation removed the generic implementation checklist, elementary comparison/constant-table/output examples, build skeleton, C-stream owner, pointer-based parser demonstration, and arena-copy example. Retained examples illustrate sentinel materialization, arithmetic bounds, exact constraints, subsumption, cancellation precedence, and atomic ordering. The current policy permits borrowed pointers at required interfaces while preferring references and bounded views; owning raw pointers and manual allocation remain prohibited.

- Replaced unsafe raw-node stack examples: immediate deletion is a reclamation/use-after-free problem, not only ABA.
- Removed coroutine owners that can be copied and destroy the same handle, resume completed frames, or silently lose exceptions. Guidance requires an established frame/runtime contract.
- Removed raw-buffer examples with failed-copy-assignment hazards and inconsistent moved-from size metadata. Prefer value members; document coupled move invariants.
- Removed a single-object pool allocator used with vector. Vector can request contiguous multi-element allocation.
- Removed SIMD loops with missing tail handling and unsupported fixed speedup claims.
- Corrected ranges materialization: iterator and sentinel types can differ.
- Replaced unconstrained or incomplete sorting predicates with random-access plus sortable requirements.
- Corrected claims that span automatically checks bounds, atomics are always lock-free, shared mutexes always win, or async is a thread pool.
- Corrected claims that constexpr functions make all runtime paths UB-free, and that uninitialized reads become safe in C++26.
- Added separate equality and NaN ordering guidance for spaceship operators.
- Removed blanket platform hardening commands and fixed overhead claims; tooling must match compiler, library, OS and mode.
- Corrected conjunction spelling (`&&`) and requirement semantics in the supplied tutorial. Practical examples include explicit headers and const-correct requirements.

## Technical sources

User-supplied concepts reference: [Constraints and Concepts in C++20](https://www.geeksforgeeks.org/cpp/constraints-and-concepts-in-cpp-20/). It supplies the requested topic coverage; its shorthand and examples are checked against language semantics.

Primary sources used to check selected technical rules:

- [Requires expressions](https://eel.is/c++draft/expr.prim.req)
- [Template constraints](https://eel.is/c++draft/temp.constr)
- [Stop tokens](https://eel.is/c++draft/thread.stoptoken)
- [Atomic operations](https://eel.is/c++draft/atomics.types.operations)
- [Execution](https://eel.is/c++draft/exec)
- [Reflection for C++26](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p2996r13.html)
- [Contracts proposal](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p2900r14.pdf)
- [Clang language status](https://clang.llvm.org/cxx_status.html)
- [libc++ C++23 status](https://libcxx.llvm.org/Status/Cxx23.html)
- [libc++ hardening](https://libcxx.llvm.org/Hardening.html)
- [Clang TSan](https://clang.llvm.org/docs/ThreadSanitizer.html)
- [CMake target features](https://cmake.org/cmake/help/latest/command/target_compile_features.html)

The online draft includes later-standard changes. Use version-specific wording and toolchain probes when implementing a baseline. Proposal examples are explanatory and must be checked against the selected implementation.
