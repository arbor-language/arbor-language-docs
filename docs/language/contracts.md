# Exceptions and contracts

Operational failures, invalid external input, and broken programmer obligations need
different treatment. Declared exceptions and checked contracts make those distinctions
visible.

!!! note "Design documentation"
This chapter describes the language design. Examples are illustrative; implementation
availability and unresolved rules are explained in [Design status](../about/status.md).

## Choosing a failure representation

| Situation                                                                   | Representation                           |
| --------------------------------------------------------------------------- | ---------------------------------------- |
| An expected absence or end of iteration                                     | Ordinary control flow or a result value  |
| An operation fails in a way its callers may need to handle                  | A declared ordinary exception            |
| External input fails parsing or domain validation                           | An ordinary result or declared exception |
| A caller breaks a precondition, or an implementation breaks a postcondition | `ContractViolation`                      |
| A test expectation is false                                                 | A test-only assertion                    |

For example, an invalid uploaded date is external-input validation. It should
not become a contract violation merely because the parser cannot accept it.
A precondition on an internal API can instead express a caller obligation.

Arbor prefers exceptions over return codes for exceptional failures, with explicit
contracts for ordinary escaping exceptions. Expected conditions have ordinary
control-flow representations. Iteration does not use a fake `StopIteration` error.

## Fundamental categories

| Type                  | Contract                                                                                   |
| --------------------- | ------------------------------------------------------------------------------------------ |
| `BaseException`       | Abstract root; cannot be thrown or caught directly                                         |
| `Exception`           | Cannot be thrown; catch-only umbrella for ordinary exceptions                              |
| `NotImplementedError` | Undeclared development/test failure; unfinished implementations are rejected in production |
| `RuntimeError`        | Unexpected runtime/platform invariant failure, potentially including memory corruption     |
| `KeyboardInterrupt`   | Generated for Ctrl-C; cannot be explicitly thrown                                          |
| `SystemExit`          | Immediate exit, default status 1                                                           |
| `ContractViolation`   | Undeclared programmer-contract failure, outside the ordinary catch-all                     |

Runtime errors, keyboard interruption, and system exit are outside `Exception`.
Catch-all application handling must not accidentally swallow them. Contract
violations can be caught explicitly, including by tests and application boundaries.
Detailed handling rules for other fundamental categories remain open.

## Throw and warn keywords

`throw` and `warn` are language keywords. Both operate on instantiated objects:

```arbor
throw AccountLockedError();
warn DeprecatedIn71Warning();
```

`throw` raises an exception and transfers control according to the exception
handling and declared-exception rules. `warn` submits a warning to diagnostic
reporting and execution continues normally. Warning classes are distinct from
exception classes; emitting a warning does not itself add an exception to a
function's declared exception contract.

The operand may also be an existing instance, so a diagnostic can be constructed
separately from its use:

```arbor
let notice = DeprecatedIn71Warning("foo_function");
warn notice;
```

Passing a class instead of an instance is a compilation error. Thrown exception
instances remain subject to the restrictions on fundamental categories above:

```arbor
throw InvalidArgument("Expected a positive value");
throw SystemExit();
throw SystemExit(status=0);
throw SystemExit; // Error: this is a type, not an instance.
```

Exception selectors in handlers are types; the instantiation rule applies to
throwing, not to selecting a handler.

## Typed diagnostic classes and construction

Errors and warnings retain reusable classes with typed contextual fields and
message templates. A diagnostic class defines its wording once, and instances
supply its contextual values. Standard logging can accept a diagnostic instance
and use the same message and structured fields; callers do not need to duplicate
message text, diagnostic identity, or default severity. Warning reporting uses
the standard diagnostic/logging facilities, including Secret redaction.

All concrete, user-constructible error and warning classes support bare
initialization. A bare instance must be a valid diagnostic with a renderable
message. Required contextual data therefore needs valid defaults or explicit
absence handling; the exact fallback-template mechanism remains open. This
construction guarantee does not remove restrictions on abstract or runtime-only
fundamental categories.

Diagnostic constructors accommodate standard `log` and `stacklevel` parameters,
using `sentinel` to distinguish omission from an explicit value. `log` controls
logging during construction. `stacklevel` selects the caller location attributed
to a report and does not alter the exception's traceback. The parameters have
consistent meanings across classes; compiler support and their default policies
remain to be specified. The usual sentinel rules still apply: omission must be
resolved during initialization, and sentinel cannot be stored in instance fields.

Supporting a common zero-argument constructor does not require identical
additional parameters across subclasses. Code constructing arbitrary diagnostic
classes with contextual arguments must require an appropriate factory signature.

Detailed reporting policy remains open, including repetition/filtering,
attribution to a later `warn` statement, and duplicate-report handling when an
instance was already logged during construction. Reporting policy must be
reconciled with checked exception contracts and the pure-function diagnostic
exemption. Exact template binding and diagnostic protocol spellings are also open.

## Development and runtime failures

Production builds reject standalone unfinished-body ellipses and explicit throws
of `NotImplementedError` in included production code, even behind a currently
false condition. Tests are excluded from that production-code check. Exact
dependency/build-artifact enforcement remains to be specified.

Actual memory corruption may make exception construction or cleanup unsafe and
require a fail-fast runtime path. Immediate `SystemExit` has no normal cleanup
guarantee. Ctrl-C cancellation and cleanup policy remain open.

## Declared operational exceptions

Ordinary exceptions escaping a function belong in its declared contract. The
compiler checks catching and propagation. Internal effect inference can assist
implementation, while public APIs remain explicit. Precise syntax, hierarchy
coverage, generic effects, and the implicit built-in failure set remain open.

Asking whether an operation is valid is preferred to using exceptions for routine
branching. For concurrent or OS operations, a preliminary check is not atomic;
APIs still need atomic operations and meaningful operational errors. The compiler
may prove or move redundant checks rather than mandating repeated checks in loops.

## Validation without signature pollution

```arbor
validate(x, positive);

pure fun square(x: int) -> int
    ensures(result >= 0)
    throws ArithmeticOverflow
{
    return x * x;
}
```

Syntax is provisional. Contracts express API obligations and promised results.
Squaring promises nonnegative, not strictly positive, because zero is valid.

- Preconditions constrain arguments; postconditions constrain successful results.
- The compiler proves contracts when possible and otherwise checks them at runtime.
- Unproved checks remain enabled in production.
- Failure raises `ContractViolation` without adding it to propagated `throws` lists.
- Callers need not add wrappers or statically prove every obligation.
- A caller precondition failure is distinguished from an implementation failing
  its postcondition.
- Predicates are pure; machine-readable contract expressions are retained in
  compiled metadata where possible.
- Redundant checks can be removed only when proven unnecessary. Opaque predicates
  may still have runtime costs.

Malformed external input is not automatically a programming mistake: a parser or
domain-validation API reports expected invalid input as an ordinary result or
declared exception. It cannot evade its operational contract by relabeling every
bad input a contract violation.

Assertions are test-only. Production validation uses contracts or `validate`.

## Related reading

- [Handling failures and resources](exception-handling.md)
- [Functions and calls](functions.md)
- [Testing](../tooling/testing.md)
