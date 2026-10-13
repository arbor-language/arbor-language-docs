# Conditionals, indexing, and loops

Arbor supports conditional values, zero-based indexing, explicit range boundaries,
and structured loops. Loop else has a precise empty-iteration meaning.

!!! note "Design documentation"
This chapter describes the language design. Examples are illustrative; implementation
availability and unresolved rules are explained in [Design status](../about/status.md).

## Ternary expressions

Arbor uses `condition ? consequent : alternative` for a short conditional choice.
The earlier suggested Python-style `value if condition else fallback` expression
is not selected. Ordinary if/else statements remain available.

```arbor
let label = enabled ? "enabled" : "disabled";
```

The condition is evaluated once and only the selected branch is evaluated. The
result follows the normal type rules, including expected types and branch effects.

Nested ternaries require explicit parentheses. A ternary directly used as another
ternary's condition or either branch must be grouped; implicit left/right
association is not a substitute for grouping:

```arbor
let value = x ? y : z ? a : b;       // Error: unparenthesized nesting.
let value = x ? y : (z ? a : b);     // Valid, explicit grouping.
let other = x ? (z ? a : b) : y;     // Valid, explicit grouping.
```

The formatter preserves required parentheses and expands nested choices across
lines. Exact indentation is a formatter detail; an illustrative rendering is:

```arbor
let value = x
    ? y
    : (
        z ? a : b
    );
```

These are compile-time syntax rules, not a linter-only nesting prohibition.
Remaining precedence against other expression forms must be specified in the
grammar. Multiple alternatives are also expressible through switch.

## Switch expressions

Switch is a Rust-style branching construct that can produce a value:

```arbor
let message = switch response.status {
    case 200 => "Success",
    case 404 => "Not found",
    case 503 => "Temporarily unavailable",
    else => "Unexpected response",
};

let category = switch response.status {
    case 200..<300 => "success",
    case 400..<500 => "client error",
    case 500..<600 => "server error",
    else => "other",
};
```

- Evaluate the subject once and execute the first matching arm.
- There is no implicit fallthrough and no required break between arms.
- Value-producing switches must be exhaustive. A switch over an ordinary enum may
  omit `else` when all distinct members are covered; aliases do not add cases.
  Flag enums additionally require coverage of unnamed valid combinations.
- Diagnose unreachable arms and check branch results against the expected type.
- Arms are comma-separated, with trailing commas allowed; the formatter expands
  every switch into multiple lines and includes a trailing comma on the last arm.
- A switch is not a loop and does not add a level to `break N`.

Value/range patterns are part of this direction. Tagged-enum payload binding and
guards should be supported, but their exact syntax, exhaustiveness analysis,
multi-statement arm syntax, and rules for non-value-producing switches remain open.

## Indexing

Indexing is zero-based. Negative indices are invalid. The current direction treats
out-of-bounds indexing as an undeclared contract violation (`IndexError`), while
ordinary lookup returns `void` on absence. General mappings follow the same
required-entry versus possibly-missing distinction. Exact exception hierarchy
names remain provisional. Explicit last-element operations cover end-relative use.

Ranges distinguish exclusive and inclusive ends:

```arbor
0..<5;             // 0, 1, 2, 3, 4
1..=5;             // 1, 2, 3, 4, 5
values[2..<5];     // Elements 2, 3, 4
```

Slices are half-open. The earlier rejection of out-of-range endpoints has been
reopened in favor of forgiving slicing: requesting an unavailable range can yield
an empty result. The proposed rule clips nonnegative endpoints to the sequence
length and returns an empty slice when the end is at or before the start. Negative
indexing remains disallowed. Final endpoint rules, range steps, open-ended slices,
and exact result ownership remain open.

## Loop forms

```arbor
for value in values {
    process(value);
}

for (index, value) in values.enumerate() {
    ...;
}

while connection.is_open {
    process_next();
}

loop {
    if finished {
        break;
    }
    process_next();
}
```

`while` is retained; `loop` expresses unconditional repetition. C-style three-part
`for` is omitted from the v0.1 direction. Infinite iterators remain useful when
they supply values. Ordinary iteration lends read-only elements; explicit mutable
iteration lends exclusive mutable access. Structural mutation is disallowed while
the relevant iteration borrows are active.

## Break depth

`break` means `break 1`. `break N` exits exactly N lexically enclosing loops. N is
a positive integer literal, and excess depth is a compilation error. It never
denotes a returned value. Cleanup runs for each exited ordinary scope.

```arbor
loop {
    for item in items {
        if finished(item) {
            break 2;
        }
    }
}
```

`continue` is supported. Depth-qualified continue has not been selected.

## Empty-iteration else

```arbor
for user in users {
    display(user);
} else {
    display("No users");
}
```

This has Jinja-style meaning: else runs only if iteration produces zero elements.
It does not mean Python's completion-without-break rule.

- One element followed by break does not trigger else.
- Immediately continuing still counts as having received an element.
- Iterator acquisition/advancement failures propagate and do not trigger else.
- The else block is outside that loop when calculating break depth.
- The rule applies to async iteration too.
- Else is not added to while or loop.

## Related reading

- [Containers](containers.md)
- [Ownership, borrowing, and freezing](ownership.md)
- [Handling failures and resources](exception-handling.md)
