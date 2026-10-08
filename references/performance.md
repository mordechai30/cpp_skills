# Performance decisions

## Layout and cache behavior

**C++20 baseline.** AoS keeps one entity's fields together; SoA keeps one field across entities together.
Prefer SoA for field-selective streaming, SIMD-friendly kernels, and independent field updates. Prefer
AoS for entity-local work that uses most fields. Hybrid layouts can group hot fields and isolate cold
metadata. Neither layout is a universal winner: measure the actual read/write mix and retained working
set, including conversion/update cost and peak memory.

Caches move lines, not abstract objects. Unused fields can consume bandwidth and capacity; scattered
indirections can create dependent cache misses. Sequential prefetching helps a regular stream but
cannot fix random access automatically. SoA can increase the number of concurrent streams and complicate
entity insertion. Measure L1/LLC misses, bandwidth, cycles per entity, and TLB behavior if relevant.

### Field-selective update

**C++20:** form spans from owned field containers. Equal extent is a contract; validation happens before
mutation. Fields represent corresponding entities at identical indices. This example assumes finite
inputs and a domain permitting floating rounding; it does not validate physical units.

```cpp
#include <span>
#include <stdexcept>

/** Updates one field in a structure-of-arrays layout.
 * All fields use identical entity order; views remain within this call. */
inline void integrate_x(std::span<float> positions,
                        std::span<const float> velocities, float seconds) {
    if (positions.size() != velocities.size()) {
        throw std::invalid_argument("field extents differ");
    }
    for (std::size_t i = 0; i < positions.size(); ++i) {
        positions[i] += velocities[i] * seconds;
    }
}
```

Do not expose unrelated mutable field vectors that permit extent drift. Resize into temporary owned
storage and commit all fields together when allocations can fail. Document stable entity IDs separately
from physical index order if compaction/reordering is permitted.

## Allocation and container tradeoffs

**C++20 baseline.** Reserve once from a useful estimate; do not reserve `size()+1` repeatedly. Batch
updates when allocation and synchronization dominate. Per-phase arenas require owner/resource lifetime
ordering. Use an established RAII allocation abstraction or owned containers; do not introduce
manual allocation or raw-pointer ownership for a microbenchmark. Borrowed addresses at an API
boundary do not extend resource lifetime.

Node containers offer different identity/invalidation tradeoffs from flat storage. **C++23:**
`flat_map`/`flat_set` suit measured read-heavy cases. **C++20 fallback:** sorted vector plus `lower_bound`
or an established flat container. **C++26:** `inplace_vector` offers bounded embedded storage;
**C++20 fallback:** a vetted static-vector, or array plus count where always-live elements are acceptable.
`vector::reserve` cannot provide a no-heap guarantee.

Avoid blanket padding. False sharing occurs when independently updated data shares coherency lines;
separate per-worker accumulators and reduce once when that satisfies the contract. A layout change can
increase footprint enough to lose more than contention savings. Check actual target interference sizes.

## Measurement and vectorization

Use an optimized unsanitized build, realistic data sizes/skew, and equivalent-output checks. Separate
setup, cold-start, and steady-state cost; report variance and hardware/compiler/library settings.
Observe benchmark results so the compiler cannot remove the work. Allocation/cache counters explain
hypotheses; wall time still determines whether the deployed workload benefits.

**C++20:** start with contiguous scalar loops and inspect vectorization reports. Prove overlap/aliasing
assumptions, process tails, and specify numerical error. SIMD changes reduction order; FMA and fast-math
can change NaN, infinity, signed-zero, and rounding behavior. **C++26:** `<simd>`/`<linalg>` may help;
**C++20 fallback:** compiler vectorization or a vetted numerical library with explicit buffer/lifetime contracts.

Guideline topics: [Per.6, Per.14, Per.16, Per.18 and Per.19](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines).
