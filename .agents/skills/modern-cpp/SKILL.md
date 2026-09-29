---
name: modern-cpp
description: Apply modern C++ idioms when writing or updating C++20 and newer code. Check the project standard, compiler, standard library, and error policy before choosing a feature.
---

# Modern C++

Guide for C++20 and newer code. Follow the project standard, compiler support, style, and error policy. Use newer features where available and useful.

## When to Use This Skill

- Writing new C++ functions, classes, or libraries
- Modernizing existing C++ code (pre-C++20 patterns)
- Choosing between legacy and modern approaches
- Working on security-critical or safety-sensitive C++
- Reviewing C++ code for modern idiom adoption

## When NOT to Use This Skill

- **User explicitly requires older standard**: Respect constraints (embedded, legacy ABI)
- **Pure C code**: This skill is C++-specific
- **Build system questions**: CMake, Meson, Bazel configuration is out of scope
- **Non-C++ projects**: Mixed codebases where C++ isn't primary

## Anti-Patterns to Avoid

| Avoid | Use Instead | Why |
|-------|-------------|-----|
| `new`/`delete` | `std::make_unique`, `std::make_shared` | Eliminates leaks, double-free |
| Raw owning pointers | `std::unique_ptr`, `std::shared_ptr` | RAII ownership semantics |
| C arrays (`int arr[N]`) | `std::array<int, N>` | Bounds-aware, value semantics |
| Pointer + length params | `std::span<T>` | Non-owning view with size; ordinary indexing is not automatically checked |
| `printf` / `sprintf` | `std::format`, `std::print` | Type-safe, no buffer overflow |
| C-style casts `(int)x` | `static_cast<int>(x)` | Explicit intent, auditable |
| `#define` constants | `constexpr` variables | Scoped, typed, debuggable |
| SFINAE / `enable_if` | Concepts + `requires` | Readable constraints and errors |
| Error codes + out params | `std::expected<T, E>` | Composable, type-safe errors |
| `union` | `std::variant` | Type-safe, no silent UB |
| Raw `mutex.lock()/unlock()` | `std::scoped_lock` | Exception-safe, no deadlocks |
| `std::thread` | `std::jthread` | Auto-join, stop token support |
| Manual CRTP | Deducing `this` (C++23) | Simpler, no template boilerplate |

See [anti-patterns.md](./references/anti-patterns.md) for the full table (30+ patterns).

## Decision Tree

```
What are you doing?
|
+-- Writing new C++ code?
|   +-- Use features supported by this project
|   +-- Keep the project's chosen standard unless an upgrade is requested
|   +-- See Feature Tiers below
|
+-- Modernizing existing code?
|   +-- Start with Tier 1 (C++20/23) replacements
|   +-- Prioritize by security impact (memory > types > style)
|   +-- See anti-patterns.md for the migration table
|
+-- Security-critical code?
|   +-- Enable compiler hardening flags (see below)
|   +-- Enable hardened libc++ mode
|   +-- Run sanitizers in CI
|   +-- See safe-idioms.md and compiler-hardening.md
|
+-- Using C++26 features?
    +-- Check implementation status before considering reflection
    +-- Contracts: cautiously, for new API boundaries
    +-- std::execution: wait for ecosystem maturity
    +-- See cpp26-features.md
```

## Feature Tiers

Features are ranked by **practical usability today**, not by standard version.

### Tier 1: Use when the target toolchain supports them (C++20/23)

| Feature | Replaces | Standard |
|---------|----------|----------|
| Concepts + `requires` | SFINAE, `enable_if` | C++20 |
| Ranges + views | Raw iterator loops | C++20 |
| `std::span<T>` | Pointer + length | C++20 |
| `std::format` | `sprintf`, iostream chains | C++20 |
| Three-way comparison `<=>` | Manual comparison operators | C++20 |
| `std::jthread` | `std::thread` + manual join | C++20 |
| Designated initializers | Positional struct init | C++20 |
| `std::expected<T,E>` | Typed results for expected failures | C++23 |
| `std::print` / `std::println` | `printf`, `std::cout <<` | C++23 |
| Deducing `this` | CRTP, const/non-const duplication | C++23 |
| `std::flat_map` | `std::map` for read-heavy use | C++23 |
| Monadic `std::optional` | Nested if-checks on optionals | C++23 |

See [cpp20-features.md](./references/cpp20-features.md) and [cpp23-features.md](./references/cpp23-features.md).

### Tier 2: Consider after a platform and build check

