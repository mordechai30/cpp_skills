# Errors and safety

## Contents

- [Typed failure contracts](#typed-failure-contracts)
- [Parsing contracts](#parsing-contracts)
- [Bounds and arithmetic](#bounds-and-arithmetic)
- [Representation and external data](#representation-and-external-data)
- [Exception guarantees](#exception-guarantees)

## Typed failure contracts

**C++20 baseline:** retain the project's failure policy. Return a domain error when callers need to distinguish recovery actions. Optional absence and failure with a cause are different contracts.

**Since C++23:** `expected<T,E>` represents either a result or an error. **C++20 fallback:** the project's established result abstraction; otherwise a variant-backed domain result with explicit branching. Variant is an established baseline utility, not a C++20 introduction. Avoid implementing a complete imitation of `expected` for one function.

`expected` does not force handling or exclude exceptions from construction, allocation, or user code. `nodiscard` remains a diagnostic aid, not control-flow enforcement.

**C++23:** monadic `expected` and `optional` operations can shorten propagation. **C++20 fallback:** explicit branches preserving the same error and value category. Distinguish `and_then` (returns another result) from `transform` (returns a plain transformed value). Check monadic feature support separately.

## Parsing contracts

**C++20 baseline contract for C++23 `expected` adoption:** distinguish empty, malformed, and out-of-range input. Specify sign/whitespace policy and full consumption. For mixed failures such as an oversized numeric prefix followed by trailing bytes, choose and test precedence explicitly; changing the result wrapper must not change classification.

The source parser accepted a minus sign, rejected leading whitespace and a plus sign, required full consumption, and classified a reported range overflow before trailing-byte rejection. Preserve those choices only when they match the domain. A no-allocation promise belongs to the actual parsing implementation, not the result type. Use an implementation permitted by the project's pointer/interface policy.

**C++23:** use `expected<int, ParseError>` and `unexpected` for those domain failures. **C++20 fallback:** a project result type or `variant<int, ParseError>` with a distinct error alternative. Preserve classification and precedence across both branches.

## Bounds and arithmetic

**Since C++20:** `in_range`, `cmp_less`, and the related integer comparison helpers avoid some signed/unsigned conversion errors. Apply them to supported integer types. A `static_cast` alone does not validate representability.

Check multiplication before allocating `count * element_size`; check subtraction bounds before testing `offset + length`. Avoid performing an overflowing expression and checking it afterward. Integer concepts establish types, not permitted values.

**C++26:** `span::at` provides checked access. **C++20 fallback:** explicit range check, then indexing. Use a typed error where exceptions are disabled. Hardening is a separate implementation choice; unchecked access is not validated input.

**C++26:** saturation arithmetic is useful when clamping is the intended numeric model. **C++20 fallback:** a checked routine. For sizes or protocol fields, rejecting overflow is often better than silently saturating.

```cpp
#include <concepts>
#include <limits>

/** Adds unsigned values using a saturation contract.
 * Comparison before addition prevents wraparound in the overflow path. */
template<std::unsigned_integral T>
    requires (!std::same_as<T, bool>)
constexpr T saturating_add(T left, T right) noexcept {
    constexpr T maximum = std::numeric_limits<T>::max();
    return right > maximum - left ? maximum : static_cast<T>(left + right);
}

static_assert(saturating_add<unsigned int>(
    std::numeric_limits<unsigned int>::max(), 1) ==
    std::numeric_limits<unsigned int>::max());
```

The example is **C++20**, the fallback for **C++26** `add_sat`. Do not reuse it for signed arithmetic: signed limits and overflow checks need a different implementation. Division also requires checking the signed minimum divided by minus one, not only zero.

## Representation and external data

**Since C++20:** `bit_cast` copies representation between same-sized trivially copyable types without aliasing through an incompatible pointer. It does not validate arbitrary bit patterns for every destination type, serialize padding portably, or change byte order. Use it only with a justified representation contract.

**Since C++20:** `endian` reports native byte ordering. **C++23:** `byteswap` reverses bytes for supported integral types. **C++20 fallback:** a reviewed unsigned shift-and-mask routine for the specific width; byte-order conversion still needs a native-order branch.

**C++23:** `to_underlying` exposes an enum's underlying value. **C++20 fallback:** `static_cast<underlying_type_t<E>>(value)`. Neither operation validates that an external integer names a permitted enumerator.

At binary boundaries, decode fields explicitly, validate lengths before forming subviews, and separate wire representation from in-memory object layout. Copying bytes into an arbitrary class is not object construction.

## Exception guarantees

**C++20 baseline:** state whether an operation offers no-throw, strong, or basic guarantees. Build replacement state first, then commit with a nonthrowing operation when the strong guarantee is required. Destructors must not report failures by throwing during unwinding.

A `noexcept` function that lets an exception escape terminates; the specifier does not prevent callees from throwing. Use conditional `noexcept` for generic forwarding only when it matches the actual invoked expression.

**C++26 contracts:** programmer preconditions are not recoverable input validation. **C++20 fallback:** explicit error-producing checks for untrusted data and assertions plus documented preconditions for internal misuse. Keep validation active where failure must be handled in release builds.
