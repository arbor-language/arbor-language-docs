# Types and generics

Static types describe both individual values and relationships between values. Protocols
express capabilities, while repeated type parameters preserve a shared type choice.

!!! note "Design documentation"
This chapter describes the language design. Examples are illustrative; implementation
availability and unresolved rules are explained in [Design status](../about/status.md).

## Choosing between a union and a type parameter

A union describes the alternatives a single value may have. Separate
union-typed parameters allow separate choices. A repeated type parameter ties
several positions to one selection.

For the finite `int or float` choice below, `(int, int)` and `(float, float)`
are permitted argument pairs; `(int, float)` is not. For two `int | float`
parameters, that mixed pair is permitted by the parameter types. The body
still has to implement the combinations it accepts.

## Type vocabulary

| Concept             | Meaning                                                          |
| ------------------- | ---------------------------------------------------------------- |
| Nominal class       | A type identified by its declaration, rather than only its shape |
| Structural protocol | An interface described by required capabilities                  |
| Generic             | A declaration parameterized by types and their relationships     |
| Union               | A value may have any one of several listed types                 |
| Finite type choice  | One invocation selects a concrete alternative used consistently  |
| Recursive type      | A type definition refers to itself through its components        |
| Variance            | The rules for substituting related types in a generic interface  |

## Supported direction

Arbor takes a pragmatic approach to static typing: nominal classes, structural
protocols, generics, complex and recursive types, and enums are required. The
working proposal also includes unions, intersections, tagged enums with payloads,
and an explicit dynamic boundary; their complete rules remain to be specified.

```arbor
type Json = null | bool | int | decimal | string | list[Json] | map[string, Json];
```

This is an illustrative recursive type. The actual JSON number representation
must account for large numbers and remains part of the numeric/data-format work.

## Enums, automatic values, and flags

**Required capabilities:** enums support `auto()`, uniqueness enforcement, and
flag enums. The declaration spellings and detailed semantics below are the
working proposal.

```arbor
@unique
enum Status {
    PENDING = auto(),
    RUNNING = auto(),
    COMPLETE = auto(),
}
```

For ordinary integer-valued enums, `auto()` starts at `1`. Each subsequent automatic
value is one greater than the highest preceding assigned value, including explicit
assignments. Thus `FIRST = 10, SECOND = auto(),` assigns `SECOND` the value `11`.
An unrepresentable automatic value is a compilation error, not a wraparound.
`auto()` is a compiler-recognized enum initializer, not a runtime counter.

Without `@unique`, duplicate underlying values are permitted as aliases for the
same member. With `@unique`, every declared member must have a distinct underlying
value; duplicates are compilation errors, including explicitly named aliases.
Uniqueness is checked after constant evaluation and automatic value assignment.

```arbor
@unique
@flags
enum Permission {
    NONE = 0,
    READ = auto(),
    WRITE = auto(),
    EXECUTE = auto(),
    READ_WRITE = READ | WRITE,
}

let rights = Permission.READ | Permission.WRITE;
let can_read = (rights & Permission.READ) == Permission.READ;
```

For flag enums, `auto()` starts at `1` and then selects the next power of two above
the highest bit used by preceding values. In this example the values are `0`, `1`,
`2`, `4`, and `3`. A preceding composite does not make the next automatic member a
composite: after values `1`, `2`, and `3`, the next automatic value is `4`.

- Flags use a nonnegative integer bitmask representation. Combining or testing
  bits does not require allocating a collection. A zero member such as `NONE` is
  explicit; `auto()` never chooses zero.
- `|`, `&`, and `^` combine values of the same flag type and return that flag type.
  Unnamed combinations are valid. Ordinary enums do not acquire these operations.
- The known-bit mask is the bitwise OR of all declared member values. Complement
  `~value` flips only bits in that mask, not unrelated bits of the storage integer.
- `@unique` checks complete values, not whether values share bits. Consequently
  `READ_WRITE = READ | WRITE` is valid alongside `READ` and `WRITE`, but a second
  name with value `3` is rejected.
