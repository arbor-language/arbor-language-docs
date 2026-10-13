# Modular arithmetic

Modular values make the modulus part of the numeric type or context. Arithmetic
preserves that modulus and does not silently mix incompatible values.

!!! note "Design documentation"
This chapter describes the language design. Examples are illustrative; implementation
availability and unresolved rules are explained in [Design status](../about/status.md).

## Modular values

Integers modulo a positive integer are first-class numeric values. The semantics
below are agreed; names and syntax such as `Mod[n]`, `extract`, and `**` remain
provisional.

A modular value carries its modulus with it. Arithmetic automatically produces
the mathematically correct modular result, even when an unreduced intermediate
would exceed the underlying integer's range. Distinct statically known moduli are
distinct types, preventing accidental operations between `Mod[7]` and `Mod[11]`.
This removes repeated `% n` operations from application code and lets the compiler
specialize arithmetic for a known modulus.

```arbor
let x = Mod[7](6);
let y = Mod[7](3);
let sum = x + y;             // Congruent to 2 modulo 7
let product = x * y;         // Congruent to 4 modulo 7
let invalid = x + Mod[11](3);  // Compile-time error
```

## Representation and extraction

The internal representation is an implementation detail. A value represents a
congruence class; the language does not promise that its stored representation is
an integer in `[0, n)`. Construction accepts arbitrary integers, including negative
values, and preserves their congruence class without a narrowing overflow.
Equality and hashing include the modulus. There is no intrinsic order; ordering
integer representatives requires explicit extraction.

Extracting an integer requires an explicit range containing exactly `n` consecutive
integers. Extraction returns the unique integer in that range congruent to the
modular value. All values in the requested range must fit the destination integer
type. The compiler checks known ranges and types statically; remaining conditions
are validated at runtime. Integer conversion always requires this explicit range.

```arbor
let day = Mod[7](0);
let zero_based = day.extract(0..<7);  // 0
let one_based = day.extract(1..=7);   // 7

let x = Mod[7](6);
let unsigned = x.extract(0..<7);     // 6
let centered = x.extract(-3..=3);    // -1
```

For weekdays, Sunday represented as `0` or `7` works directly. Changing the
numbering origin, such as Monday=`0` to Monday=`1`, requires an explicit offset
before extraction. Selecting a range preserves congruence.

Conversion between different moduli requires extracting a representative and
constructing the destination value explicitly. Runtime moduli require runtime
compatibility checks before combining modular operands.

## Arithmetic and overflow guarantees

| Operation on `Mod[n]`                 | Right operand               | Semantics                                                                                               |
| ------------------------------------- | --------------------------- | ------------------------------------------------------------------------------------------------------- |
| Addition, subtraction, multiplication | Integer or another `Mod[n]` | Return the exact modular result, preserving the modulus                                                 |
| Division                              | Integer or another `Mod[n]` | Multiply by the divisor's inverse; declare `NonInvertibleError` if no inverse exists                    |
| Exponentiation                        | Integer                     | Preserve the modulus; a negative exponent requires an invertible base and declares `NonInvertibleError` |

Unary negation also preserves the modulus. For addition, subtraction,
multiplication, and division, integer operands are interpreted modulo `n`.
Integer exponents retain their full value and are not reduced modulo `n`.

Overflow protection covers operand reduction and all intermediate calculations,
including inversion and exponentiation. A valid modular operation must not fail
or return an incorrect result because an internal fixed-width intermediate
overflows. The implementation must use appropriate widening or overflow-safe
modular algorithms. It must also handle the most negative signed exponent without
overflow while obtaining its magnitude.

Division is modular division, and invertibility is required even when the divisor
is nonzero. For an integer representative `b`, an inverse exists exactly when
`gcd(b, n) = 1`. Negative operands are valid; a negative exponent uses the inverse
of its base. The provisional `try_inverse()` reports absence ordinarily, while
`inverse()`, division, and negative exponentiation declare `NonInvertibleError`
when inversion is required and unavailable.

```arbor
Mod[7](3) / -2;      // Congruent to 2
Mod[8](3) / 2;       // NonInvertibleError
Mod[7](3) ** -1;     // Congruent to 5
Mod[8](2) ** -1;     // NonInvertibleError
```

A modular value cannot be used directly as an exponent: congruent exponents can
give different results. For example, `1` and `6` are congruent modulo `5`, but
`Mod[5](2) ** 1` is congruent to `2` and `Mod[5](2) ** 6` is congruent to `4`.
Using a modular value as an exponent requires extracting an integer with an
explicit range first.

The guarantee starts when a modular operation receives its operands. In
`x + (a * b)`, where `x` is modular and `a` and `b` are ordinary integers, `a * b`
still follows ordinary integer overflow rules. Converting an operand before
multiplication brings that operation under the modular guarantee.

Compile-time moduli permit specialized representations and arithmetic. Runtime
moduli use shared immutable contexts with precomputed reduction parameters.
Large moduli can use `bigint`. Small values should avoid heap allocation.

Fast modular arithmetic does not imply constant-time cryptographic arithmetic.
Cryptographic timing guarantees require a separate explicit contract.

## Inversion is not ordinary division

For modulus 6, the nonzero value 2 has no multiplicative inverse: no integer
multiplied by 2 is congruent to 1 modulo 6. The value 5 does have an inverse,
because `5 * 5` is congruent to 1. Use the absence-reporting operation when
non-invertibility is expected, and the declared-exception operation when a
missing inverse is an operational failure.

## Related reading

- [Numbers and constants](numbers.md)
- [Exceptions and contracts](contracts.md)
