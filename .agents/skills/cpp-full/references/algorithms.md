# Algorithms and language features

## Contents

- [Ranges and materialization](#ranges-and-materialization)
- [Traversal and invalidation](#traversal-and-invalidation)
- [Comparison](#comparison)
- [Constant evaluation](#constant-evaluation)
- [Formatting and output](#formatting-and-output)

## Ranges and materialization

**Since C++20:** ranges algorithms express operations on whole ranges and support projections. Views defer traversal; they do not cache every transformed value, and a view can own state or allocate through its underlying range. Check range category, borrowed lifetime, sentinel behavior, and evaluation count.

**C++23:** `ranges::to` materializes a range. **C++20 fallback:** iterate and append. Constructing a vector from `begin` and `end` does not work for all iterator/sentinel pairs. A `common` adapter is possible, but a loop is clearer when generic materialization is the only need.

```cpp
#include <cstdint>
#include <ranges>
#include <span>
#include <vector>

/** Copies positive samples as widened squares.
 * Materialization uses iteration so different sentinel types are supported. */
[[nodiscard]] inline std::vector<std::int64_t>
positive_squares(std::span<const std::int16_t> input) {
    auto selected = input
        | std::views::filter([](auto value) { return value > 0; })
        | std::views::transform([](auto value) {
              const auto wide = static_cast<std::int64_t>(value);
              return wide * wide;
          });
    std::vector<std::int64_t> output;
    for (const auto value : selected) { output.push_back(value); }
    return output;
}
```

This example is **C++20**. The bounded input type makes the square representable in `int64_t`. A concept such as `integral` alone would not establish that bound.

Use `reserve` when the count is known without consuming the input or repeating an expensive predicate. A filter can be traversed twice by “count then materialize”; measure before preferring that to one-pass growth.

**C++23:** `zip` ends at the shortest range and `enumerate` carries an index. **C++20 fallback:** explicit iterators and a counter. If equal lengths are a contract, check them separately; shortest-range traversal is not validation.

**C++23:** `mdspan` separates shape and strides from ownership. **C++20 fallback:** a span-backed wrapper with checked dimensions and multiplication. Validate storage extent before building it; neither descriptor creates the storage it describes.

## Traversal and invalidation

**C++20:** `erase_if` avoids manually advancing an invalidated iterator. It does not preserve references to removed or shifted vector elements. Never mutate predicate-relevant state while depending on a filter view's cached beginning without rebuilding the view.

Do not return iterators into temporary owners. Some ranges algorithms return `ranges::dangling` for non-borrowed temporaries; this protects the result type, not a separate view with an invalid owner.

**C++23:** range-for extends more temporary lifetimes in the range initializer. **C++20 fallback:** name the owning temporary in the loop's init-statement or before the loop. This also makes ownership easier to inspect. Neither version repairs a view returned from a function whose by-value parameter has already died.

Use a plain loop when stateful branching, short-circuiting, mutation, or numerical bounds are clearer there. Composition is useful only when it preserves the needed semantics.

## Comparison

**Since C++20:** defaulted `<=>` compares bases and members lexicographically in declaration order. A defaulted spaceship can supply an implicit defaulted equality operator; a custom spaceship does not automatically supply the desired equality implementation. Declare equality explicitly in that case.

Floating members can produce `partial_ordering`; NaNs are unordered. Do not convert that into an assumed total order for sorting. Define a NaN policy if such values can occur. Defaulted ordering is inappropriate when declaration order differs from domain order.

## Constant evaluation

**Since C++20:** `consteval` requires immediate evaluation; `constinit` requires static initialization without implying constness. Expanded `constexpr` permits useful temporary allocation, but dynamically allocated storage must be released within the applicable constant evaluation. Return a scalar or fixed array rather than assuming an allocated vector can persist as a constexpr object.

A `constexpr` function can execute at runtime. Runtime overflow is still a bug. Constant-expression checks reject prohibited operations in the evaluated constant expression; they do not prove every runtime path safe.

**C++23:** `if consteval` provides an immediate-function context. **C++20 fallback:** use `is_constant_evaluated` for selecting compatible implementations; redesign when calling an immediate function would require the stronger context. Keep the observable mathematical result consistent across paths.

## Formatting and output

**Since C++20:** `format` provides typed formatting. Use fixed format strings when the format is controlled by the program. A runtime format must go through the appropriate API and can fail validation; formatted arguments do not sanitize HTML, SQL, terminal controls, or log record boundaries.

**C++23:** `print` and `println` combine formatting and output. **C++20 fallback:** `format` plus the existing stream; use the project's vetted formatting library when standard `<format>` is unavailable.

**Since C++20:** `osyncstream` can emit a complete record from cooperating writers to the same stream buffer. Writers bypassing that discipline can interleave.

When the target library lacks `osyncstream`, use a shared mutex around each complete record. This **C++20 baseline fallback** requires all writers to share the same mutex. It deliberately holds that mutex across stream I/O; keep this boundary narrow and do not invoke callbacks inside it.

Select output error handling explicitly. Changing formatting libraries can change locale, encoding, formatting errors, and exceptions; it is not purely a spelling change.
