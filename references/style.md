# Naming and structure

## Naming and interface design

**C++20 baseline policy.** Naming should make temporal and ownership differences visible. A function
called `snapshot` should return owned/stable state; `view` should disclose borrowing. A `close` operation
that only requests closure needs that distinction documented. A unit suffix helps a local variable,
but distinct quantity types prevent errors across interfaces. Avoid names that promise lock-free
progress or constant-time work without a verified contract.

Follow the established project convention. The fallback in AGENTS.md is local policy, not a claim
that the Core Guidelines mandate PascalCase or snake_case. Do not rename a public symbol merely to
align with a new private convention; migrations can break callers and ABI.

Relevant [C++ Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines):

- **NL.8:** consistent naming style; a coherent local convention matters more than a universal spelling.
- **NL.10:** prefer underscore-separated names when choosing that style; do not mix policies.
- **F.2:** an operation should implement one logical purpose; isolate validation from unrelated I/O.
- **F.16/F.18:** distinguish reading from consuming arguments; a reference does not imply ownership.
- **C.2:** choose a class when there is an invariant; coupled length/storage must not be freely mutable.
- **C.20:** use rule-of-zero members when they preserve the actual invariants.
- **C.21:** if special members are needed, evaluate the complete copy/move/destruction contract.
- **SF.7/SF.11:** avoid namespace pollution in headers and make headers self-contained.

## Type visibility and interface shape

**C++20:** choose explicit types where deduction changes meaning. `auto count = 0` chooses `int`;
`auto count = input.size()` preserves the container size type. Neither is universally preferable:
use checked conversion when crossing a bounded signed domain. `const auto&` avoids copying an element
but can bind a proxy, not the expected value reference. Check proxy-containing ranges explicitly.

Use aggregates for independent settings; C++20 designated initializers must follow declaration order.
Do not expose an aggregate when callers can construct invalid combinations. Export minimal dependency
surfaces: a template placed in a public header exposes requirements and compile-time cost to clients.

**C++23:** explicit object parameters can reduce cv/ref overload duplication. **C++20 fallback:**
ref-qualified overloads sharing a helper. Avoid returning borrowed references from an rvalue owner.
A convenience overload must not silently weaken a lifetime guarantee.
