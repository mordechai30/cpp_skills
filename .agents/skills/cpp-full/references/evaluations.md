# Skill maintenance evaluations

## Behavioral evaluations

Run these requests in a temporary workspace with cpp-full available. Evaluate observable artifacts and behavior. Do not modify production code merely to run an evaluation. These cases are specifications, not claims of completed tests on every agent model.

## Constrained sorting

Request: “In C++20, write a projected range sort that rejects immutable input and supports a record projection.”

Pass criteria: the implementation constrains random access and sortable iterator/comparator/projection use; mutable records sort correctly; const input is rejected through the constraint; comparator semantics are documented. It does not require unnecessary nested iterator aliases.

Failure indicators: a const range reaches a body error, a proxy range is excluded without need, or a floating comparison is assumed total despite NaNs.

## Owned async work and cancellation

Request: “In C++20, run two independent analyses asynchronously, inspect task readiness, collect results/errors, and provide cooperative cancellation without hand-managed threads.”

Pass criteria: uses bounded async tasks or the established executor; each task owns its input or an immutable snapshot; futures are retained and consumed deliberately; valid/readiness/deferred states differ; cancellation has a separate protocol; results are combined after completion; no shared writable output or custom pool is introduced.

Failure indicators: substitutes jthread for task APIs, captures an escaping span/reference, drops an async future, treats timeout as cancellation, or launches unbounded per-element tasks.

## Portable typed parsing

Request: “The project targets C++20. Provide integer parsing with distinct empty, malformed and overflow errors. Show how to adopt expected when C++23 becomes supported.”

Pass criteria: baseline code compiles without expected; full input consumption is checked; error categories survive the C++23 change; partial success is rejected; the expected feature and fallback are labeled.

Failure indicators: parser returns a placeholder value, missing handling of trailing bytes, or expected appears unconditionally in a baseline header.

## Performance and C++26 choice

Request: “Optimize a fixed-capacity sample buffer in C++20. Consider inplace_vector and SIMD without adding a heap allocation.”

Pass criteria: baseline storage meets the occupancy and element-lifetime contract; C++26 features are labeled and gated; reserve is not presented as embedded storage; scalar output is verified for tail lengths; performance conclusions require measurements.

Failure indicators: unsupported standard headers, discarded tail elements, an always-constructed array used for a nondefault-constructible type without explanation, or claimed speedups without measurements.
