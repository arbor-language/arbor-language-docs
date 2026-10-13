# Language tour

This tour introduces Arbor's intended programming model. It is a reading guide:
there is no released toolchain or runnable installation procedure established by
the design baseline. Complete grammar, some spellings, and several APIs remain
open. The reference chapters call out those boundaries.

## Read a function

Arbor source files use the `.arb` extension and UTF-8 text. A function begins
with `fun`, declares parameter types, and encloses its body in braces:

```arbor
fun add(x: int, y: int) -> int {
    return x + y;
}

let total = add(1, 2);
let same_total = add(x=1, y=2);
```

`int` is an alias for `i64`, a signed 64-bit integer. The result annotation
`-> int` describes the value returned by the function. Local bindings can infer
types; public APIs expose their signatures and contracts explicitly. Simple
statements end in `;`; newlines do not insert terminators. Block definitions and
compound statements need no extra semicolon. Comma-separated lists permit trailing
commas. See [Statement terminators](../language/functions.md#required-statement-terminators)
and [Formatting](../language/functions.md#trailing-commas-and-black-style-formatting).

All parameters can be supplied by position or keyword. Arguments evaluate in
source order, and binding the same parameter twice is an error. Equivalent
bindings have a canonical representation for inspection, mocks, and caching.

Fixed-width overflow is checked even in optimized builds. Short examples such
as this one may omit exception annotations; the full `throws` grammar and the
implicit built-in failure set remain open.

Read [Functions and calls](../language/functions.md) and
[Exceptions and contracts](../language/contracts.md) for the complete rules.

## Choose the numeric type

A literal begins as an exact value. An expected type determines its
representation; without one, integer literals become `int` and fractional
literals become `decimal`.

```arbor
let count = 24; // int
let exact = 1.4; // decimal
let sample: f32 = 1.4; // Rounded directly to f32.
let samples: list[f32] = [1.4, 2.8];
```

This contextual typing applies to literals. A value that already has a numeric
type does not silently convert to another numeric type. Out-of-range literals
are compilation errors. Decimal arithmetic policy is still open; do not assume
that every decimal computation produces a finite exact result.

[Numbers and constants](../language/numbers.md) covers widths, floating-point
formats, and compile-time evaluation.

## Use collections and explicit ranges

```arbor
let values = [1, 2, 3];
let pair = (1, "hello");
let names = set{"alice", "bob"};
let counts = {"alice": 3, "bob": 2};
```

Lists are homogeneous runtime-sized sequences. Tuples have fixed positions and
can hold different types. Sets contain unique elements; mappings associate
unique keys with values. Sets and mappings preserve insertion order. Their
elements and keys, respectively, must be deeply frozen and hashable.

Indexing starts at zero and rejects negative indices. Ranges mark their end
explicitly:

```arbor
0..<5; // Excludes 5: 0, 1, 2, 3, 4.
1..=5; // Includes 5: 1, 2, 3, 4, 5.
values[0..<2]; // The first two elements.
```

Out-of-bounds indexing is currently an undeclared contract violation. Ordinary
lookup returns the stored value or `void` on absence, distinguishing missing entries
from stored `null`. A storable fallback, such as `counts.lookup("carol", 0)`, removes
the absence alternative.

Slicing is being reconsidered in favor of clipping nonnegative endpoints to the
sequence length and returning an empty slice when the end is at or before the
start. Final endpoint rules and result ownership remain open.

See [Containers](../language/containers.md) and
[Conditionals, indexing, and loops](../language/control-flow.md).

## Choose a conditional value

```arbor
let label = enabled ? "enabled" : "disabled";

let message = switch response.status {
    case 200 => "Success",
    case 404 => "Not found",
    else => "Unexpected response",
};
```

A ternary evaluates only its selected branch; nested ternaries require explicit
parentheses. A switch evaluates its subject once, selects the first matching arm,
and has no implicit fallthrough. Value-producing switches must be exhaustive.
See [Conditional expressions](../language/control-flow.md).

## Write output and diagnostics

```arbor
write("Saved");
log.info("Saved record", record_id=record.id);
write(term'An unrecoverable $bold${red}error$reset occurred.');
```

`write` defaults to stdout. Standard-library logging is available with usable
defaults and optional manifest configuration. The `term` literal describes styled
text; terminal rendering produces escapes or plain text as appropriate. See
[Output and diagnostics](../library/index.md#output-and-diagnostics-write-and-log)
and [Terminal styled text](../library/index.md#terminal-styled-text).

## Separate binding, borrowing, and freezing

The working convention uses `let` for a non-reassignable binding and `var` for
a reassignable one. Declaration and borrowing syntax remain provisional.
A binding's reassignment rule does not by itself establish whether its data is
mutable.

A mutable value has an owner. The owner can lend multiple read-only borrows or
one exclusive mutable borrow. While a read-only borrow is active, the borrowed
data cannot be changed. After the borrow ends, its owner can mutate again.

Deep freezing consumes mutable ownership and makes the reachable value graph
immutable. Eligible frozen values can be shared without deep copying. This is
useful for stable mapping keys, configuration, and sharing between tasks.

[Ownership, borrowing, and freezing](../language/ownership.md) explains these
rules and the separate obligations of raw pointers and unsafe code.

## Make compile-time work explicit

`const` requires compile-time evaluation and a deeply frozen result. Pure
functions can participate when their implementation and inputs are available
to the compiler:

```arbor
pure fun times(x: int, y: int) -> int {
    return x * y;
}

const HOURS_PER_DAY = 24;
const SECONDS_PER_DAY = times(times(HOURS_PER_DAY, 60), 60);
```

Pure functions cannot accept mutable arguments, perform observable mutation or
I/O, read clocks or randomness, or schedule work. Diagnostic logging is a narrow
exception whose occurrence is not guaranteed.

Purity also permits runtime caching, but it does not guarantee a cache entry.
Entries are local to a process and may be evicted. Stable keys and sound result
ownership are additional requirements.

Read [Purity and caching](../language/purity.md) before relying on that behavior.

## Distinguish temporal meanings

```arbor
let birthday = date'2000-04-18';
let opening = time'09:30:00';
let event = instant'2026-10-11T09:30:00Z';
let elapsed = duration'PT24H';
let calendar_step = period'P1D';
```

A date has no time of day. A local clock time is not a timeline instant.
A duration measures elapsed time; a calendar period changes calendar fields.
Adding one calendar day can differ from adding 24 hours across a daylight-saving
transition. Named-zone literals contain both an offset and a zone so the compiler
can check their agreement.

[Dates, times, and timezones](../language/time.md) lists all temporal literal forms
and the unresolved range, precision, and calendar-arithmetic policies.

## Handle failures according to their meaning

Ordinary operational exceptions escaping a function belong in its declared
contract. Expected absence has an ordinary result representation. Programmer
obligations use preconditions and postconditions, checked at runtime when the
compiler cannot prove them.

A failed contract raises `ContractViolation`, which is outside the ordinary
`Exception` catch-all. Malformed external input should be reported as an
ordinary validation result or declared exception. Assertions are test-only.

Context managers provide acquisition and cleanup:

```arbor
with open("report.txt") as file {
    process(file);
}
```

This example assumes the surrounding APIs and setup. Entry, body, and cleanup
effects contribute to the enclosing function's contract. Cleanup runs on ordinary
scope exits and exception unwinding; immediate termination cannot guarantee it.

Continue with [Exceptions and contracts](../language/contracts.md) and
[Handling failures and resources](../language/exception-handling.md).

## Observe a task's completion

Calling an async function starts it immediately and returns a promise. Assuming
an async `upload` function and a suitable file value:

```arbor
let pending = upload(file);
let result = await pending;
```

Repeated awaits observe the same completed operation; they do not execute it
again. Frozen results can be shared. Uniquely owned mutable results require
controlled observation and a one-time ownership transfer, whose `take()`
spelling is provisional.

`upload.now(file)` starts and waits, and `upload.fire(file)` explicitly detaches
work. Awaiting is allowed from synchronous functions too. Cancelling a waiter
cancels the wait, not automatically the underlying task.

[Tasks and async](../language/concurrency.md) covers completion, ownership,
failures, and the remaining runtime decisions.

## Organize a project

```text
manifest.yaml
src/
    main.arb
tests/
    main_test.arb
```

This illustrates an application layout; exact entry-point and module rules
remain open. Tests live separately and can inspect private state without
bypassing memory safety. Production source cannot import test-only modules or
dependencies, including indirectly.

The intended commands are `arbor build`, `arbor run`, `arbor test`, `arbor fmt`,
`arbor lint`, and `arbor doc`. They describe the official tooling interface;
this edition does not claim that they are available to install or execute.

Continue with [Projects and official tools](../tooling/projects.md),
[Testing](../tooling/testing.md), and [Packages and dependencies](../tooling/packages.md).
