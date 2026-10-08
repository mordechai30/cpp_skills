# Task-based concurrency

## Contents

- [Task-first concurrency](#task-first-concurrency)
- [Owned inputs and result collection](#owned-inputs-and-result-collection)
- [Future inspection](#future-inspection)
- [Cancellation and retained work](#cancellation-and-retained-work)
- [Unavoidable shared state](#unavoidable-shared-state)
- [Coroutines and newer task APIs](#coroutines-and-newer-task-apis)

## Task-first concurrency

**C++20 baseline:** choose a task/result abstraction before synchronization primitives.

- A small bounded set of independent operations: use `async` with explicit `launch::async` when concurrent
  launch is required. Keep the returned futures and collect their results.
- Repeated CPU work: use the project's established bounded executor, with queue capacity/rejection policy.
  Do not create one asynchronous operation per element when batching can bound overhead and concurrency.
- Async I/O/composition: use the established runtime's task API and scheduler. `async` around blocking I/O
  does not become event-driven I/O, and a future is not automatically awaitable.
- Inputs: copy small values or move owned data. For large shared reads use an immutable snapshot, excluding
  other mutable aliases. Return owned results and combine them after completion.

Keep work synchronous when scheduling cost outweighs useful overlap/parallelism.
Do not make direct thread creation, manual joins, custom pools, or locks around shared output the default.
Do not assume `async` provides a standard pool or backpressure. The default policy may defer execution;
explicit `launch::async` can fail to launch, so preserve submission and resource-failure handling.

Authority: [CP.4: tasks](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#rconc-task),
[CP.3: shared data](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#rconc-data), and
[CP.31: value inputs](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#rconc-data-by-value).
Futures and async are established facilities used here in C++20, not features introduced in C++20.

## Owned inputs and result collection

**C++20 baseline:** the task owns its moved input; the caller receives an owned result. No shared
writable container or synchronization code is needed. Capturing a span instead would leave its storage
lifetime with the caller. The 16-bit domain bounds each widened square, not a general integer product.

```cpp
#include <cstdint>
#include <future>
#include <utility>
#include <vector>

/** Launches a bounded-domain transform with exclusive input ownership.
 * The future delivers owned output or the task's exception. */
[[nodiscard]] inline std::future<std::vector<std::int64_t>>
square_async(std::vector<std::int16_t> input) {
    return std::async(std::launch::async, [input = std::move(input)] {
        std::vector<std::int64_t> output;
        output.reserve(input.size());
        for (const auto value : input) {
            const auto wide = static_cast<std::int64_t>(value);
            output.push_back(wide * wide);
        }
        return output;
    });
}
```

**C++20 baseline:** share immutable input when copying a large dataset is undesirable. Each operation
returns its own result. Launch a bounded number; this example launches exactly two. Collect before
using the aggregate. On failure, remaining async futures still own their tasks and may wait in cleanup.

```cpp
#include <algorithm>
#include <cstddef>
#include <cstdint>
#include <future>
#include <memory>
#include <utility>
#include <vector>

/** Contains results produced independently from one immutable input snapshot.
 * Both values are complete when analyze_async returns. */
struct Analysis {
    std::size_t positive_count{}; ///< Number of strictly positive samples.
    std::size_t negative_count{}; ///< Number of strictly negative samples.
};

/** Computes independent counts without sharing writable state.
 * The snapshot has no mutable alias; futures retain it until their tasks finish. */
[[nodiscard]] inline Analysis analyze_async(std::vector<std::int16_t> input) {
    auto snapshot = std::make_shared<const std::vector<std::int16_t>>(std::move(input));
    auto positive = std::async(std::launch::async, [snapshot] {
        return static_cast<std::size_t>(std::count_if(snapshot->begin(), snapshot->end(),
            [](auto value) { return value > 0; }));
    });
    auto negative = std::async(std::launch::async, [snapshot] {
        return static_cast<std::size_t>(std::count_if(snapshot->begin(), snapshot->end(),
            [](auto value) { return value < 0; }));
    });
    return {positive.get(), negative.get()};
}
```

Do not infer the snapshot's immutability from a const handle alone. Here `make_shared<const T>` creates
const storage; no writable alias exists. A const view of externally mutable data does not give that guarantee.

## Future inspection

**C++20 baseline:** choose the operation by intent:

- `valid()`: determine whether the handle has a shared state; it says nothing about completion.
- `wait_for(0s)`: inspect ready/timeout/deferred without consuming the result. Do not busy-poll it.
- `get()`: wait and retrieve a value or rethrow a task exception; consumes a unique future.
- `shared_future`: allow several consumers to access one completed result; do not concurrently mutate it.

A future does not report task progress or provide cancellation. Timed waiting leaves the task running.
Some futures originating from async wait when the last reference is destroyed. Dropping temporary
futures can serialize calls; keep them until the application's completion point. Do not wait/get or
release a potentially waiting future while holding a lock needed by the task.

Default `async` policy may return a deferred state. If the application requires concurrent launch,
select `launch::async`; otherwise handle deferred work rather than waiting for it to start by itself.

## Cancellation and retained work

**Since C++20:** use a stop token with a task API that defines cancellation semantics. A request is
cooperative; the task must observe it. Do not advertise a hard deadline for a blocking operation that
cannot be interrupted. Distinguish task cancellation from timing out while awaiting a future.

```cpp
#include <cstdint>
#include <future>
#include <optional>
#include <stop_token>
#include <utility>
#include <vector>

/** Computes finite-domain samples with cooperative cancellation between elements.
 * Absence means cancellation; allocation failures still propagate through the future. */
[[nodiscard]] inline std::future<std::optional<std::vector<std::int64_t>>>
square_cancellable(std::vector<std::int16_t> input, std::stop_token stop) {
    return std::async(std::launch::async, [input = std::move(input), stop]()
        -> std::optional<std::vector<std::int64_t>> {
        if (stop.stop_requested()) { return std::nullopt; }
        std::vector<std::int64_t> output;
        output.reserve(input.size());
        for (const auto value : input) {
            if (stop.stop_requested()) { return std::nullopt; }
            const auto wide = static_cast<std::int64_t>(value);
            output.push_back(wide * wide);
        }
        return output;
    });
}
```

The token/source shared state remains valid through its own ownership; callers retain a stop source
when they need to request cancellation. A request arriving after the last check can race with successful
completion. Define that precedence instead of assuming requests always win. This example returns an
owned result and never commits partial output into caller state.

Retained work needs a runtime/task owner that collects completion before dependencies disappear.
Specify whether shutdown drains accepted work or cancels it, and how rejected submissions report errors.
Do not capture borrowed views, references, or `this` as a shortcut to owning asynchronous input.

## Unavoidable shared state

Apply this section only when task-local ownership, immutable snapshots, or result passing cannot meet
the requirement. Prefer a vetted concurrent abstraction before writing a synchronization protocol.

- Protect coupled data with a mutex bound to that data and named RAII guards. Do not call unknown
  callbacks or await a future while holding it. Atomic fields do not make a multi-field invariant atomic.
- **Since C++20:** stop-aware `condition_variable_any::wait` supports cancellation for a blocking
  runtime queue. Use a predicate; ordinary condition-variable waits do not become stop-aware automatically.
- **Since C++20:** atomic wait/notify can coordinate a state transition; notification alone is not a
  transition and waits have no stop-token overload. Use a known ordering protocol, not ad hoc weakening.
- Do not infer lock-free progress from `atomic<T>`. Verify support and use a vetted reclamation design
  if a measured requirement needs a concurrent container. Raw-pointer implementations are prohibited.

This is runtime implementation guidance, not a recommendation to replace async tasks with hand-managed
threads or low-level synchronization in application code.

## Coroutines and newer task APIs

**Since C++20:** use an established coroutine task type/runtime for suspension and composition; the
language machinery alone supplies no scheduler, cancellation, or frame-lifetime protocol. Pass owned
values that survive suspension. Do not implement a frame owner merely to make one operation async.

**C++23:** `generator` provides synchronous lazy production, not asynchronous execution.
**C++20 fallback:** range/callback production or the existing coroutine library.

**C++26:** senders/receivers can express value/error/stopped completion and composition when the selected
implementation and scheduler support them. **C++20 fallback:** an established async library with those
channels, or bounded futures-based tasks with explicit cancellation. Do not assume constructing a sender
starts work or supplies the application's I/O scheduler.

Verify task results, deferred behavior, error delivery, owned-input lifetime, cancellation before work,
and shutdown with pending work. Use bounded timeouts; TSan checks executed shared-state paths, not progress.
Primary semantics: [async](https://eel.is/c++draft/futures.async),
[future members](https://eel.is/c++draft/futures.unique.future), and
[stop tokens](https://eel.is/c++draft/thread.stoptoken).
