# Compilation and toolchains

Compilation favors runtime performance and reuse of previous work. Toolchain bundles
coordinate independently released official components.

!!! note "Design documentation"
This chapter describes the language design. Examples are illustrative; implementation
availability and unresolved rules are explained in [Design status](../about/status.md).

## Compiler front end and editor grammar

The compiler uses **Flex** for lexing and **Bison** for parsing. This selects the
front-end generators, not the bootstrap language, Bison parsing mode, or backend.

Tree-sitter uses a separate incremental grammar and its own lexer/parser. The
Flex/Bison definitions provide a precise reference, but are not a drop-in source
for Tree-sitter generation. Productions, token boundaries, and language precedence
must agree; semantic actions and parser-specific conflict recovery need not match.

Develop both grammars against a shared corpus of `.arb` syntax examples. Check
equivalent recognized structure rather than requiring identical concrete trees.
Add editor-specific cases for incomplete and malformed input, plus incremental
edits. Define boundaries around numeric ranges, fractional literals, typed literal
prefixes, and triple-quoted strings explicitly. Introduce a Tree-sitter external
scanner only where its built-in lexing facilities are insufficient.

Neither grammar should require type resolution, ownership analysis, C3 execution,
or runtime decorator evaluation to recover the source structure. These are later
semantic operations, with results exposed through compiler tooling.

## Compilation priorities

Arbor prioritizes runtime performance and reuse of compiled work over minimum
clean-build latency. The compiler may spend meaningful effort on specialization,
contract proofs, redundant-check elimination, and optimization.

Incremental artifact identities account for source contents, compiler version,
target, options, and relevant dependencies. Unchanged work is reused. Dependency
implementation changes should not automatically rebuild every consumer, but
inlining, specialization, and compile-time evaluation must track the exact
implementation facts they depend on.

Ordinary calls execute according to their contracts whether or not their results
are bound. An unused result is not permission to remove observable behavior:

```arbor
save();                      // Executes; may receive a linter warning.
save(); // noqa: standalone-call
write("Saved");              // Defaults to stdout; no suppression needed.
log.info("Saved");           // No suppression needed.
sleep(duration'PT1S');        // No suppression needed.
let result = save();         // Error if save returns only void.
```

The proposed `discard` statement and optimization-dependent call omission are
withdrawn. At every optimization level, the compiler preserves required argument
evaluation, exceptions, I/O, mutation, synchronization, and control flow. Inlining
and proven behavior-preserving elimination remain permitted. Pure-function caching
and its diagnostic-logging exemption retain their separately specified rules.

## Standalone-call linting

A standalone call can receive a linter warning about its intentionally unused
result or effect-only use. This is a linter policy, not a compiler warning or an
execution modifier. Explicit `noqa` suppression acknowledges the intended call.
The rule identifier and precise comment syntax above are provisional.

Standard-library writing, logging, and sleeping are exempt by default: users need
neither an execution keyword nor a suppression comment for them. Exemptions should
follow resolved API identities, including import aliases, rather than matching
arbitrary functions merely named `write`, `log`, or `sleep`.

Linting and `noqa` never change execution or suppress type errors. In particular,
binding an exclusively void result remains a compiler error. Async detachment
still uses `.fire()`; suppressing a lint does not implicitly detach a promise.
The full rule scope, extension/annotation mechanism for other APIs, and diagnostic
codes remain to be specified.

## Independently patched tools

Official components share a supported **major.minor** series, but have independent
patch releases:

```text
compiler   0.3.5
formatter  0.3.7
linter     0.3.2
docs       0.3.4
lsp        0.3.6
```

A supported toolchain bundle selects a tested combination of exact component
versions. Projects can pin the bundle and override components. Cross-series
combinations are allowed but untested; actual protocol or artifact incompatibility
must produce a useful diagnostic. Matching-series patches preserve compatibility.

Editor marketplace versions may be independent while advertising the supported
Arbor series. Toolchain manifest keys and the bundle format remain provisional.

## Related reading

- [Numbers and constants](../language/numbers.md)
- [Purity and caching](../language/purity.md)
- [Versions and publication](versions.md)
