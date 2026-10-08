# Build and dependency decisions

## Target configuration and reproducibility

CMake/vcpkg/Conan versions are independent of ISO C++ language levels. Use CMake targets as the build
contract; IDE projects may consume that contract. Preserve a project's selected manager and lock policy.
Choose either vcpkg or Conan when a target needs a new dependency workflow, not both for the same graph.

**C++20 baseline configuration:** public standard requirements follow public headers. Private warning
flags must not become consumers' usage requirements. This snippet assumes the named source/include
paths exist in the target project; it is not a buildable repository skeleton.

```cmake
cmake_minimum_required(VERSION 3.20)
project(domain LANGUAGES CXX)
add_library(domain src/domain.cpp)
target_compile_features(domain PUBLIC cxx_std_20)
set_target_properties(domain PROPERTIES CXX_EXTENSIONS OFF)
target_include_directories(domain PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>)
if(MSVC)
    target_compile_options(domain PRIVATE /W4)
else()
    target_compile_options(domain PRIVATE -Wall -Wextra -Wpedantic)
endif()
```

`cxx_std_20` is a minimum requirement. For an exact C++20 validation job, configure the target's
`CXX_STANDARD 20`, `CXX_STANDARD_REQUIRED YES`, and extensions policy explicitly; dependency usage
requirements can still demand a higher minimum. Inspect effective commands and reject unexpected
standard promotion in a strict baseline job. Compile/link feature probes under identical flags/library.

## Package graph and ABI

With vcpkg, configure its toolchain before the first `project()` call, pin the registry baseline, and
select a target triplet matching architecture/linkage requirements. With Conan, use profiles for
compiler/standard library/runtime/build-type identity, pin revisions or a reviewed lockfile, and use
CMakeToolchain with CMakeDeps or CMakeConfigDeps as supported by the pinned Conan version.
Keep generated files in build directories.

A dependency advertised as C++20-compatible may still have platform or ABI requirements. Standard
library ABI, CRT linkage, sanitizer instrumentation, architecture, and exception settings must match
across the actual binary boundary. Do not update a pinned graph as an incidental fix for an unrelated
source change. Use target-scoped warnings-as-errors for owned code, not dependencies.

## Modules and hardening

**Since C++20:** modules need integrated scanning, compiler, generator, and packaging support. BMIs are
not portable across arbitrary compilers/configurations. **C++23:** `import std` additionally needs
library-module support. **C++20 fallback:** self-contained headers; keep build-time claims measured.

Vendor hardening can check selected preconditions but is not runtime input validation or universal
bounds checking. **C++26** standardized hardening does not replace recoverable boundary checks;
**C++20 fallback:** supported library hardening plus explicit validation. Verify the installed vendor
mode and compatibility across translation units before changing it.

[CMake target features](https://cmake.org/cmake/help/latest/command/target_compile_features.html),
[vcpkg CMake integration](https://learn.microsoft.com/en-us/vcpkg/users/buildsystems/cmake-integration),
[Conan CMake integration](https://docs.conan.io/2/integrations/cmake.html), and
[libc++ hardening](https://libcxx.llvm.org/Hardening.html) provide current tooling contracts.
