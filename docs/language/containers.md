# Containers

Tuples, lists, sets, and mappings cover the built-in collection shapes. Ownership and
freezing determine which operations are safe and which values can serve as keys.

!!! note "Design documentation"
This chapter describes the language design. Examples are illustrative; implementation
availability and unresolved rules are explained in [Design status](../about/status.md).

## Selecting a collection

Use a tuple when positions have fixed roles, a list for a runtime-sized sequence,
a set for uniqueness, and a mapping to associate keys with values. Insertion
order in a set or mapping is an iteration guarantee; it does not settle the
still-open equality rules for those containers.

A lookup distinguishes a present `null` from absence (`void`). A storable fallback
removes the no-value alternative. Required-entry indexing has a separate contract.

## Shapes and literals

Four built-in container shapes are selected; names remain provisional:

| Shape             | Purpose                                       | Illustrative spelling |
| ----------------- | --------------------------------------------- | --------------------- |
| Tuple             | Fixed positions; homogeneous or heterogeneous | `(int, string)`       |
| Growable sequence | Homogeneous runtime-sized collection          | `list[int]`           |
| Set               | Unique elements                               | `set[int]`            |
| Mapping           | Unique keys with associated values            | `map[string, int]`    |

There is no separate fixed-size array type: tuples cover the fixed-size shape.
Tuples can have mutable slots or be deeply frozen. Tuple shape alone does not
make the contents immutable.

```arbor
let values = [1, 2, 3];
let pair = (1, "hello");
let singleton = (1,);
let names = set{"alice", "bob"};
let counts = {"alice": 3, "bob": 2};
```

Expected types guide element literals. Empty literals require enough context for
inference. `{}` denotes a mapping; `set{}` denotes a set. Exact construction and
mutation syntax remain provisional.

## Iteration, keys, and mutation

Container rules:

- Mappings and sets preserve insertion order during iteration.
- Replacing a mapping value preserves its position; removing and reinserting moves
  the entry to the end.
- Keys and set elements are deeply frozen and hashable.
- Hashes and equality depend on frozen contents, not mutable external state.
- Frozen containers can be hashable if all relevant contents are hashable.
- Mutable objects are allowed as mapping values.
- Lookup without a fallback returns the stored value when present and `void` when
  missing, including for collections permitting stored `null`. Its result type is
  `T | void`. An explicit storable fallback removes the no-value alternative.
- Requiring an entry with indexing is a separate operation. The current direction
  is an undeclared contract violation on absence, rather than propagating a
  checked lookup exception through every caller; exact error names remain open.
- Duplicate mapping-literal keys are errors; explicit update operations replace
  existing values.
- Structural mutation during active borrowed iteration is prohibited in safe code.

Protocols expose collection capabilities without requiring one concrete
implementation. Homogeneous tuple notation, heterogeneous runtime indexing,
slicing ownership, collection equality, and precise hash contracts remain open.

## Sequences as mappings

An array-like sequence is conceptually a mapping from the contiguous integer
domain `[0, length)` to its element type. Contiguous physical storage is an
implementation detail, not the user-facing definition. This does not add a
separate fixed-size array type.

Sequences and general mappings share predictable lookup semantics:

```arbor
sequence.lookup(index);  // Element or void.
mapping.lookup(key);     // Value or void.
```

The working read-interface proposal exposes `lookup`, `contains_key`, `keys`,
`values`, `items`, and `length`. Explicit traversal methods avoid imposing one
default iteration meaning on every container. A sequence can satisfy a read-only
`Mapping[int, T]` interface without satisfying unrestricted mutable mapping
operations: arbitrary key insertion could violate its contiguous domain.
Heterogeneous tuples need position-dependent types rather than losing precision
through a simple union of all element types. Exact protocols remain provisional.

`void` is unstorable: it cannot be a collection entry or ordinary object-field
payload. An expression whose type still includes `void` cannot be assigned to such
storage. Assigning it is a compilation error, not an implicit deletion. Removal
uses an explicit operation. Stored `null` remains allowed when the destination
type permits it.

Lookup accepts an explicit fallback, consistently for sequences and mappings:

```arbor
foo["yolo"] = bar.lookup(21);        // Error: result may be void.
foo["yolo"] = bar.lookup(21, null);  // Valid if foo accepts the value type and null.
foo["yolo"] = bar.lookup(21, 0);     // Use zero only when the entry is absent.
```

Without a fallback, lookup returns `T | void`. With a fallback of storable type
`D`, the result is `T | D` (simplified when the alternatives coincide). Providing
`null` yields `T | null`; it does not bypass the destination's type or mutation
requirements. Fallbacks are used only for missing entries, never to replace a
present null, zero, false, or empty value. An explicit fallback may also be passed
by keyword under Arbor's ordinary argument-binding rules; the parameter spelling
is provisional. Passing `void` as the fallback does not satisfy the storable
fallback contract.

## Related reading

- [Ownership, borrowing, and freezing](ownership.md)
- [Types and generics](types.md)
- [Conditionals, indexing, and loops](control-flow.md)
