# Sources and policy

## Source coverage and policy

The user-selected C++ Core Guidelines are the design authority, specifically [R.1](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#rr-raii), [CP.4](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#rconc-task), and [CP.3](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#rconc-data). The project bans raw-pointer creation/ownership and manual allocation. Prefer references for borrowed objects and bounded views for buffers; borrowed raw pointers remain permitted at required interfaces, consistent with R.2/R.3. Task-first async examples supersede the previous thread/queue/publication demonstrations. Standard wording and implementation documentation check version/support details.

This instruction set adapts `cpp-full` and its consolidation of cpp, cpp-coding-standards,
cpp-modern-features, cpp-pro, and modern-cpp. That consolidation reviewed all five entrypoints and
24 supporting references. The original sources remain unchanged. Selected version, concept, range,
and concurrency references are copied here so this instruction set does not depend on a skill
reference chain. C++26 adoption checks are consolidated into the version reference.
New memory, naming, performance, type-safety, test, build, and documentation references
supply the requested repository-specific policy and examples.

The project prefers values, references, containers, and bounded views. Borrowed raw pointers may
appear at required interfaces, but must not create/own objects or transfer a cleanup obligation.
Manual allocation is prohibited. Source examples are selected for practical value, not reproduced
merely because a borrowed pointer is now permitted.

No new tutorial section is devoted to an older standard. Established facilities remain when they
support C++20-and-newer designs, and their baseline label does not claim a C++20 introduction date.
Task-based design does not mean `std::async` is a pool or a future exposes task progress/cancellation.
Layout recommendations depend on measured access patterns rather than universal SoA superiority.

## Primary and supplied material

- [C++ Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines): topic rationale;
  this repository bans raw-pointer creation/ownership and manual allocation.
- [Concepts tutorial supplied by the user](https://www.geeksforgeeks.org/cpp/constraints-and-concepts-in-cpp-20/):
  requested concepts coverage; requirement semantics are checked against standard wording.
- [Requires expressions](https://eel.is/c++draft/expr.prim.req) and
  [template constraints](https://eel.is/c++draft/temp.constr): syntax, satisfaction, and ordering.
- [Async wording](https://eel.is/c++draft/futures.async) and
  [future members](https://eel.is/c++draft/futures.unique.future): result-state semantics.
- [Stop tokens](https://eel.is/c++draft/thread.stoptoken),
  [atomic operations](https://eel.is/c++draft/atomics.types.operations), and
  [execution](https://eel.is/c++draft/exec): concurrency contracts.

The supplied progressive-disclosure guide, updated to allow AGENTS.md below 500 lines and 20 KB,
sets the structure. All local references are reached directly from AGENTS.md using heading anchors.
The online draft includes later changes; select version-specific behavior and probe toolchain support.
