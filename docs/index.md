# Arbor

Arbor is a compiled language designed for readable programs, explicit contracts,
and runtime performance. Its tools share a single command, `arbor`, and its
standard library includes support for collections, numbers, time, concurrency,
configuration, and testing.

> We trust programmers to make deliberate choices. We design defaults for tired
> programmers making accidental ones.

```arbor
fun add(x: int, y: int) -> int {
    return x + y;
}

let total = add(1, y=2);
```

!!! note "First documentation edition"
This site describes Arbor's language design, not a released implementation.
Syntax marked **provisional** may change, and **open** rules remain unresolved.
Examples illustrate semantics and have not been checked by an Arbor compiler.
See [Design status](about/status.md) for the full boundaries.

## Start reading

- [Language tour](getting-started/tour.md): learn the core ideas through short examples.
- [Functions and calls](language/functions.md): read signatures and bind arguments.
- [Ownership and freezing](language/ownership.md): understand mutation and shared data.
- [Exceptions and contracts](language/contracts.md): distinguish operational failures from broken obligations.
- [Projects and tools](tooling/projects.md): understand project structure and the intended workflow.

## Language reference

| Area          | Chapters                                                                                                                                                                              |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Values        | [Numbers and constants](language/numbers.md), [modular arithmetic](language/modular-arithmetic.md), [strings and secrets](language/strings.md), [temporal values](language/time.md)   |
| Data modeling | [Containers](language/containers.md), [types and generics](language/types.md), [null and omission](language/special-values.md), [classes](language/classes.md)                        |
| Behavior      | [Functions](language/functions.md), [loops and indexing](language/control-flow.md), [purity and caching](language/purity.md), [decorators and settings](language/decorators.md)       |
| Boundaries    | [Ownership](language/ownership.md), [contracts](language/contracts.md), [handling failures and resources](language/exception-handling.md), [tasks and async](language/concurrency.md) |

## Tooling and libraries

An Arbor project uses `manifest.yaml`, production source in `src/`, and tests in
`tests/`. Test dependencies and privileges are kept out of production code.
Packages use scoped names and exact versions; published releases are immutable.

Read about [testing](tooling/testing.md), [dependencies](tooling/packages.md),
[package versions](tooling/versions.md), and [compilation and toolchains](tooling/compilation.md).
The [standard-library overview](library/index.md) records the intended library
scope without assuming finalized module names.

The [principles](about/principles.md) explain the language's direction. The
[open-decisions catalog](about/status.md#unresolved-specification-work) identifies
what needs further specification.
