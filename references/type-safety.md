# Type safety and errors

## Domain types and checked operations

**C++20 baseline.** Explicit domain types prevent mixing equal-representation values such as bytes and
sample counts. Validate at the boundary and keep a valid-state invariant inside. `explicit` blocks
implicit conversion; it does not itself validate the supplied value. Avoid redundant validation only
when a type's construction/access policy really excludes invalid state.

```cpp
#include <cstddef>
#include <stdexcept>

/** Identifies an interval that lies within a supplied extent.
 * Validation avoids overflow by subtracting only after checking the start. */
struct CheckedInterval {
    std::size_t first; ///< Valid starting offset in the supplied extent.
    std::size_t count; ///< Length bounded by the remaining extent.
};

/** Validates offsets before an interval is used to form a view.
 * The returned aggregate is a validated result, not an unforgeable proof type. */
[[nodiscard]] inline CheckedInterval checked_interval(
    std::size_t extent, std::size_t first, std::size_t count) {
    if (first > extent || count > extent - first) {
        throw std::out_of_range("interval exceeds extent");
    }
    return {first, count};
}
```

A private constructor would be needed if the application wants an unforgeable validated type. Even
then, subsequent owner resizing can invalidate an interval's applicability; validation belongs to a
specific extent/version. Do not treat validation as timeless authorization to access mutated storage.

For `count * stride`, establish `count <= max / stride` when stride is nonzero before multiplying.
Widen both operands before arithmetic, not only the result. `std::integral` includes types whose domain
is unsuitable for a particular algorithm; constrain meaningful domains and check runtime ranges.

**Since C++20:** `std::in_range<Destination>(value)` can check integer conversion suitability.
A cast changes representation without creating that proof. Floating-to-integer boundaries need a
separate finite/range/rounding contract; integer helpers do not establish it.

## Failure representation and comparison

**C++23:** choose `expected<T,E>` when callers handle recoverable domain failures. **C++20 fallback:**
use a project result type or `variant<T,E>` with a distinct error type. Do not use success/error
alternatives of the same ambiguous type. Propagate exceptions according to policy; an expected result
does not catch allocator failures or exceptions from callbacks by itself.

**Since C++20:** defaulted comparisons derive ordering from members. Floating fields produce partial
ordering in ordinary cases; NaN can violate a sorting predicate's semantic requirements. Define NaN
rejection or a documented total-order policy before ordered lookup/sorting. Concepts cannot verify
strict weak ordering dynamically. Equality and ordering policies must remain coherent for the domain.

**C++23:** `to_underlying` clarifies enum representation. **C++20 fallback:** `static_cast` to the
underlying type at a narrow serialization boundary; validate input values before constructing domain
state. A scoped enum does not validate arbitrary numeric casts.

Core topics: [I.4, ES.46, ES.103, T.20](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines).
