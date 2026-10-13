# Design status and open decisions

This is the first documentation edition, based on the language design baseline. It
describes intended behavior and does not certify that a compiler or any individual
feature has been implemented.

## How to read this documentation

- **Selected** rules describe the design baseline.
- **Provisional** identifies syntax, names, or mechanisms that may change.
- **Open** identifies behavior that has not been settled.
- **v0.1 boundary** records an initial limit or a deferred capability.

Examples explain semantics; the complete grammar has not been frozen. Short
examples may omit setup or unrelated exception annotations. `...` is a
placeholder for development examples, not a finished production implementation.
No installation procedure or executed Arbor example is claimed in this edition.

The reference covers the whole design baseline. Inclusion here does not imply
availability in v0.1. The standard-library page likewise lists intended
capabilities rather than invented import paths or signatures.

## Unresolved specification work

The following areas need further specification. An example spelling does not
resolve an open rule. The topic chapters preserve local qualifications alongside
the selected semantics.

| Area                            | Still to settle                                                                                                                                                                                                                     |
| ------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Compiler architecture           | Bootstrap language, backend, initial OS/architecture, internal representations, artifact formats                                                                                                                                    |
| Parser integration              | Bison parsing mode, shared syntax corpus, recovery rules, Tree-sitter scanner requirements                                                                                                                                          |
| LLM tooling                     | Structured diagnostics and API export schemas, check-only interface, evaluation corpus and measures                                                                                                                                 |
| Memory                          | Reclamation, cycles, lifetime notation/inference, moves/copies, thawing, resource freezing                                                                                                                                          |
| Unsafe/FFI                      | ABI, pointer types, pinning, retained addresses, trusted wrapper contracts                                                                                                                                                          |
| Grammar                         | Final declaration, modifier, borrowing, decorator, and choice spellings; full comma-separated production inventory                                                                                                                  |
| Formatting                      | Default line width and remaining layout details under mandatory semicolons and Black-style trailing-comma rules                                                                                                                     |
| Conditional syntax              | Full ternary precedence; switch guards, payload binding, arm blocks, exhaustiveness analysis, and statement-only cases                                                                                                              |
| Enums                           | Confirm automatic numbering and alias policy; backing widths, representation annotations, conversion APIs, string-backed enums, iteration/reflection, serialization, and tagged payload syntax                                      |
| Numeric semantics               | Decimal division/precision, fraction literals, promotion, equality/hash, wrapping APIs, literal overload resolution                                                                                                                 |
| Modular values                  | Final type/API names, extraction target-type inference and runtime validation errors, modulus-mismatch errors, and implementation of runtime moduli and inverse results                                                             |
| Strings                         | Representation, Unicode indexing, interpolation, normalization, detailed escapes, mutation operations                                                                                                                               |
| Temporal values                 | Precision/range, leap-second policy finalization, period arithmetic policies, geographic lookup scope                                                                                                                               |
| Containers                      | Shared mapping protocols, lookup/fallback ownership, homogeneous tuples, heterogeneous runtime indexing, slice endpoints/ownership, empty accessors, equality/hash details                                                          |
| Classes                         | Shared ancestor storage, constructor/destructor order, field conflicts, default overrides, mixin/root interaction                                                                                                                   |
| Accessors                       | Lazy-cell effects, frozen-object eligibility, class cache ownership, setter signatures                                                                                                                                              |
| Effects                         | Full throws grammar, built-in exceptions, generic effect checking, exceptional control events, always-raising NoReturn verification                                                                                                 |
| Warnings and diagnostic classes | Template binding and bare-init fallback messages, shared diagnostic protocol, constructor reporting defaults, warn attribution, filtering/repetition, duplicate-report handling, and interaction with checked exceptions and purity |
| Lint policy                     | Standalone-call rule scope and codes, noqa syntax, extension mechanism for effect-oriented APIs                                                                                                                                     |
| Ellipsis                        | Runtime value behavior versus production unfinished-code rejection                                                                                                                                                                  |
| Contracts                       | Predicate rules, metadata, proof reuse, blame diagnostics, runtime failure representation                                                                                                                                           |
| Variance                        | Initial supported proofs and future checked capability annotations                                                                                                                                                                  |
| Async                           | Scheduler, cancellation/cleanup, blocking bridges, promise ownership and failure representation                                                                                                                                     |
| Caching                         | Eligibility, key snapshots, mutable results, eviction controls, context identity                                                                                                                                                    |
| Testing                         | Fixture teardown, patch implementation, mutable call snapshots, resource isolation, coverage details                                                                                                                                |
| Packaging                       | Local-label grammar, resolver details, registry/publishing protocols, source overrides, lockfile schema                                                                                                                             |
| Configuration                   | Supported YAML subset, schema implementation, bootstrap use of settings, logging defaults/schema, restricted dev-only manifest fragments                                                                                            |
| Output                          | Writer protocols, newline/encoding/buffering, write return conventions and errors, term preset declarations, value interpolation, full style vocabulary and renderer option APIs                                                    |
| Toolchain                       | Supported bundle format, compatibility protocol, exact submodule boundaries/remotes                                                                                                                                                 |

## v0.1 boundaries

These are limits and priorities, not a promise that every other designed feature
will ship in the first implementation:

- No 256-bit numeric types.
- No separate fixed-size array type.
- Full YAML can wait; manifest support cannot.
- Logging defaults and base-manifest configuration are core; `manifest.yaml.d/`
  development overrides may be deferred beyond v0.1.
- Separate static accessors and borrowed-key containers are deferred.
- Advanced mutation/variance proofs can reject valid-but-unproved cases initially.
- Ordinary exact dependency pins precede any possible future range syntax.
- Code reuse through incremental compilation matters more than exceptionally fast
  clean builds.
- Tooling, testing, and diagnostics are part of the product, not optional polish.
- LLM-friendly generation, editing, and repair are explicit design and evaluation
  requirements from the first release.

The implementation plan should turn the selected semantics into small accepted
and rejected examples, define the unresolved foundations, and stage features
without pretending the entire language already fits into a minimal first release.

## Related reading

- [Principles](principles.md)
- [Compilation and toolchains](../tooling/compilation.md)