These options depend on the compiler, standard library, operating system, and build mode:

- **Compiler hardening flags** — `-D_FORTIFY_SOURCE=3`, `-fstack-protector-strong`, `-ftrivial-auto-var-init=zero`
- **Hardened libc++** — `-D_LIBCPP_HARDENING_MODE=_LIBCPP_HARDENING_MODE_FAST` for ~0.3% overhead bounds-checking
- **Sanitizers in CI** — ASan + UBSan as minimum; TSan for concurrent code
- **Warning flags** — `-Wall -Wextra -Wpedantic -Werror`

See [compiler-hardening.md](./references/compiler-hardening.md).

### Tier 3: Plan For (C++26, worth restructuring around)

**Reflection** may reduce some boilerplate when the target compiler and library support it. Possible uses include:
- Serialization boilerplate (one generic function replaces per-struct `to_json`)
- Code generators (protobuf codegen, Qt MOC)
- Macro-based registration and enum-to-string hacks

Do not restructure existing code around reflection without an explicit compatibility plan.

### Tier 4: Watch (C++26, needs maturation)

- **Contracts** (`pre`/`post`/`contract_assert`) — Better than `assert()`, but no virtual function support and limited compiler support. Adopt cautiously for new API boundaries.
- **std::execution** (senders/receivers) — Powerful async framework, but steep learning curve, no scheduler ships with it, and poor documentation. Wait for ecosystem maturity.

See [cpp26-features.md](./references/cpp26-features.md).

## Compiler Hardening Quick Reference

### Candidate flags (verify on the target platform)

```
-Wall -Wextra -Wpedantic -Werror
-D_FORTIFY_SOURCE=3
-fstack-protector-strong
-fstack-clash-protection
-ftrivial-auto-var-init=zero
-fPIE -pie
-Wl,-z,relro,-z,now
```

### Clang-Specific

```
-Wunsafe-buffer-usage
```

### Hardened libc++ (Clang/libc++ only)

```
-D_LIBCPP_HARDENING_MODE=_LIBCPP_HARDENING_MODE_FAST
```

Measure hardening overhead and verify the selected mode on the project toolchain.

See [compiler-hardening.md](./references/compiler-hardening.md) for the full guide.

## Rationalizations to Reject

| Rationalization | Why It's Wrong |
|----------------|----------------|
| "It compiles without warnings" | Warnings depend on which flags you enable. Add `-Wall -Wextra -Wpedantic`. |
| "ASan is too slow for production" | Use GWP-ASan for sampling-based production detection (~0% overhead). |
| "We only use safe containers" | Iterator invalidation and unchecked `optional` access are still exploitable. |
| "Smart pointers are slower" | `std::unique_ptr` has zero overhead vs raw pointers. Measure before claiming. |
| "Our code doesn't have memory bugs" | Google found 1000+ bugs when enabling hardened libc++. So did everyone else. |
| "C++26 features aren't available yet" | Use C++20/23 features that the project supports. Select hardening flags by platform and compiler. |
| "Modern C++ is harder to read" | `std::expected` is more readable than checking error codes across 5 out-params. |

## Best Practices Checklist

- [ ] Use smart pointers for ownership, raw pointers only for non-owning observation
- [ ] Prefer `std::span` over pointer + length for function parameters
- [ ] Consider `std::expected` for typed, expected failures when C++23 is available and the project uses this error policy
- [ ] Constrain templates with concepts, not SFINAE
- [ ] Select supported hardening flags for each target platform and build mode
- [ ] Run ASan + UBSan in CI; add TSan for concurrent code
- [ ] Use `constexpr` / `consteval` where possible (UB-free by design)
- [ ] Mark functions `[[nodiscard]]` when ignoring the return value is likely a bug
- [ ] Prefer value semantics; use `std::variant` over `union`, `enum class` over `enum`
- [ ] Initialize all variables at declaration

## Read Next

- [anti-patterns.md](./references/anti-patterns.md) — Full legacy-to-modern migration table (30+ patterns)
- [cpp20-features.md](./references/cpp20-features.md) — Concepts, ranges, span, format, coroutines
- [cpp23-features.md](./references/cpp23-features.md) — expected, print, deducing this, flat_map
- [cpp26-features.md](./references/cpp26-features.md) — Reflection, contracts, memory safety improvements
- [compiler-hardening.md](./references/compiler-hardening.md) — Flags, sanitizers, hardened libc++
- [safe-idioms.md](./references/safe-idioms.md) — Security patterns by vulnerability class
