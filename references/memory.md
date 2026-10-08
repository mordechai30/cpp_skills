# Memory and RAII

## Ownership and enforcing RAII

**C++20 baseline.** Do not create or own objects/arrays through raw pointers or use manual allocation.
Prefer references for borrowed objects and bounded views for sequences. Borrowed raw pointers remain
permitted where required by an interface; they do not transfer ownership. Preferred representations:

- Independent state: value members and standard containers.
- Exclusive dynamic lifetime: `unique_ptr` created by `make_unique`; transfer the owner explicitly.
- Independent shared lifetime: `shared_ptr`; use `weak_ptr` to observe without extending it.
- Borrowed identity: reference or `optional<reference_wrapper<T>>` when absence is meaningful.
- Borrowed bounded storage: `span`; read-only text: `string_view` with an owner lifetime contract.
- Graph identity: validated indices or generational handles; reject stale generations.

A `shared_ptr` cycle leaks; a weak edge must reflect actual nonownership. A const shared owner does
not make its pointee immutable. `shared_ptr<const T>` prevents mutation through that owner, but other
aliases can still modify T unless the construction/publication protocol excludes them.

Enforce RAII through construction, membership, and nonduplicable ownership. A constructor that throws
runs destructors of fully constructed bases/members, not the incomplete object's destructor. Acquire
resources into members/local owners before the next failure point. Do not put the sole cleanup for a
partially acquired resource in the outer object's destructor.

### Practical transaction owner

**C++20:** this domain-specific rollback object restores a vector's previous size. It is deliberately
noncopyable and nonmovable; it must not outlive its borrowed vector. Rolling back size does not restore
mutated earlier elements, so callers must only append during this transaction.

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

Prefer an established owner to writing a general-purpose scope guard. **C++23:** `scope_exit` supports
local cleanup. **C++20 fallback:** a vetted guard or the domain owner above. A guard design must specify
movement, moved-from state, release, and throwing cleanup policy. `[[nodiscard]]` helps catch ignored
results but does not force callers to keep an owner alive. A temporary lock guard releases at the end
of its full expression; bind it to a named scope owner.

## Buffer boundaries and view escape

**Since C++20:** a span's extent travels with the reference to storage. Validate indices and intervals
before access; `.subspan()` has preconditions, not a portable error result. Check `first <= size` then
`count <= size - first`, which avoids overflow in `first + count`. `vector::at`/`array::at` can express
checked access when throwing fits the contract. **C++26:** `span::at` provides checked access;
**C++20 fallback:** explicitly check then index, returning the project's error or throwing.

A span does not allocate or pin vector storage. Reallocation, destruction, and relevant moves can
invalidate it. `string_view` can include null characters and is not required to terminate. UTF-8 byte
indices are not character or grapheme indices. Preserve an explicit encoding and length contract.

Do not retain a view into an incoming temporary. For deferred work capture/move the owner, then form
views inside the task. For persistent immutable snapshots, shared ownership can be justified; it is
not a remedy for unsynchronized mutations or unbounded retention.

## Cleanup and external resources

Nonthrowing destruction can release a resource but cannot reliably report a failed commit. Provide
checked explicit completion for durability, close, or protocol acknowledgment when failure matters.
Define the postfailure state and whether retry is permitted. Abrupt process termination does not run
normal scope destructors; RAII alone does not prove persisted-data durability.

At a pointer-based C boundary, keep storage in an owner and borrow its address only for the required
call. Validate length, nullability, termination, and retention. Prefer a reference/view-based interface
when one exists. Do not use an owning raw pointer or manual allocation as an interoperability shortcut.
If a dependency returns a manually managed allocation, use an established RAII wrapper or report the
policy conflict; do not invent an implicit exception.

Core Guidelines topics: [R.1, C.20, C.21, C.41, E.6 and E.16](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines).
