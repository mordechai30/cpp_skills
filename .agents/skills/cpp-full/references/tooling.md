# Build and validation

## Contents

- [Target and feature checks](#target-and-feature-checks)
- [Diagnostics and sanitizers](#diagnostics-and-sanitizers)
- [Hardening](#hardening)
- [Modules](#modules)
- [Baseline coverage](#baseline-coverage)

## Target and feature checks

**C++20-and-newer tooling:** target-scoped `cxx_std_20` is a minimum requirement. Public-header requirements propagate to consumers; implementation-only requirements need not. Dependency warnings and ABI settings are not owned-target private policy.

An exact-baseline job must configure the standard explicitly and inspect effective commands; transitive usage requirements can still promote it. Library support requires an actual feature probe.

Use `<version>` macros and a compile/link probe for the precise API. Probe under the same toolchain, library, flags, and language mode as the target. Keep the fallback behind a small compatibility boundary instead of scattering conditional compilation through business logic.

**C++23 adoption:** feature-specific probes plus the C++20 fallback are required before raising a target's requirement. **C++26 adoption:** also check draft wording and implementation restrictions. A successful experimental compiler run is not a supported deployment matrix.

Use the existing dependency manager and lock/version policy. Do not introduce FetchContent, a new package manager, or a floating remote dependency solely to demonstrate C++20.

## Diagnostics and sanitizers

**Tooling for C++20 and later; not ISO-versioned features.** Start with the project's warning set. Enable conversion diagnostics where they address boundary arithmetic, and resolve the underlying issue rather than applying blanket casts. Apply warnings-as-errors to owned code according to the project policy; do not expose arbitrary private warning flags through installed targets.

Instrumentation coverage depends on executed paths and compatible dependencies. MSan needs a suitably instrumented dependency stack; a clean sanitizer run cannot prove unexecuted lifetime paths or progress. Static analysis needs actual compile commands, not a guessed build configuration.

For supported Clang/GCC targets, an address/undefined build commonly uses `-O1 -g -fsanitize=address,undefined -fno-omit-frame-pointer`; supply the sanitizer flags at link time too. Use a separate `-fsanitize=thread` build for concurrency. Probe target support; do not copy these flags to MSVC or unsupported platforms.

Unit tests should verify domain outcomes, failure paths, and lifecycle behavior. Avoid tests of standard containers that never exercise the new code. Use fuzzing for parsers or decoders when malformed inputs are the important risk. Check that a negative compile test fails at the intended constraint.

## Hardening

**Vendor tooling usable with C++20 and later.** Choose the installed library's documented mode. libc++ hardening can add selected precondition checks; libstdc++ and MSVC have different controls and compatibility requirements. Do not assume every iterator or span operation is checked.

A libc++ candidate is `-D_LIBCPP_HARDENING_MODE=_LIBCPP_HARDENING_MODE_FAST` when supported by the installed version. Measure overhead and check mode compatibility across project targets. Do not publish another deployment's overhead as a project guarantee. See [libc++ hardening documentation](https://libcxx.llvm.org/Hardening.html).

`_FORTIFY_SOURCE` depends on libc, optimization, and compiler support. ELF linker options such as RELRO are not portable macOS or Windows settings. Stack protection and automatic initialization flags are compiler/platform options; initialization instrumentation does not repair a broken source-level invariant.

Verify the effective compile/link commands and resulting binary where applicable. A macro printed by the preprocessor proves definition, not that a particular call has been fortified. Use platform-specific binary inspection; do not infer that a absent canary in a function with no protected frame means the entire build option failed.

**C++26:** standardized library hardening is not automatic recoverable validation. **C++20 fallback:** supported vendor hardening plus explicit checks at untrusted boundaries. Keep checks that must report ordinary errors independent of assertion/contract enforcement settings.

## Modules

**Since C++20:** language modules can separate exported interfaces. Check compiler, build-system dependency scanning, generator, and library integration together. Module interface and implementation units are separate files with separate roles; putting incompatible module declarations in one translation unit is not a practical example.

Do not promise faster builds without measuring clean and incremental builds. BMI artifacts are toolchain/configuration-dependent; do not distribute them as universal binaries. Keep macros and configuration out of accidental import-order contracts.

**C++23:** `import std` uses standard library modules. **C++20 fallback:** self-contained standard headers; language module support alone does not imply `std` is supplied. Continue using headers when the project's deployment tools do not support module dependencies reliably.

## Baseline coverage

Maintain at least a C++20 compatibility build when a fallback is promised. Add C++23/C++26 jobs only for the features actually deployed, and record unsupported facilities separately. CI combinations must be valid compiler/OS/library combinations, not a blind Cartesian matrix.

Primary tooling references: [CMake target features](https://cmake.org/cmake/help/latest/command/target_compile_features.html) and [Clang TSan](https://clang.llvm.org/docs/ThreadSanitizer.html).
