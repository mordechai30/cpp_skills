---
name: cpp
description: Develop C++ in the project's selected standard, with guidance for C++20 and newer, ownership, and performance.
---

# C++ Development Guidelines

Check the project standard, compiler, standard library, and style policy before choosing a feature or convention. Use C++20 and newer features when the project supports them.

## Code Style and Structure

- Use the project standard. Prefer C++20 and newer idioms when the project supports them
- Use meaningful variable and function names
- Follow the Single Responsibility Principle
- Prefer composition over inheritance
- Keep functions small and focused

## Naming Conventions

- Follow the repository's existing naming policy. If it is absent, choose one consistent convention for types, functions, variables, constants, and members.
- Do not mix Google-specific naming rules with other project styles.

## Memory Management

### Smart Pointers
- **Do not use raw pointers for object/array creation.** 
- Do not use manual `new`/`delete`, `malloc`/`free`, C arrays, or C-string interfaces.
- Use `std::unique_ptr` for exclusive ownership
- Use `std::make_unique` over direct use of `new` for `std::unique_ptr` whenever possible
- Use `std::shared_ptr` only when shared ownership is required
- Use `std::make_shared` over direct use of `new` for `std::shared_ptr` whenever possible
- Use `std::weak_ptr` to break circular references

### RAII (Resource Acquisition Is Initialization)
- Use RAII for all resource management
- Wrap resources in classes with proper destructors
- Ensure exception safety through RAII
- Use scope guards for cleanup operations

### Best Practices
- Prefer stack allocation over heap allocation
- Use containers instead of raw arrays

## Modern C++ Features

- Use structured bindings for tuple unpacking
- Use `std::optional` for values that may not exist
- Use `std::variant` for type-safe unions
- Use `if constexpr` for compile-time conditionals
- Use `std::string_view` for non-owning string references
- Use `std::span` for non-owning array/vector views

- Use concepts for template constraints
- Use ranges for cleaner algorithms
- Use coroutines for asynchronous operations
- Use modules for faster compilation (when supported)

## Error Handling

- Follow the project error-handling policy. Use exceptions only when enabled and suitable; use typed results where the chosen standard and API call for them.
- Define domain errors when callers need to distinguish failures
- Use `noexcept` for functions that don't throw
- Catch exceptions by const reference
- Provide strong exception guarantees where possible

## Performance

- Use `const` and `constexpr` liberally
- Move when ownership transfers and the source remains in a valid state; do not move from objects that are still needed
- Use perfect forwarding with `std::forward`
- Avoid unnecessary copies
- Profile before optimizing
- Use `inline` for linkage or definitions in headers; measure before changing code for speed

## Security

### Buffer Safety
- Use `std::array` or `std::vector` instead of C-style arrays
- Use `std::vector`; use `.at()` or verified library hardening when bounds checks are required
- Prefer `std::string` over C-style string
- Use `std::string_view` for function parameters that accept read-only string data such as a C-style string, or `std::string`
- Use `std::span` for function parameters that accept read-only array or vector or C-style arrays.
- Store string(as member) as `std::string` instead of C-style string

### Type Safety
- Avoid C-style casts; use `static_cast`, `dynamic_cast`, etc.
- Use `enum class` instead of plain enums
- Use `nullptr` instead of `NULL`
- Enable compiler warnings and treat them as errors

## Concurrency

- Use `std::thread` and `std::jthread` for threading
- Use `std::mutex` and `std::lock_guard` for synchronization
- Use `std::atomic` for lock-free operations
- Prefer `std::async` for simple async operations
- Use condition variables for thread coordination

## Testing

- Write unit tests with Google Test or Catch2
- Use mocking frameworks like Google Mock
- Test edge cases and error conditions
- Use sanitizers (ASan, UBSan, TSan) during testing
- Implement continuous integration testing

## Documentation

- Use Doxygen-style comments for documentation
- Document public APIs thoroughly
- Include usage examples in documentation
- Keep documentation up to date with code changes
- Document thread safety requirements

## Build System

- Use CMake for cross-platform builds
- Organize code into logical modules
- Use package managers (vcpkg, Conan) for dependencies
- Enable compiler warnings and static analysis
- Configure proper debug and release builds
