---
name: cpp-full
description: >-
  Guides C++20-and-newer implementation and review with explicit ownership, borrowing, async-task,
  range, constraint, and performance choices, plus C++23/C++26 fallbacks.
---

# C++ Full

Use the [C++ Core Guidelines](../../../core-guidelines/CppCoreGuidelines.md#rr-raii) as the design  
authority.  
Follow the choices below. Read references for examples and subtle contracts.

## Standard and compatibility

- Use C++20 or newer. Treat unmarked rules as C++20 baseline guidance, including established facilities.
- Label C++23/C++26 features and provide a C++20 fallback.
- Check compiler and standard-library support for the actual API; do not rely on the language flag alone.
- Preserve exception policy, ABI, ownership, and invalidation when replacing an implementation.
- Read [feature choices](references/versions.md#selection-and-support) for support checks and fallbacks.
- Read [C++26 adoption checks](references/versions.md#c26-adoption-checks) before using C++26 facilities.

## Pointers, arrays, memory management/safety

- **Do not create or own objects/arrays through raw pointers.**
- **Do not use manual `new`/`delete` or `malloc`/`free`.** Use values, containers, or smart owners.
- Use `std::array` for fixed-size storage and `std::vector` for dynamic-size storage. Do not use C arrays.
- Use `std::string` for owned text. Do not store text in C-style character arrays or owning `char*`.
- Prefer local values and value members over separate heap allocations. Check stack limits for large buffers.
- Use `std::unique_ptr` for exclusive dynamic ownership; create it with `std::make_unique`.
- Use `std::shared_ptr` only when independent users must extend one lifetime; create it with  
  `std::make_shared`.
- Use `std::weak_ptr` to observe shared ownership without extending lifetime or creating an ownership cycle.
- Pass small, cheap inputs by value. Prefer `const T&` for larger read-only objects and `T&` for mutation.
- Prefer `std::optional<std::reference_wrapper<T>>` for optional borrowed objects.
- Use `std::span<const T>` for read-only contiguous parameters and `std::span<T>` for mutable ones.
- Use `std::string_view` for borrowed read-only text parameters.
- Pass smart pointers only to transfer ownership or participate in lifetime management. Otherwise borrow the  
  object.
- Use borrowed raw pointers only where an interface requires them. Specify nullability and owner lifetime;  
  never delete them.
- At C/API boundaries, keep buffers owned by containers and limit pointer extraction to the call. Check size  
  and termination requirements.
- Do not return or retain references, pointers, or views to destroyed or invalidated storage.
- Read [ownership and lifetime details](references/lifetimes.md#ownership-and-borrowing).

## Language methods to enforce RAII

- Manage memory, files, locks, and other paired resources through RAII owners.
- Acquire resources in constructors or checked factories. Release them in nonthrowing destructors.
- Store acquired resources in owning members so failed construction cleans up completed members.
- Prefer rule-of-zero classes. Delete copying for exclusive resources; define moves only when their  
  invariants require it.
- Keep resource state private. Do not require callers to remember cleanup on each return or exception path.
- Bind lock and rollback guards to named local variables; an unnamed temporary releases too early.
- Report fallible commit/close through a checked operation. Do not report failure by throwing from a  
  destructor.
- Mark important acquisition/error results `[[nodiscard]]`; enable diagnostics. This does not enforce owner  
  lifetime.
- Use C++23 `std::scope_exit` for cleanup; C++20 fallback: a vetted guard or transaction owner.
- Read [construction, rollback, and move details](references/lifetimes.md#resource-boundaries).

## Concurrency: async tasks first

- **Use async tasks and result APIs instead of managing threads for each operation.**
- Use `std::async(std::launch::async, ...)` for a small, bounded set of independent calls.
- Use the project's bounded executor/async library for repeated tasks, composition, or asynchronous I/O.
- Do not build a thread pool as routine application plumbing or launch unbounded per-element tasks.
- Minimize shared writable data. Copy small inputs, move owned inputs, or share genuinely immutable  
  snapshots.
- Return task results and combine them after completion. Avoid tasks mutating shared output.
- Keep futures. Use `wait_for` to inspect readiness and `get()` to retrieve values or exceptions.
- Do not treat `valid()` as readiness, discard potentially blocking async futures, or busy-poll for  
  completion.
- Give retained tasks an owner and a shutdown policy. Do not capture borrowed input that can expire before  
  completion.
- Use cooperative cancellation, such as C++20 `std::stop_token`, when supported by the task API. Timeout is  
  not cancellation.
- Use mutexes, condition variables, and atomics only when shared state is unavoidable. Prefer library  
  abstractions.
- Do not use `volatile` for synchronization or implement lock-free structures without a measured need and  
  reclamation plan.
- Use C++26 senders/receivers with supported schedulers; C++20 fallback: an established async task API.
- Read [task details and examples](references/concurrency.md#task-first-concurrency).

## Performance

- Use SoA (struct of arrays) for hot loops reading a few fields across many entities.
- Use AoS (array of structs) when each operation uses most fields of one entity.
- Keep SoA field sizes and entity order consistent. Preserve identity when compacting or reordering storage.
- Prefer contiguous hot data. Batch work, reserve from useful estimates, and avoid repeated allocation in  
  hot paths.
- Use task-local accumulation followed by reduction instead of shared counters or shared output.
- Measure optimized builds with realistic inputs and equivalent results. Do not infer speed from a layout's  
  name.
- Check cache misses, working-set size, bandwidth, allocations, and contention to explain bottlenecks.
- Add padding only for measured false sharing. Add SIMD only after checking tails, overlap, and numerical  
  tolerance.
- Read [layout, allocation, and measurement details](references/performance.md#measure-the-contract).

## Ranges, views, and function parameters

- Use C++20 `std::span` for contiguous borrowing and `std::string_view` for immediate text reads.
- Use constrained ranges for noncontiguous input. Pass owned values to code that retains data or runs  
  asynchronously.
- Keep an explicit owner/invalidation contract for stored or returned views. References and views do not  
  extend lifetime.
- Check external indices and sizes before indexing or forming subviews. `span::operator[]` is not a portable  
  bounds check.
- Do not assume `string_view` is null-terminated. Use owned terminated text when an API requires it.
- Use views to avoid intermediate containers; materialize when ownership or repeated evaluation requires it.
- Account for distinct iterator/sentinel types and cached view state. Do not change predicates' underlying  
  data blindly.
- Use C++23 `std::ranges::to` for materialization; C++20 fallback: reserve when size is cheap, then append.
- Read [range and materialization details](references/algorithms.md#ranges-and-materialization).

## Type safety

- Use domain types for quantities/identities that must not mix. Use `enum class` for distinct choices.
- Initialize values before use. Check numeric bounds before arithmetic and narrowing conversions.
- Do not use casts to hide overflow, truncation, or an incompatible type contract.
- Use C++23 `std::expected` for recoverable domain errors; C++20 fallback: a project result or distinct  
  `std::variant` alternatives.
- Preserve the project's exception policy. Do not replace a failure with a plausible placeholder value.
- Read [typed errors and checked operations](references/errors.md#typed-failure-contracts).

## Templates, concepts, and constraints

- Use C++20 constrained `auto` for simple generic parameters and named concepts for reusable requirements.
- Use `requires` to control overload participation. Match requirements to the actual const/reference  
  operations.
- Reuse named concepts when refining constraints so overload subsumption works as intended.
- Specify semantic requirements separately: concepts cannot prove ordering laws, owner lifetime, or thread  
  safety.
- Read [constraint design and examples](references/concepts.md#concepts-and-requirements).

## Style, build, tests, and documentation

- Follow the target's naming and formatting. Name purpose, units, and async/blocking behavior explicitly.
- Use target-based CMake and the project's pinned vcpkg/Conan workflow. Do not rely on IDE-only settings.
- Use Google Test or Catch2; use Google Mock for protocol boundaries. Test edge cases and failure paths.
- Run ASan/UBSan and a separate TSan job where applicable. Test promised C++20 fallbacks in CI.
- Document ownership, invalidation, errors, cancellation, and numerical assumptions; use Doxygen for public  
  contracts.
- Read [toolchain and validation details](references/tooling.md#target-and-feature-checks).

## Skill maintenance

- Read [source decisions](references/source-audit.md#coverage-and-selection) when updating guidance.
- Use [behavioral evaluations](references/evaluations.md#behavioral-evaluations) to check outcomes.
