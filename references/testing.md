# Testing C++20 and newer

## Testing and CI decisions

Google Test, Catch2, Google Mock, sanitizers, and CI are external tooling, not C++20 language additions.
Use the existing test framework; choose one when a target is new. Observe domain outcomes and failed
invariants rather than asserting the exact internal call sequence of a refactoring.

Use Google Mock for deliberate protocol boundaries, such as retry/error delivery or submission
rejection. A fake is often clearer for stateful storage. Overconstrained ordered expectations can test
an implementation detail while missing the public failure mode. Validate externally meaningful order
only where it is actually part of the contract.

## High-value cases

- Ownership: failed construction after acquisition, move/assignment invariants, retained views, and
  borrowed callbacks that must not escape. Test a supported rejection path; do not deliberately invoke
  undefined behavior as an ordinary unit-test assertion.
- Buffers: empty/maximal interval, `first > size`, `count > size - first`, integer conversion extremes,
  malformed encoding, and overlapping views where the contract permits or rejects them.
- SoA: mismatched extents, synchronized entity order, zero/short/tail lengths, and numerical tolerance.
- Concepts: accepted domain types and rejected types; negative compile tests must identify the intended
  constraint failure. A compiler command failing because an include is missing is not that evidence.
- Tasks: ready and deferred results, exception delivery, closed submission, cancellation while idle,
  draining/discarding pending work, and owner shutdown before dependencies disappear.

Coordinate concurrency with barriers/latches/events, not sleeps that assume a scheduler. Use an outer
bounded test timeout so a deadlock cannot hang CI indefinitely. Stress repetition can reveal schedules
but is not a proof of progress or complete race freedom.

## Sanitizer and CI configurations

For supported Clang/GCC configurations, compile and link ASan plus UBSan with
`-fsanitize=address,undefined -fno-omit-frame-pointer -g`. Use a separate TSan configuration with
`-fsanitize=thread`; do not combine it with ASan. Include dependency instrumentation where the tool
requires it, and report unsupported platforms/coverage. Use compiler-specific settings for MSVC.
A clean sanitizer run only covers executed instrumented paths.

CI should include a valid C++20 compiler/library pair and each newer deployment pair. Compile examples
and promised fallback branches explicitly; silently excluded preprocessor branches do not count as
validation. Keep target flags and dependencies reproducible. Publish test logs and sanitizer traces
without secret environment values. Use optimized unsanitized performance checks separately.

[GoogleTest primer](https://google.github.io/googletest/primer.html),
[Google Mock cookbook](https://google.github.io/googletest/gmock_cook_book.html),
[Catch2 CMake integration](https://catch2-temp.readthedocs.io/en/latest/cmake-integration.html),
[Clang ASan](https://clang.llvm.org/docs/AddressSanitizer.html),
[Clang UBSan](https://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html), and
[Clang TSan](https://clang.llvm.org/docs/ThreadSanitizer.html) are the authoritative tooling references.
