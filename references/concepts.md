# Concepts and constraints

## Contents

- [Concepts and requirements](#concepts-and-requirements)
- [Match the expression](#match-the-expression)
- [Constrain algorithms precisely](#constrain-algorithms-precisely)
- [Refinement and subsumption](#refinement-and-subsumption)
- [Packs and version differences](#packs-and-version-differences)
- [Validation](#validation)

## Concepts and requirements

**Since C++20.** A concept names a compile-time predicate. A `requires` clause determines whether a template candidate participates. A `requires` expression tests types and expressions without executing them. Constraint conjunction uses `&&`, not bitwise `&`; disjunction uses `||`.

Choose a standard concept before inventing a container taxonomy. Use `input_range`, `forward_range`, `random_access_range`, or `contiguous_range` according to traversal needs. A range need not have identical iterator and sentinel types. Requiring nested `iterator` names rejects useful ranges unnecessarily.

The four requirement forms serve different purposes:

| C++20 form | Meaning | Common mistake |
|---|---|---|
| `expression;` | The expression is well-formed | It does not assert that the expression is true |
| `typename T::value_type;` | A dependent type exists | It does not guarantee complete storage or constructibility |
| `{ expression } noexcept -> std::same_as<U>;` | Expression validity, optional nonthrowing property, and result constraint | Result checking uses `decltype((expression))`, including references |
| `requires Predicate<T>;` | A nested constraint is satisfied | Omitting `requires` tests only the validity of the boolean expression |

Use `requires(const T& value)` for operations invoked on const objects. The local parameter does not construct a `T`. Do not impose default construction unless the implementation needs it. Require `noexcept` only when callers or recovery paths depend on it.

Dependent substitution failures in the immediate requirement context can make a requirement false. This is not a general error-catching mechanism: invalid nondependent expressions and errors triggered inside instantiated function bodies can still be hard errors. Concepts cannot refer to themselves recursively.

The supplied [concepts tutorial](https://www.geeksforgeeks.org/cpp/constraints-and-concepts-in-cpp-20/) motivates named constraints. The rules above distinguish well-formed expressions from boolean truth and use the four requirement forms defined by [requires-expression wording](https://eel.is/c++draft/expr.prim.req).

## Match the expression

**C++20.** This domain concept supports encoding immutable objects. The function forwards a borrowed output stream, not ownership of the object.

```cpp
#include <concepts>
#include <ostream>
#include <string>

/** Models const encoding with an owned textual result.
 * Use for immutable values that also write to a caller-owned stream. */
template<class T>
concept Encodable = requires(const T& value, std::ostream& out) {
    { value.encode() } -> std::same_as<std::string>;
    { value.write(out) } -> std::same_as<void>;
};

/** Writes one encoded value to the supplied stream.
 * Stream and encoder failures follow the project's exception policy. */
template<Encodable T>
void write_record(const T& value, std::ostream& out) {
    value.write(out);
}
```

`convertible_to<string>` would be a broader requirement; choose it only if conversion is part of the intended contract. Requiring a string says nothing about UTF-8 validity, escaping, or deterministic encoding. Document and test those separately.

**C++20.** The next example intentionally shows all four forms, with a narrow domain contract. Its purpose is a reference to pre-existing cached bytes, not a general “container” concept.

```cpp
#include <concepts>
#include <cstddef>
#include <span>

/** Models an immutable cache with byte-sized values and a borrowed byte view.
 * The owner must keep the returned view valid for the documented duration. */
template<class T>
concept ByteCache = requires(const T& cache) {
    typename T::value_type;
    cache.size();
    { cache.bytes() } noexcept -> std::same_as<std::span<const std::byte>>;
    requires std::same_as<typename T::value_type, std::byte>;
};
```

## Constrain algorithms precisely

**C++20.** Sorting requires more than random access: the iterator must be sortable with the selected comparator and projection. This avoids accepting a const range and failing later inside the algorithm.

```cpp
#include <algorithm>
#include <functional>
#include <iterator>
#include <ranges>
#include <utility>

/** Sorts a mutable random-access range by a projection.
 * Comparator and projection must model the semantic ordering requirements. */
template<std::ranges::random_access_range R,
         class Comp = std::ranges::less, class Proj = std::identity>
    requires std::sortable<std::ranges::iterator_t<R>, Comp, Proj>
void sort_by(R& values, Comp comp = {}, Proj proj = {}) {
    std::ranges::sort(values, std::move(comp), std::move(proj));
}
```

A comparator that changes its answer between calls can compile and still violate strict weak ordering. Floating NaNs need an explicit ordering policy. A concept checks structural requirements; it is not a runtime invariant checker.

For callbacks, constrain the exact invocation and use `std::invoke` if member pointers are intended. Check `F&` if a named callable is invoked as an lvalue; `invocable<F>` may test a different value category. Use `regular_invocable` or `predicate` only when their semantic promises fit the algorithm.

## Refinement and subsumption

**C++20.** Reuse a named concept in stronger concepts. The compiler compares normalized atomic constraints, not arbitrary logical equivalence. Writing equivalent traits twice can create unrelated atoms and ambiguous overloads.

```cpp
#include <concepts>

/** Models types whose const instances expose a size.
 * This structural check makes no complexity or value guarantee. */
template<class T>
concept Sized = requires(const T& value) { value.size(); };

/** Refines Sized with a nonthrowing size query.
 * The shared Sized atom allows the stronger overload to subsume the weaker. */
template<class T>
concept NothrowSized = Sized<T> && requires(const T& value) {
    { value.size() } noexcept;
};

/** Selects the general size-capable path.
 * The integer result makes overload selection observable in tests. */
template<Sized T>
constexpr int size_path(const T&) { return 1; }

/** Selects the nonthrowing size-capable path.
 * Named concept refinement makes this overload more constrained. */
template<NothrowSized T>
constexpr int size_path(const T&) { return 2; }
```

Keep constraint spelling and ordering consistent across redeclarations. Do not rely on a logically equivalent rearrangement being a valid redeclaration. Use named concepts to stabilize the public constraint vocabulary.

## Packs and version differences

**C++20:** fold a named concept over a pack for acceptance, for example `(std::movable<Ts> && ...)`. Do not assume that overloads containing different fold expressions will order by their element predicates.

**C++26:** fold-expanded constraints improve subsumption for certain compatible folds. **C++20 fallback:** factor the weaker pack constraint into a named concept and reuse that exact concept in the stronger overload, or select one overload and dispatch in its body. Test overload selection in the baseline mode rather than assuming C++26 behavior.

**C++23:** explicit object parameters can reduce cv/ref overload duplication. **C++20 fallback:** use ref-qualified member overloads delegating to a common helper. Avoid returning a reference into a temporary merely to reduce duplicate code.

## Validation

Test intended acceptance and rejection with `static_assert`. Include const objects, proxy references, different sentinels, move-only types, and throwing/nonthrowing operations when they are relevant. Negative compile tests should fail because of the intended constraint, not a missing header or syntax error. Verify overload selection, not only concept truth.

Primary source for normalization and ordering: [template constraints](https://eel.is/c++draft/temp.constr). The online draft also includes later-standard rules; retain the C++20-specific behavior stated here.
