---
name: cpp-coding-standards
description: Apply relevant C++ Core Guidelines when writing or reviewing C++ code. Check the project standard, naming policy, and error strategy first.
metadata:
  origin: ECC
---

# C++ Coding Standards (C++ Core Guidelines)

Apply the [C++ Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines) where they fit the project. First check the required C++ standard, local style, and exception policy. Use C++20 or newer features only when supported.

## When to Use

- Writing new C++ code (classes, functions, templates)
- Reviewing or refactoring existing C++ code
- Making architectural decisions in C++ projects
- Enforcing consistent style across a C++ codebase
- Choosing between language features (e.g., `enum` vs `enum class`, raw pointer vs smart pointer)

### When NOT to Use

- Non-C++ projects
- Legacy C codebases that cannot adopt modern C++ features
- Embedded/bare-metal contexts where specific guidelines conflict with hardware constraints (adapt selectively)

## Cross-Cutting Principles

These themes recur across the entire guidelines and form the foundation:

1. **RAII everywhere** (P.8, R.1, E.6, CP.20): Bind resource lifetime to object lifetime
2. **Immutability by default** (P.10, Con.1-5, ES.25): Start with `const`/`constexpr`; mutability is the exception
3. **Type safety** (P.4, I.4, ES.46-49, Enum.3): Use the type system to prevent errors at compile time
4. **Express intent** (P.3, F.1, NL.1-2, T.10): Names, types, and concepts should communicate purpose
5. **Minimize complexity** (F.2-3, ES.5, Per.4-5): Simple code is correct code
6. **Value semantics over pointer semantics** (C.10, R.3-5, F.20, CP.31): Prefer returning by value and scoped objects

## Read by task

- Interfaces and functions: [interfaces-and-functions.md](references/interfaces-and-functions.md).
- Classes, ownership, and RAII: [classes-and-resources.md](references/classes-and-resources.md).
- Expressions, constants, and error handling: [expressions-and-errors.md](references/expressions-and-errors.md).
- Concurrency, templates, and concepts: [concurrency-and-templates.md](references/concurrency-and-templates.md).
- Standard library, headers, and naming: [library-and-style.md](references/library-and-style.md).
- Performance and review checklist: [performance-checklist.md](references/performance-checklist.md).

Use the project's chosen C++ standard, style, and exception policy. Treat each rule as applicable only when its assumptions hold.
