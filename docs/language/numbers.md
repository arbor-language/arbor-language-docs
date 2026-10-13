# Numbers and constants

Choose numeric types according to range, representation, and exactness. Numeric literals
adopt an expected type; already typed values do not silently change numeric families.

!!! note "Design documentation"
This chapter describes the language design. Examples are illustrative; implementation
availability and unresolved rules are explained in [Design status](../about/status.md).

## Choosing a representation

Use `int` for the default fixed-width signed integer, `bigint` when values may
exceed fixed-width ranges, and explicitly sized integers for storage or foreign
interfaces. Use binary floating point for computations that accept binary
rounding. A fractional literal without context is `decimal`, so write an
explicit floating-point type when that is what you intend.

```arbor
let whole = 24; // int, which is i64.
let fraction = 1.4; // decimal.
let approximation: f64 = 1.4;
let too_large: u8 = 256; // Error: the literal does not fit.
```

Do not infer a decimal division or rounding policy from the word “exact.”
Those arithmetic rules remain open, especially for results with no finite
decimal representation.

## Numeric families

| Name                              | Meaning                                                           |
| --------------------------------- | ----------------------------------------------------------------- |
| `int`                             | Alias for `i64`                                                   |
| `float`                           | Alias for `f64`                                                   |
| `i8`, `i16`, `i32`, `i64`, `i128` | Fixed-width signed integers                                       |
| `u8`, `u16`, `u32`, `u64`, `u128` | Fixed-width unsigned integers                                     |
| `bigint`                          | Arbitrary-precision signed integer, bounded by resources          |
| `f32`, `f64`, `f128`              | Binary floating-point formats                                     |
| `decimal`                         | Exact decimal-oriented numeric type; arithmetic policy still open |
| Rational type                     | Exact fractions; final name and literal/division rules open       |

The platform direction is modern 64-bit targets. Narrow value types remain
important for data transport, storage, arrays of values, and foreign interfaces.
No 256-bit types are planned for v0.1. Arbitrary-precision binary floating point
and its precision context remain a possible later standard-library facility.

Numeric category protocols are distinct from concrete aliases: the provisional
names `Integer` and `BinaryFloat` describe families of types. `int` does not mean
any integer type, and `float` does not include decimal arithmetic.

`f128` means actual IEEE binary128, including software implementation where
necessary. Wide types do not imply native hardware speed. Fixed-width integer
overflow is checked, including optimized builds; explicit wrapping arithmetic
needs a defined API.

## Contextual numeric literals

A numeric literal is initially exact and adopts an explicitly expected type.
Without such a context, integer literals infer `int` and fractional literals
infer `decimal`.

```arbor
let count = 24;                  // i64
let exact = 1.4;                 // decimal
let sample: f32 = 1.4;
let precise_float: f128 = 1.4;
let factors: list[f32] = [1.4, 2.8];
```

Construction rounds directly to the selected floating-point format rather than
passing through another runtime numeric type. The proposed rounding rule is
nearest, ties to even. Out-of-range literals are compilation errors.

Context includes parameter types, returns, and collection elements. Already
typed values do not silently convert between numeric types. Ambiguous overloads
must have deterministic resolution rules; no arbitrary widening is permitted.

Exact decimal division, decimal precision/rounding contexts, rational syntax,
cross-type equality/hashing, and detailed numeric overload resolution remain open.

## Constants

`const` guarantees compile-time evaluation and a deeply frozen result:

```arbor
pure fun times(x: int, y: int) -> int {
    return x * y;
}

const HOURS_PER_DAY = 24;
const SECONDS_PER_DAY = times(times(HOURS_PER_DAY, 60), 60);
const LIMITS = {"uploads": 8, "timeout_seconds": SECONDS_PER_DAY};
```

Initializers use literals and pure computations on compile-time inputs. Operators
qualify through their purity. The compiler needs the implementation of any
function being evaluated. Nested values and mapping keys are frozen.

Uncaught exceptions, unsupported evaluation, or exceeding a compile-time resource
budget are compilation errors; evaluation is not silently deferred to startup.
Diagnostic logging during evaluation is suppressed by default or exposed through
an optional compiler trace, and is not replayed by the resulting program.

## Related reading

- [Modular arithmetic](modular-arithmetic.md)
- [Purity and caching](purity.md)
- [Exceptions and contracts](contracts.md)
