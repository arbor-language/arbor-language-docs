# Functions and calls

Function signatures describe parameter types and results. Callers may bind parameters
positionally or by name without changing the meaning of the call.

!!! note "Design documentation"
This chapter describes the language design. Examples are illustrative; implementation
availability and unresolved rules are explained in [Design status](../about/status.md).

## Reading a signature

In `fun add(x: int, y: int) -> int`, `fun` introduces a function,
`x` and `y` name its parameters, and `-> int` describes its result.

The body uses braces and `return` supplies the result. For example:

```arbor
let total = add(y=2, x=1);  // Binds x to 1 and y to 2.
add(1, x=2);               // Error: x is bound twice.
```

Keyword order can change evaluation order even when parameter bindings are
equivalent. Argument expressions are evaluated in their written order.

## Source and call rules

- UTF-8 source and Unicode support are required.
- Examples use `fun`, braces, type annotations, and local type inference.
- Semicolons terminate statements; there is no automatic semicolon insertion.
- Trailing commas are allowed in comma-separated syntactic lists.
- Public APIs expose signatures and contracts explicitly.
- Every argument can be supplied positionally or by keyword.
- Argument expressions evaluate in source order.
- Duplicate bindings of the same parameter are errors.
- After binding, calls have one canonical argument representation.

```arbor
fun add(x: int, y: int) -> int {
    return x + y;
}

add(1, 2);
add(x=1, y=2);
add(1, y=2);
```

These calls bind the same parameter values, which matters for inspection, mocks,
tracing, and caching. Omission remains distinguishable from explicitly supplying
a value when a parameter uses the single global `sentinel` as its default.

Arguments are read-only unless marked `mut`. A reassignable binding and permission
to mutate the object it references are separate concepts. The working convention
is `let` for non-reassignable bindings and `var` for reassignable bindings; the
complete declaration and borrowing syntax remains provisional.

## Required statement terminators

Simple statements end in `;`, including bindings, assignments, calls, return,
throw, warn, break, continue, assertions, imports, and non-block declarations.
A missing terminator is a syntax error; a newline does not insert one.

```arbor
let total = calculate(
    first,
    second,
);

if total > 0 {
    write("Positive");
}
```

This is statement termination, not a requirement to put a semicolon on every
physical line. Function/class definitions and compound statements such as if,
for, while, loop, try, and with end with their block structure, without an extra
semicolon. Field declarations inside classes do end with semicolons. Decorator
annotations and function signature clauses are not separate statements.

When a braced expression is part of an assignment, the assignment still needs its
terminator: `let value = switch subject { ... };`. Switch arms are separated by
commas, including an allowed trailing comma after the final arm. Their result
expressions are not individually terminated by semicolons. Statements inside any
arm block follow the ordinary semicolon rule; exact arm-value syntax remains open.

The formatter puts separate statements on separate lines and places a statement's
semicolon after its final expression token, before an end-of-line comment. It does
not silently repair missing terminators by guessing boundaries from newlines.

## Trailing commas and Black-style formatting

The formatter follows Black's structural approach to trailing commas:

- Calls, parameter lists, tuples, sequences, sets, mappings, switch arms, and other
  comma-separated productions permit a final comma where the grammar allows it.
- A multiline comma-separated list is normally formatted with one item per line,
  a trailing comma, and a closing delimiter aligned with the construct's opening
  context. Nested expressions are formatted recursively.
- An existing optional trailing comma is a **magic trailing comma**: it requests
  the expanded layout even when the contents would fit on one line.
- A short list without a trailing comma may remain compact. Once an expanded list
  has a trailing comma, it remains expanded until that optional comma is removed.
- Commas required for meaning, particularly `(value,)` for a one-element tuple,
  are preserved and do not alone force expansion. `(value)` remains grouping.
- Formatting never changes a tuple, index, argument, or other expression's meaning
  merely to add or remove a comma. A trailing comma does not permit an extra empty
  element, argument, or mapping entry.

For example, `send(first, second,);` formats as:

```arbor
send(
    first,
    second,
);
```

Formatting is deterministic and idempotent. Comments and string contents retain
their meaning. Arbor adopts these formatting conventions within its own grammar;
default line width and remaining layout details are still to be specified.

## Related reading

- [Ownership, borrowing, and freezing](ownership.md)
- [Null, omission, and return types](special-values.md)
- [Exceptions and contracts](contracts.md)
