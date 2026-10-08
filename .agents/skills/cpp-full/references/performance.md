# Performance engineering

## Contents

- [Measure the contract](#measure-the-contract)
- [Allocation and locality](#allocation-and-locality)
- [Container selection](#container-selection)
- [SIMD and numerical behavior](#simd-and-numerical-behavior)
- [Generic abstraction costs](#generic-abstraction-costs)

## Measure the contract

**C++20 baseline workflow; measurement tools are external.** Tie a layout/allocation hypothesis to working-set size, bandwidth, cache/TLB misses, and contention where relevant. Preserve identity, invalidation, numerical output, and peak-memory requirements when comparing representations.

Separate cold startup from steady state and setup from the operation measured. Keep results observable to avoid dead-code elimination. Repeat measurements and report variation. A constant sample can let a compiler precompute work; a microbenchmark can omit the application's allocation, cache, and contention costs.

Instrumentation/debug builds are not production-speed evidence. Report compiler/library/flags, hardware, input distribution, and variation; counters explain a mechanism but do not replace deployed-workload timing.

## Allocation and locality

**C++20 baseline:** choose the layout by access pattern. Contiguous values reduce pointer chasing when processing whole sequences. Indirection can still be required for identity, polymorphism, stable addresses, or large seldom-used data. Do not erase those requirements for a blanket “vector is faster” rule.

A structure of arrays is useful when kernels read only a few fields across many objects. Keep all field arrays at compatible sizes and verify update order. An array of structures can be better when each operation consumes most fields of one object. Measure bandwidth and working-set behavior.

Use a cheap known count to reserve once. Calling `reserve(size()+1)` repeatedly can force excessive growth. Constructing with a count and constructing with an initializer list are different operations; braces do not universally express “allocate this many.”

**C++20 baseline use of established PMR facilities:** a monotonic resource can reduce repeated allocation for a bounded phase. Containers must be destroyed before their resource and its backing storage. Individual deallocation does not reclaim monotonic storage; release ends the phase and invalidates outstanding allocations. Unsynchronized resources are not made thread-safe by using thread-safe containers around them.

Bounded phase storage must define exhaustion and whether a fallback allocation is permitted. A monotonic resource with an allocating upstream does not prove a no-heap contract. Custom raw-address construction is excluded by this repository's policy; the lifetime/reclamation analysis still applies to compliant allocation abstractions.

For custom allocators, satisfy allocation counts, alignment, equality, propagation, and exception contracts. A single-object pool allocator cannot serve `vector`'s multi-element allocations. Use established resources before implementing a new allocator.

## Container selection

**Since C++23:** `flat_map` / `flat_set` can improve locality in read-heavy lookup. **C++20 fallback:** a sorted vector with `lower_bound`, or an established flat container. A node-based map remains suitable when updates or stable references dominate. Flat-map storage need not be a vector of key/value pairs; use its actual API and invalidation rules.

**Since C++26:** `inplace_vector<T,N>` stores up to N dynamically constructed elements without container heap allocation. **C++20 fallback:** a vetted static-vector; an array plus count only when always-constructed T elements are acceptable. `vector::reserve` still allocates and permits growth; it cannot meet an embedded-storage guarantee.

**C++26:** `hive` targets stable element storage with insertion/erasure. **C++20 fallback:** a suitable stable-node container or an existing pool. Verify traversal order, invalidation, and complexity rather than assuming it is a vector replacement.

Keep capacity and occupancy separate in API contracts. Pool exhaustion must have a defined outcome. Small-buffer optimization in a library type is implementation-specific unless that type's contract guarantees it.

## SIMD and numerical behavior

**C++20 baseline:** start with a scalar contiguous loop that the compiler can vectorize. Check aliasing, alignment, strides, data dependencies, and compiler reports before introducing intrinsics. Do not use architecture-specific instructions without target selection and a scalar fallback.

```cpp
#include <span>
#include <stdexcept>

/** Adds two equal-sized input arrays into a separate output array.
 * Output may exactly alias an input; partially overlapping spans are excluded. */
inline void add_samples(std::span<const float> left,
                        std::span<const float> right,
                        std::span<float> output) {
    if (left.size() != right.size() || left.size() != output.size()) {
        throw std::invalid_argument("sample counts differ");
    }
    for (std::size_t i = 0; i < output.size(); ++i) {
        output[i] = left[i] + right[i];
    }
}
```

The **C++20** loop covers every element, including tail lengths. Partial-overlap exclusion is a caller precondition; size checks do not establish non-overlap. Test empty input, short lengths, exact alias, and large arrays before adding a SIMD path.

**C++26:** `<simd>` and `<linalg>` add standard numerical facilities. **C++20 fallback:** compiler-vectorized loops, a vetted portable SIMD library, or BLAS for suitable matrix work. Verify CPU/runtime dispatch, library linkage, shape/stride contracts, and numerical results.

Vector reductions reorder additions, and fused multiply-add changes rounding. `fast-math` can alter NaN, infinity, signed-zero, and reassociation behavior. Record allowed numerical error before enabling it. A fixed “4–8x SIMD speedup” is not credible without a workload and measurements.

False sharing depends on actual cache topology and placement. Padding every field to 64 bytes can waste memory and still miss the target. Prefer independent per-worker state and a final reduction; measure before changing alignment.

## Generic abstraction costs

**Since C++20:** concepts constrain compilation; they do not themselves insert runtime checks. Ranges can avoid intermediate containers, but repeated traversal and predicates still cost work. A template or coroutine is not automatically a zero-allocation abstraction.

Avoid expression-template systems unless temporary elimination materially matters. Stored references to temporary expression nodes can dangle; a fluent expression such as `b + c + d` needs a correct operand ownership model. Weigh compile time, binary growth, and debugability against runtime benefits.

**C++23:** explicit object parameters reduce some overload duplication. **C++20 fallback:** shared helpers and ref-qualified overloads. This change is mainly about API maintenance, not a guaranteed speedup.

**C++26:** reflection can reduce descriptor boilerplate. **C++20 fallback:** explicit tables or generated code. Measure compile time and binary size, and retain explicit serialization/versioning policy; automatically enumerating members does not establish a stable schema.
