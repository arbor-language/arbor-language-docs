# Ownership, borrowing, and freezing

Ownership controls access to mutable data. Borrowing provides temporary access; deep
freezing makes an entire eligible value graph immutable and shareable.

!!! note "Design documentation"
This chapter describes the language design. Examples are illustrative; implementation
availability and unresolved rules are explained in [Design status](../about/status.md).

## Three separate questions

| Question                                                           | Mechanism                                         |
| ------------------------------------------------------------------ | ------------------------------------------------- |
| Can this name be rebound to another value?                         | Binding declaration (`let` or `var`, provisional) |
| Can this access path change the referenced value?                  | Ownership or an exclusive mutable borrow          |
| Can any safe access path change this value or its nested contents? | Deep freezing prohibits this                      |

A non-reassignable binding alone does not establish deep immutability. Conversely,
a read-only borrow prevents mutation while the borrow is active without freezing
the value permanently. Freezing consumes mutable ownership, so it requires more
than changing the annotation on one reference.

## Ownership and borrowing

Distinguish binding reassignment, read-only access, and deep freezing:

- A mutable value has an owner.
- Its owner may lend multiple read-only borrows or one exclusive mutable borrow.
- Active read-only borrows prohibit mutation of the borrowed data.
- Read-only borrowing is temporary; the owner may mutate again after it ends.
- A frozen value permits only read-only access, including through nested values.
- Freezing consumes mutable ownership, after outstanding checked borrows have ended.
- Frozen values can have shared ownership without deep copying.

Deep freezing covers the reachable value graph, not just the outer container.
Ordinary classes should support freezing when their contents do. Resources and
classes containing raw pointers need explicit eligibility rules; not every value
can automatically become a deeply frozen value.

Keys and set elements are owned by their containers or held through shared frozen
ownership, so they remain alive and stable. Borrowed-key containers are deferred.
Lifetime syntax, cycle support, memory reclamation, and thawing remain open.

## Pointers

Checked references are the default. Raw pointers remain available for systems
interoperability and deliberate low-level work. Dereferencing, pointer arithmetic,
and foreign operations that cannot uphold checked guarantees require `unsafe`.

> In safe code, a frozen value and its reachable contents cannot change. Unsafe
> code must preserve that invariant.

Writing into frozen storage through a raw pointer violates the language contract;
it is not a supported unfreezing operation. The compiler may rely on immutability.

Raw pointees do not automatically join the deep-freeze graph. Pointer-containing
objects do not automatically qualify for freezing, purity-based caching, or
constant evaluation. A trusted abstraction can establish those capabilities by
proving the relevant ownership, lifetime, and immutability obligations.

Freezing a wrapper around an external resource does not freeze that resource.
Escaped raw pointers impose obligations on unsafe code that the borrow checker
cannot generally verify. Address retention requires defined stability/pinning
rules, and foreign APIs must expose their ownership expectations.

Pure functions cannot freely read externally mutable raw memory. Foreign ABI,
pinning syntax, unsafe verification rules, and trusted abstraction mechanisms
remain open.

## Related reading

- [Containers](containers.md)
- [Purity and caching](purity.md)
- [Tasks and async](concurrency.md)
