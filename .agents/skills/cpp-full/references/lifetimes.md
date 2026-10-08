# Lifetimes and interfaces

## Contents

- [Ownership and borrowing](#ownership-and-borrowing)
- [Resource boundaries](#resource-boundaries)
- [Move invariants](#move-invariants)
- [Callback ownership](#callback-ownership)
- [Layout and API stability](#layout-and-api-stability)

## Ownership and borrowing

**C++20 baseline policy:** do not create or own objects/arrays through raw pointers; do not use manual
allocation. Prefer `const T&` for read-only objects, `T&` for mutable objects, optional reference wrappers
for optional identity, and spans/string views for sequences/text. Use borrowed raw pointers only where
an interface requires them. Define nullability and owner lifetime; do not delete borrowed objects.
References do not prevent dangling or concurrent mutation; retained work must keep an appropriate owner.

**C++20 baseline:** a smart-owner parameter changes the lifetime/transfer contract; it is not interchangeable with borrowing the object. For retained views, identify the owner and invalidation events, not only the parameter type.

**Since C++20:** `span<T>` carries an extent but neither pins storage nor checks ordinary indexing portably. `span<const T>` prevents mutation through the view; a const `span<T>` still permits element mutation.

Return a view only when its owner and invalidation contract are clear. Vector growth, string modification, owner moves, and owner destruction can invalidate stored views. For deferred work, capture the owner or a stable shared snapshot rather than an untracked span.

**C++20 practical example:** return an owned selection derived from a borrowed input. The view remains within the function.

```cpp
#include <algorithm>
#include <span>
#include <stdexcept>
#include <vector>

/** Copies a validated interval from borrowed contiguous input.
 * Subtraction after the first check prevents overflow in first + count. */
[[nodiscard]] inline std::vector<int> copy_interval(
    std::span<const int> input, std::size_t first, std::size_t count) {
    if (first > input.size() || count > input.size() - first) {
        throw std::out_of_range("interval outside input");
    }
    const auto part = input.subspan(first, count);
    return {part.begin(), part.end()};
}
```

A string view need not be null-terminated. Retention or a terminated-text boundary requires a separate lifetime/representation contract, subject to project interface restrictions.

## Resource boundaries

Follow [R.1](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#rr-raii): enforce acquisition/release through an owner. Use owning members for partial-construction cleanup, deleted copying for exclusive resources, and checked completion for fallible commit. Do not manually mirror cleanup across branches.

**C++20:** this append transaction is a domain rollback owner. Its borrowed vector must outlive it; only appends are permitted before commit.

```cpp
#include <cstddef>
#include <vector>

/** Rolls back appended values unless committed.
 * The borrowed vector must outlive this nontransferable transaction. */
class AppendTransaction {
public:
    explicit AppendTransaction(std::vector<int>& values) noexcept
        : values_{values}, initial_size_{values.size()} {}
    AppendTransaction(const AppendTransaction&) = delete;
    AppendTransaction& operator=(const AppendTransaction&) = delete;
    AppendTransaction(AppendTransaction&&) = delete;
    AppendTransaction& operator=(AppendTransaction&&) = delete;
    ~AppendTransaction() noexcept {
        if (!committed_) {
            while (values_.size() > initial_size_) { values_.pop_back(); }
        }
    }
    void commit() noexcept { committed_ = true; }
private:
    std::vector<int>& values_; ///< Borrowed append-only transaction target.
    std::size_t initial_size_; ///< Size to restore when unwinding.
    bool committed_{}; ///< Suppresses rollback after successful completion.
};
```

**C++20 baseline:** incomplete-object destruction does not run the outer destructor; already-constructed members must own partial acquisition. Cleanup cannot report failed durability/commit by throwing during unwinding: use a checked completion operation and define the postfailure state. At C boundaries, keep storage owned and limit borrowed pointer extraction to the required call; check retention, size, and termination requirements. Manual allocation remains prohibited.

**C++23:** `scope_exit` is suitable for local rollback. **C++20 fallback:** use an existing scope guard or a narrow resource/transaction owner. A custom guard must define movement, release, and nonthrowing cleanup; a destructor-only sketch is not a complete general utility.

Introducing a typed result does not change allocation failure behavior automatically; preserve the exception-disabled factory contract where applicable.

## Move invariants

**C++20 baseline:** defaulted moves can violate coupled metadata invariants: moving a unique owner while copying its size can leave a nonzero length paired with an empty owner.

A moved-from object must satisfy its permitted-use contract, not necessarily its original value invariant. Allocator-sensitive movement can throw; conditional `noexcept` must follow the actual members/allocator behavior.

Return a named local without `std::move(local)` so NRVO remains possible. NRVO is permitted, not guaranteed. Guaranteed prvalue copy elision is an established baseline facility, not a new C++20 feature. Moving a const object commonly copies because the usual move constructor accepts a nonconst rvalue.

Do not make all members `const` merely to advertise immutability: that can delete assignment or turn moves into copies. Restrict mutation through the public interface while retaining a usable value type.

## Callback ownership

**C++20:** prefer constrained callable parameters for immediate invocation when no fixed type-erased ABI is needed.

```cpp
#include <concepts>
#include <functional>
#include <span>

/** Visits borrowed samples synchronously without storing the callable.
 * F is invoked as an lvalue; references need only outlive this call. */
template<class F>
    requires std::invocable<F&, int>
void visit_samples(std::span<const int> samples, F&& visit) {
    for (const int value : samples) { std::invoke(visit, value); }
}
```

**C++23:** `move_only_function` can own a callable with move-only captures. **C++20 fallback:** an existing owning wrapper; for one-shot tasks delivering futures, `packaged_task` can fit. Converting everything to `shared_ptr` solely to fit a copyable wrapper changes ownership and adds costs.

**C++26:** `function_ref` provides borrowed type erasure. **C++20 fallback:** the constrained synchronous API above, or an established borrowed wrapper when a fixed interface is required. Never store a borrowed wrapper beyond its callable's lifetime.

An asynchronous lambda capturing `this` does not keep the object alive. A shared pointer keeps the pointee alive but does not synchronize its mutable state. Avoid reference-returning accessors on temporary owners; reject rvalue access or return a value.

## Layout and API stability

**C++20:** designated initializers make aggregate setup readable. Designators must follow declaration order and cannot be mixed with positional entries. Aggregates are suitable for independent configuration fields, not for invariants requiring constructor validation.

**C++23:** explicit object parameters and `forward_like` can centralize cv/ref behavior. **C++20 fallback:** ref-qualified overloads; use a shared helper where it removes substantive duplication. Review returned reference lifetimes separately.

**C++26:** `indirect` and `polymorphic` support dynamically allocated value semantics. **C++20 fallback:** ordinary value members or a unique owner with explicit deep-copy/clone operations. Deep copy and alias-preserving shared ownership are different contracts.
