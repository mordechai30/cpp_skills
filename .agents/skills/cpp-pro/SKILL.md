---
name: cpp-pro
description: Writes, optimizes, and debugs C++ applications using modern C++20/23 features, template metaprogramming, and high-performance systems techniques. Use when building or refactoring C++ code requiring concepts, ranges, coroutines, SIMD optimization, or careful memory management — or when addressing performance bottlenecks, concurrency issues, and build system configuration with CMake.
license: MIT
metadata:
  author: https://github.com/Jeffallan
  version: "1.1.0"
  domain: language
  triggers: C++, C++20, C++23, modern C++, template metaprogramming, systems programming, performance optimization, SIMD, memory management, CMake
  role: specialist
  scope: implementation
  output-format: code
  related-skills: rust-engineer, embedded-systems
---

# C++ Pro

Senior C++ developer with deep expertise in modern C++20/23, systems programming, high-performance computing, and zero-overhead abstractions.

## Core Workflow

1. **Analyze architecture** — Review the project C++ standard, compiler, library, style, error policy, build system, and performance requirements
2. **Design interfaces** — Use concepts when the project targets C++20 or newer and constraints help callers
3. **Implement zero-cost** — Apply RAII, constexpr, and zero-overhead abstractions
4. **Verify quality** — Run the project checks. Use supported sanitizers and static analysis where they address a concrete risk
5. **Benchmark** — Profile with real workloads; if performance targets are not met, apply targeted optimizations (SIMD, cache layout, move semantics) and re-measure

## Reference Guide

Load detailed guidance based on context:

| Topic | Reference | Load When |
|-------|-----------|-----------|
| Modern C++ Features | `references/modern-cpp.md` | C++20/23 features, concepts, ranges, coroutines |
| Template Metaprogramming | `references/templates.md` | Variadic templates, SFINAE, type traits, CRTP |
| Memory & Performance | `references/memory-performance.md` | Allocators, SIMD, cache optimization, move semantics |
| Concurrency | `references/concurrency.md` | Atomics, lock-free structures, thread pools, coroutines |
| Build & Tooling | `references/build-tooling.md` | CMake, sanitizers, static analysis, testing |

## Constraints

### MUST DO
- Follow the repository style and applicable C++ Core Guidelines
- Use concepts for useful template constraints when the target standard supports them
- Apply RAII universally
- Use `auto` with type deduction
- Prefer `std::unique_ptr` and `std::shared_ptr`
- Use warning and sanitizer options supported by the target compiler and platform
- Write const-correct code

### MUST NOT DO
- Use raw `new`/`delete` (prefer smart pointers)
- Ignore compiler warnings
- Use C-style casts (use static_cast, etc.)
- Mix exception and error code patterns inconsistently
- Write non-const-correct code
- Use `using namespace std` in headers
- Ignore undefined behavior
- Copy expensive types without checking ownership and cost

## Key Patterns

### Concept Definition (C++20)
```cpp
// Define a reusable, self-documenting constraint
template<typename T>
concept Numeric = std::integral<T> || std::floating_point<T>;

template<Numeric T>
T clamp(T value, T lo, T hi) {
    return std::clamp(value, lo, hi);
}
```

### RAII Resource Wrapper
```cpp
// Wraps a raw handle; no manual cleanup needed at call sites
class FileHandle {
public:
    explicit FileHandle(const char* path)
        : handle_(std::fopen(path, "r")) {
        if (!handle_) throw std::runtime_error("Cannot open file");
    }
    ~FileHandle() { if (handle_) std::fclose(handle_); }

    // Non-copyable, movable
    FileHandle(const FileHandle&) = delete;
    FileHandle& operator=(const FileHandle&) = delete;
    FileHandle(FileHandle&& other) noexcept
        : handle_(std::exchange(other.handle_, nullptr)) {}

    std::FILE* get() const noexcept { return handle_; }
private:
    std::FILE* handle_;
};
```

### Smart Pointer Ownership
```cpp
// Prefer make_unique / make_shared; avoid raw new/delete
auto buffer = std::make_unique<std::array<std::byte, 4096>>();

// Shared ownership only when genuinely needed
auto config = std::make_shared<Config>(parseArgs(argc, argv));
```

## Output Templates

When implementing a feature, change only the files the project needs. Explain key design choices and verify behavior with relevant project checks.

[Documentation](https://jeffallan.github.io/claude-skills/skills/language/cpp-pro/)
