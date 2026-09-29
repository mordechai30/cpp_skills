# Review Checklist

## Best Practices

1. **Prefer auto for complex types**: Use `auto` to avoid verbose type
   declarations, especially with iterators and lambdas
2. **Use range-based for when possible**: Clearer intent and less error-prone
   than traditional loops
3. **Prefer lambdas for simple operations**: Especially in algorithm calls like
   `std::transform` and `std::for_each`
4. **Use smart pointers for ownership**: Prefer `unique_ptr` and `shared_ptr`
   over raw pointers for owned resources
5. **Embrace move semantics**: Implement move constructors/assignment for
   resource-owning types
6. **Use structured bindings**: Makes code more readable when working with
   pairs, tuples, and maps
7. **Prefer std::optional over special values**: Use `std::optional` instead of
   sentinel values like -1 or nullptr
8. **Use concepts to constrain templates (C++20)**: Makes template errors
   clearer and documents requirements
9. **Leverage ranges and views (C++20)**: Can improve composition; measure performance for the workload
10. **Use constexpr for compile-time computation**: Move computation to
    compile-time when possible for better performance

## Common Pitfalls

1. **Dangling references with auto**: `auto&` can create dangling references
   to temporaries
2. **Capturing by reference in lambdas**: Reference captures can outlive the
   captured variables
3. **Moving from const objects**: `std::move` on const objects doesn't
   actually move
4. **Default capturing [=] or [&]**: Can accidentally capture more than
   intended
5. **Forgetting to mark move operations noexcept**: Prevents some optimizations
   in containers
6. **Using std::move after using an object**: Moved-from objects are in valid
   but unspecified state
7. **Range-based for with temporaries**: Container temporaries destroyed before
   loop body
8. **Optional value access without checking**: Using `*` or `value()` without
   verifying `has_value()`
9. **Variant access without checking**: `std::get` throws if wrong type; use
   `get_if` or visitor
10. **Overusing auto**: Can hide important type information; use judiciously

## When to Use

Use this skill when:

- Working with modern C++ codebases (C++11 and later)
- Refactoring legacy code to use modern features
- Implementing resource management with RAII
- Writing generic code with templates and lambdas
- Optimizing performance with move semantics
- Implementing type-safe optional or variant types
- Working with ranges and functional-style algorithms
- Creating compile-time computed values
- Applying concepts to constrain templates
- Learning or teaching modern C++ practices

## Resources

- [C++ Reference - Lambda](https://en.cppreference.com/w/cpp/language/lambda)
- [C++ Reference - Auto](https://en.cppreference.com/w/cpp/language/auto)
- [C++ Reference - Range-based for](https://en.cppreference.com/w/cpp/language/range-for)
- [C++ Reference - Smart Pointers](https://en.cppreference.com/w/cpp/memory)
- [C++ Reference - Move Semantics](https://en.cppreference.com/w/cpp/language/move_constructor)
- [C++ Reference - Structured Bindings](https://en.cppreference.com/w/cpp/language/structured_binding)
- [C++ Reference - Optional](https://en.cppreference.com/w/cpp/utility/optional)
- [C++ Reference - Variant](https://en.cppreference.com/w/cpp/utility/variant)
- [C++ Reference - Concepts](https://en.cppreference.com/w/cpp/language/constraints)
- [C++ Reference - Ranges](https://en.cppreference.com/w/cpp/ranges)
- [C++ Reference - Coroutines](https://en.cppreference.com/w/cpp/language/coroutines)
- [C++ Reference - Constexpr](https://en.cppreference.com/w/cpp/language/constexpr)
