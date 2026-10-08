# Documentation decisions

## Contract documentation

Document assumptions that are not enforced by types or ordinary control flow. Public C++20-and-newer
APIs need an ownership/retention contract, invalidation conditions, valid-input domain, error behavior,
thread-safety policy, cancellation semantics, and observable complexity where those are consequential.
Do not repeat a self-explanatory signature or promise checks that the implementation does not perform.

Doxygen contracts should distinguish checked errors from caller preconditions. A size check can reject
mismatched spans but cannot establish their owner's lifetime or nonoverlap. A `noexcept` declaration
means escaping exceptions terminate; document why the implementation can honor that promise.

For nontrivial resource members, document dependency/destruction order. For atomics, document what
state an operation publishes and which acquire observes it. For a generic API, document semantic
requirements such as strict weak ordering that a concept cannot prove. Put such decisions beside the
relevant declaration/definition instead of in distant narrative.

## Examples and version maintenance

Keep examples focused, self-contained, and explicit about owner lifetime. Include required headers.
Prefer references and bounded views; document owner lifetime for required borrowed raw pointers.
Do not use owning raw pointers or manual allocation in examples. Use a concrete triggering
input or shutdown sequence when explaining a bug. Avoid introducing a full runtime merely to illustrate
one cancellation property.

Label examples by the minimum supported standard. C++23/C++26 examples need an immediately stated
C++20 alternative and any lost contract. Verify snippets in the promised mode. Tooling instructions
should state platform/library prerequisites rather than repeat a stale universal version table.

## Completion evidence

Record meaningful tests run and gaps, not an inventory of routine commands. For numerical changes,
state tolerance and tested input distributions. For optimizations, include equivalent-output checks,
compiler/library/flags/hardware, and variation. For concurrent changes, state lifecycle schedules tested
and distinguish executed race detection from a proof of ordering/progress.

Use direct topic links from AGENTS.md. Supporting documents may link their own headings or primary
external sources, but must not require another local reference to complete a rule. Keep examples next
to the decision they explain; a separate examples tree is unnecessary for these small snippets.
