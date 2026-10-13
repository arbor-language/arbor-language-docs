# Classes and inheritance

Classes provide nominal types and implementation reuse. Multiple inheritance uses a
predictable method order; behavioral substitutability remains an explicit obligation.

!!! note "Design documentation"
This chapter describes the language design. Examples are illustrative; implementation
availability and unresolved rules are explained in [Design status](../about/status.md).

## Access and class forms

Members are public by default. Read and write access can be restricted separately,
including private methods and private setters. The access policy is:

> Unnecessary privacy is poor practice. Restrict access when it protects a
> concrete invariant or boundary, and document that reason.

Tests receive elevated access to inspect internal coherence without bypassing
memory safety. Production code does not receive test privileges. Singletons and
frozen objects are supported.

Classes support multiple inheritance. Mixins may participate in inheritance, cannot
be instantiated on their own, and a class declaration inheriting only mixins
remains a mixin until it has a concrete class ancestor. Exact syntax and the
interaction with an implicit root class remain open.

## C3 and explicit method selection

C3 linearization defines method resolution, including mixins. Declared base order
matters. Inconsistent hierarchies are compilation errors. The computed order
should be inspectable in tooling and documentation.

```arbor
class Pegasus extends Bird, Horse {
    fun voice use Horse;
}
```

This spelling is provisional. C3 normally prefers Bird over Horse here, while
`voice` explicitly selects the implementation resolved through Horse.

- The selected provider must be an ancestor providing a compatible method.
- Selection changes only that method, not the class's global linearization.
- The receiver remains the actual Pegasus object.
- Calls through `self` retain ordinary dynamic dispatch.
- `super` follows the actual object's C3 order, starting after the class that
  defined the executing implementation.
- Signature, exceptions, and contracts remain visible.

There is no separate Horse-only resolution chain created by method selection.

## Substitutability

Intentional departures from Liskov substitutability are supported explicitly.
Implementation inheritance and substitutability must be distinguishable. The
circle/ellipse problem is a legitimate modeling issue.

An explicitly non-substitutable subclass can reuse implementation without
automatically being accepted wherever the parent's behavioral contract is needed.
Exact declaration syntax and conversion rules remain open. When substitutability
is claimed, overrides cannot strengthen preconditions, weaken postconditions, or
broaden declared exceptions.

Shared-ancestor storage, field conflicts, constructors, destruction, default
overridability, and the detailed inheritance ABI remain open. C3 alone does not
settle these matters.

## Related reading

- [Types and generics](types.md)
- [Decorators, accessors, and settings](decorators.md)
- [Exceptions and contracts](contracts.md)
