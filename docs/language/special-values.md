# Null, omission, and return types

Nullability, omitted arguments, unfinished implementations, and the absence of a return
value express different contracts. They are not interchangeable.

!!! note "Design documentation"
This chapter describes the language design. Examples are illustrative; implementation
availability and unresolved rules are explained in [Design status](../about/status.md).

## Strong constants

`true`, `false`, `sentinel`, `null`, and `void` are distinct built-in singletons.
They exist for the program lifetime, cannot be garbage collected, and can be
checked by identity. These guarantees describe language-visible identity;
implementations may use immediate representations rather than heap allocations
provided that identity behavior remains the same.

- `true` and `false` are the ordinary Boolean values, spelled in lowercase.
- `sentinel` means a function argument was not passed. There is exactly one
  sentinel across the language, not one per type or parameter.
- `null` is a present None-like value. Every type can explicitly admit it; it can
  be stored and propagated wherever permitted by the type contract.
- `void` represents absence and cannot be stored in arrays/sequences or mappings.
  Other restrictions on storing it as an ordinary payload still apply.

Being an immortal singleton does not make every constant legal in every context.
In particular, sentinel propagation and void storage remain prohibited.

## Special-value and result contracts

These have distinct roles:

- `null` is available for any base type only when explicitly permitted.
- Ellipsis is accepted by the compiler as an argument regardless of base type,
  requires no opt-in annotation, and is truthy.
- The global `sentinel` is admissible as an omission default regardless of a
  parameter's type, but cannot propagate as an ordinary value.
- `void` means no value/result, including an absent collection lookup result, and
  cannot be stored as an ordinary value.
- `NoReturn` means the function raises for every accepted input; it has no normal
  result, including no `void` result.

`null` is a present value equivalent in role to Python's `None`; `void` expresses
absence. A possibly missing lookup has result type `T | void`. If `T` permits
null, the cases remain distinct: a concrete value, present `null`, or absent
`void`. No public `Found`/`Missing` wrapper is required for lookup.

```arbor
let result = records.lookup(key);
if result is void {
    handle_missing();
} else {
    use_present(result);
}
```

The check syntax is provisional. The compiler tracks the no-value alternative and
requires it to be handled or propagated through a compatible result type before
using the result as a definite `T`. The test above removes only `void`, not any
nullable alternative in `T`. This is not a truthiness check: zero, false, an empty
string, and null do not mean a missing entry.

An exclusively `-> void` function returns control normally on successful
completion but has no bindable value. `let x = save()` is invalid when `save`
returns only void. `save()` is an ordinary call statement; any standalone-call
warning is linter-only and may be explicitly suppressed with `noqa`. Writing,
logging, and sleeping require no suppression. Suppression does not affect the
call's execution, as described in [Compilation and toolchains](../tooling/compilation.md).

A local result binding such as `result` above may track the absence alternative for
inspection, narrowing, and return propagation; this does not turn absence into a
storable collection or field value. Narrowing away `void` or supplying a storable
fallback is required before storage. Boxing or widening to a dynamic type must not
evade this rule. Runtime representation and the complete rules for passing no-value
results remain open; `void` is not automatically accepted wherever a value is
required.

The ability to inspect a `T | void` lookup result does not permit binding an
exclusively void call: the former has a possible value to handle, whereas the
latter never supplies one.

`NoReturn` instead specifies an exceptional outcome for every accepted input:

```arbor
fun fail(message: string) -> NoReturn
    throws InvalidArgument
{
    throw InvalidArgument(message);
}
```

There is no `NoReturn` value to store or inspect. An always-raising expression can
occupy a position expecting a value of any type because it never supplies a normal
value; its exception effects still apply. Normal return and fallthrough are
invalid. Nontermination alone does not satisfy this always-raising contract.
Exact compiler verification, including possible divergence during evaluation,
remains open; a general termination proof is not implied by the annotation.

```arbor
fun add(x: null mut int, y: int) -> int { ...; }
fun configure(limit: int = sentinel) {
    prepare_configuration();
    if limit is sentinel {
        ...;
    }
}
```

The first argument to `add` permits null and mutation; `y` permits neither. A
function accepting `mut` arguments cannot be pure. Sentinel handling may be
delayed until later in the function; there is no mandatory entry-time check.
The compiler tracks the potentially omitted parameter until the code checks or
resolves it before an operation requiring an actual typed value. The sentinel
cannot be forwarded to another function, returned, or stored as an ordinary value.
These restrictions belong to parameter handling, while the sentinel's identity
is global. This supersedes the original parameter-specific sentinel proposal.

## Unfinished code and ellipsis values

Standalone `...` marks unfinished implementation and raises `NotImplementedError`
in development/test execution. That use is prohibited in production builds.
Passing, forwarding, and inspecting an ellipsis value is a separate capability.

**Open:** the final interaction between universally admissible ellipsis values,
operations requiring concrete data, and the production ban on unfinished-code
failures. A proposed rule was to raise `NotImplementedError` on concrete use, but
this needs reconciliation with the later production restriction. Ellipsis must
not silently become permission to ship unimplemented bodies.

## Related reading

- [Functions and calls](functions.md)
- [Exceptions and contracts](contracts.md)