- `@flags` and `@unique` are compiler-recognized declaration modifiers. Their
  relative decorator order does not change the meaning.

Enums are nominal types, with immutable, hashable values. An enum member is not
implicitly an integer, and values from different enum types cannot be mixed just
because their underlying integers match. Conversion to or from the representation
is explicit. Conversion into an ordinary enum validates membership; conversion
into flags rejects unknown bits. Exact conversion APIs and failure reporting
remain open.

Enum bodies contain comma-separated member declarations, with trailing commas
allowed. Member values are compile-time constants; `auto()` is the special
automatic initializer. References to earlier members can define aliases and flag
composites. Automatic numbering follows declaration order, so APIs that promise
stable wire or stored numeric values should assign those values explicitly.

A switch over an ordinary enum must cover its distinct members, counting aliases
only once. Flags also admit unnamed combinations: covering every named member
alone does not establish exhaustiveness. Use an `else` arm unless the compiler can
prove all valid bit patterns are covered.

Tagged enums with payloads remain a separate part of the type-system proposal;
flag enums cannot carry payloads. Default backing widths, explicit representation
annotations, string-backed enums, iteration/reflection, serialization, and payload
syntax remain open.

## Unions versus correlated choices

An ordinary union permits arguments to choose independently:

```arbor
type Number = int | float;
fun foo(x: Number, y: Number) -> Number { ...; }
```

A finite type choice preserves the relationship between arguments and result:

```arbor
choice T = int or float;
fun foo(x: T, y: T) -> T { ...; }
```

The choice syntax is provisional. Its meaning is the pair of signatures:

```arbor
fun foo(x: int, y: int) -> int;
fun foo(x: float, y: float) -> float;
```

Each invocation selects one concrete alternative. Every occurrence of `T` uses
that alternative; the compiler must not choose `T = int | float` to make a mixed
call fit. A shared body is checked for every alternative. Separate overload bodies
are also supported when implementations differ. Contracts and exceptions belong
to the selected alternative.

Repeated type parameters express relationships in general generics too:

```arbor
fun find_latest_by_updated[K: Hashable](
    table: Table[K],
    keys: list[K],
) -> null Row[K] {
    ...;
}
```

Integer-keyed tables require integer keys; string-keyed tables require string keys.
No silent conversion or widening defeats the relationship. Distinct parameters
allow independent choices. Nominal key types can further distinguish identifiers
with the same underlying representation.

Union-typed inputs need narrowing unless every possible combination is supported.
Ambiguous calls are compilation errors. Protocol bounds and finite choices should
remain distinct concepts even if the final syntax unifies their notation.

## Variance and mutation capabilities

Full mutable containers remain invariant. However, the compiler should eventually
understand verified restrictions on mutation:

```arbor
fun duplicate[K](x: mut list[K]) {
    let original_length = x.length;
    for i in 0..<original_length {
        x.append(x[i]);
    }
}
```

Distinguish preserving an element type `K` from the stronger promise of using only
elements already present. Reordering, removing, and duplicating existing values
can preserve an unknown element type without granting arbitrary insertion rights.
Capability annotations must be checked, not blindly trusted.

```arbor
class StatementParser[S: Statement] {
    statements: list[S];
}

class CreditCardStatementParser
    extends StatementParser[CreditCardStatement] {
    ...;
}
```

Inherited operations must remain valid under specialization. Narrowing a mutable
field cannot leave an inherited method able to insert a broader type. Read-only
interfaces can be covariant; restricted mutable interfaces may expose proven
type-preserving operations.

**v0.1 boundary:** conservative rejection is acceptable. The language should leave
room for more capable verification later without making current mutable-container
rules unsound. Exact capability syntax and proof machinery are open.

## Related reading

- [Classes and inheritance](classes.md)
- [Containers](containers.md)
- [Exceptions and contracts](contracts.md)
