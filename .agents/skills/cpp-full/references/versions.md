# Versions and C++20 alternatives

## Contents

- [Selection and support](#selection-and-support)
- [C++20 facilities](#c20-facilities)
- [C++23 alternatives](#c23-alternatives)
- [C++26 alternatives](#c26-alternatives)
  - [C++26 adoption checks](#c26-adoption-checks)
- [Feature probes](#feature-probes)

## Selection and support

Language version, library version, operating system support, and build integration are distinct. Check each selected feature. Do not use blanket claims such as “Clang version X supports C++23.” A Clang installation can use libc++ or libstdc++ from a different release.

For C++26, check the current draft and target implementation. Do not interpret an experimental extension as portable support. Preserve a C++20 implementation when the compatibility contract requires it.

CMake, sanitizers, profilers, and vendor hardening modes are **tooling**, not C++ language features. Their workflows support C++20 and later; state the tool/platform requirement instead of inventing an ISO introduction version.

## C++20 facilities

| Since C++20 | Useful decision | Limitation to retain |
|---|---|---|
| Concepts and `requires` | Constrain the actual operation at an API boundary | Semantic requirements need documentation and tests |
| Ranges algorithms and views | Compose traversal and use projections | Views can borrow and repeat work |
| `std::span` | Pass contiguous storage with its extent | No ownership; `operator[]` is not portable bounds checking |
| `std::jthread`, stop tokens | Join automatically and request cooperative stop | Cancellation does not interrupt arbitrary blocking I/O |
| Atomic wait/notify, `atomic_ref` | Wait for value changes; access existing suitably aligned storage atomically | Lifetime and access discipline remain necessary |
| `latch`, `barrier`, semaphores | Express one-shot completion, repeated phases, permit counts | Participants and shutdown must balance |
| `std::format`, `std::osyncstream` | Typed formatting; coordinated stream records | Library support and participating writers matter |
| `<=>`, designated initialization | Memberwise ordering; readable aggregate configuration | Floating NaNs and declaration order matter |
| `consteval`, `constinit`, expanded `constexpr` | Require compile-time computation; prevent dynamic initialization | `constinit` does not make an object immutable |
| `std::bit_cast`, endian, integer comparison helpers | Read representation and compare signed/unsigned safely | They do not validate external data or convert wire order |
| `std::erase_if` | Remove values without invalidated-iterator traversal | Container-specific invalidation still applies |
| Coroutines and modules | Model suspension; separate exported interfaces | No standard async runtime in C++20; build support must be proven |

## C++23 alternatives

| Since C++23 | When useful | Best C++20 baseline |
|---|---|---|
| `std::expected<T,E>`; monadic operations | Return recoverable domain failures | Existing project result type; otherwise `variant<T,E>` behind a small typed API; explicit branching for chaining |
| Monadic `optional` | Transform an optional result | Explicit presence test; retain absence without inventing an error |
| `std::print` / `println` | Typed formatted output | `std::format` plus stream output; existing formatting dependency if `<format>` is missing |
| `ranges::to`, range constructors | Materialize a view | Range-for with `push_back`; reserve only for a cheap valid size |
| `views::zip`, `enumerate` | Pair ranges or attach indices | Iterator loop stopping at the shortest range; explicit counter |
| `views::chunk`, `slide`, `stride` | Process grouped or sampled elements | Checked span subviews or explicit iterator loops respecting range category |
| `views::join_with` | Flatten with separators | Append each range with an explicit separator rule |
| `ranges::contains`, `contains_subrange` | Membership queries | `ranges::find != end`; `ranges::search` with an explicit empty-needle rule |
| `ranges::fold_left` | Ordered accumulation | Explicit accumulator loop; avoid unordered `reduce` when order matters |
| Explicit object parameters, `forward_like` | Reduce cv/ref overload duplication | Ref-qualified overloads and a shared helper; self-parameter lambda for recursion |
| `std::move_only_function` | Store owning move-only callbacks | Existing owning callable wrapper; `packaged_task` for a one-shot future task, not a general substitute |
| `std::scope_exit` / `scope_fail` / `scope_success` | Local rollback or exit action | Existing scope guard or a narrow RAII transaction object; preserve exception semantics |
| `std::mdspan` | Describe multidimensional storage and strides | Owning storage plus a checked span-based shape/stride wrapper |
| `std::flat_map` / `flat_set` | Read-heavy sorted lookup | Sorted vector plus `lower_bound`, or existing vetted flat container; use `map` for stable nodes |
| `std::generator` | Lazy synchronous coroutine traversal | Range/view, callback producer, or existing C++20 coroutine library |
| `if consteval` | Separate immediate and runtime paths | `is_constant_evaluated`; it cannot grant the same immediate-function context |
| `std::byteswap`, `to_underlying` | Byte order and enum representation | Unsigned shift/mask routine; explicit cast to `underlying_type_t<E>` |
| `import std` | Import standard library modules | Self-contained headers; C++20 language modules alone do not supply `std` |
| Range-for temporary lifetime extension | Safer nested temporary expressions | Name the owning object in a loop init-statement; still reject views into destroyed by-value parameters |

Check `expected` monadic support separately from basic `expected`. Keep fallback semantics visible at public boundaries.

## C++26 alternatives

These entries refer to C++26 draft facilities. Verify exact spelling and support before adoption.

| C++26 facility | Best C++20 baseline and difference |
|---|---|
| Static reflection and expansion statements | Explicit descriptors or build-time generation; less automatic, preserves reviewed schema |
| Contracts | Explicit runtime validation for external inputs; documented preconditions and assertions for internal bugs; no contract metadata |
| Standard-library hardening | Vendor-supported hardening plus explicit boundary checks; coverage is implementation-specific |
| `span::at` | Bounds check before indexing; throws or returns a typed error per project policy |
| Erroneous-value rules | Initialize storage and validate reads; no assumption that C++26 makes uninitialized reads safe |
| `std::execution` senders/receivers | Existing C++20 async library or bounded tasks with explicit result/error/cancellation channels |
| `inplace_vector` | `array<T,N>` plus count for suitable always-live elements, or vetted static-vector; `vector::reserve` is not a no-heap guarantee |
| `hive` | Stable-node container or vetted stable pool; iterator order and complexity may differ |
| `function_ref` | Constrained forwarding callable for synchronous use; existing borrowed wrapper if a fixed type-erased ABI is needed |
| `indirect`, `polymorphic` | Value member where possible; `unique_ptr` with explicit deep-copy/clone operations |
| `atomic::fetch_min` / `fetch_max` | Tested compare-exchange loop with specified ordering |
| Saturating arithmetic | Overflow-checked unsigned arithmetic or vetted numeric library; keep saturation separate from error reporting |
| `<simd>`, `<linalg>` | Autovectorized span loops, vetted SIMD/BLAS libraries, or isolated intrinsics with scalar fallback |
| Pack indexing | `tuple_element_t<I, tuple<Ts...>>`; compile-time complexity can differ |
| Fold-expanded constraints | Explicit named concept refinement when overload ordering is required |
| `views::concat` | Visit ranges sequentially; materialize only if a single owned range is necessary |
| `#embed` | Build-time generator producing a source array with a tracked input dependency |
| Deleted-function reason, name-independent declarations | `= delete` with a comment; distinct descriptive local names |

### C++26 adoption checks

Apply these checks in addition to the fallback matrix; they capture differences that a feature name
or successful compilation does not establish.

- **Reflection:** verify reflection values, splicing, expansion support, and access context in the
  deployed implementation. Keep serialized field selection, names, ordering, privacy, and schema
  versions explicit. C++20 descriptor/generator output must preserve that reviewed schema.
- **Contracts:** keep predicates side-effect-free and safe to evaluate. Do not use assertion evaluation
  to perform required work or assume continuing after a violation repairs state. C++20 explicit
  validation must remain active for recoverable external-input errors, including release builds.
- **Hardening and erroneous values:** verify the operations actually covered and enforcement mode.
  Neither hardening nor erroneous-value treatment makes invalid access/uninitialized reads safe.
  Do not apply `[[indeterminate]]` as a routine optimization. C++20 fallback: explicit validation and
  initialized storage, with supported vendor hardening where useful.
- **Storage and callables:** define fixed-capacity overflow and element-lifetime behavior; reserved
  dynamic storage is not embedded storage. Verify callable lifetime, type-erasure ABI/code-size cost,
  and deep-copy versus aliasing semantics. Use the matching C++20 fallback from the matrix.
- **Numerics:** check shape/stride, tail handling, runtime dispatch, and allowed rounding differences.
  Do not assume a library facility guarantees numerical equivalence or allocation-free execution.
- **Execution:** verify scheduler, cancellation propagation, result ownership, and operation-state
  lifetime. Constructing a sender does not start work or provide an I/O scheduler. C++20 futures do
  not automatically supply sender composition/cancellation; use the established task runtime.

Design sources: [reflection proposal](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p2996r13.html),
[contracts proposal](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p2900r14.pdf), and
[execution wording](https://eel.is/c++draft/exec). Proposal examples still need current implementation checks.

## Feature probes

**C++20 baseline workflow; `std::expected` itself is C++23.** Include `<version>` for library feature macros. Macro presence is only the first gate; compile and link the actual API with the project flags.

```cpp
#include <version>
#if defined(__cpp_lib_expected) && __cpp_lib_expected >= 202202L
#include <expected>
static_assert(std::expected<int, int>{7}.value() == 7);
#endif
```

For monadic `expected`, require the later feature-test value `202211L`. Do not use `__has_include` alone: a header can exist without the selected mode exposing the feature.

Primary implementation references: [Clang language status](https://clang.llvm.org/cxx_status.html), [libc++ C++23 status](https://libcxx.llvm.org/Status/Cxx23.html), and the selected standard library's own release documentation.
