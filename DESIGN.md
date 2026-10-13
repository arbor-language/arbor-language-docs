# Arbor design decisions

This document records the language and toolchain choices made during Arbor's
initial design discussions. It belongs at the root of the `arbor/` repository.
It is a design baseline, not a claim that these features have been implemented.

Later decisions supersede earlier proposals. **Open** and **provisional** identify
questions or spellings that still need resolution. Examples illustrate semantics;
the complete grammar has not yet been frozen. A feature being part of the design
does not automatically commit its full implementation to v0.1.

Short examples may omit unrelated exception annotations and setup. Bodies using
`...` are development placeholders, not production-ready implementations.

## Contents

1. [Identity and principles](#1-identity-and-principles)
2. [Repository and official tools](#2-repository-and-official-tools)
3. [Compilation and toolchain versions](#3-compilation-and-toolchain-versions)
4. [Source syntax and calls](#4-source-syntax-and-calls)
5. [Numbers and constant evaluation](#5-numbers-and-constant-evaluation)
6. [Modular arithmetic](#6-modular-arithmetic)
7. [Strings and secret values](#7-strings-and-secret-values)
8. [Dates, times, and timezones](#8-dates-times-and-timezones)
9. [Containers](#9-containers)
10. [Ownership, freezing, and pointers](#10-ownership-freezing-and-pointers)
11. [Types, generics, and variance](#11-types-generics-and-variance)
12. [Null, ellipsis, sentinels, and void](#12-null-ellipsis-sentinels-and-void)
13. [Purity and runtime caching](#13-purity-and-runtime-caching)
14. [Classes and inheritance](#14-classes-and-inheritance)
15. [Decorators, accessors, and settings](#15-decorators-accessors-and-settings)
16. [Exceptions, validation, and contracts](#16-exceptions-validation-and-contracts)
17. [Context managers and exception handling](#17-context-managers-and-exception-handling)
18. [Tasks and async](#18-tasks-and-async)
19. [Conditional expressions, indexing, and loops](#19-conditional-expressions-indexing-and-loops)
20. [Testing](#20-testing)
21. [Packages and dependency groups](#21-packages-and-dependency-groups)
22. [Package versions and immutability](#22-package-versions-and-immutability)
23. [Standard library and data formats](#23-standard-library-and-data-formats)
24. [Open decisions and v0.1 boundaries](#24-open-decisions-and-v01-boundaries)

## 1. Identity and principles

- **Language:** Arbor.
- **Source extension:** `.arb`.
- **Unified command:** `arbor`.
- **Project manifest:** `manifest.yaml`.
- **Visual identity:** the supplied luminous tree whose roots become circuit
  traces. A simplified version can serve as an editor or small-format icon.

Guiding principle:

> We trust programmers to make deliberate choices. We design defaults for tired
> programmers making accidental ones.

Arbor is compiled, performance-oriented, readable, and straightforward to parse.
It includes a substantial standard library, formatter, testing framework, package
tooling, and editor support. It should be productive for humans and AI-assisted
development through predictable syntax, precise diagnostics, and inspectable
contracts.

Explicit decisions remain possible: raw pointers, intentional departures from
substitutability, mutation, and mixed tool versions all have a place. Important
boundaries should be visible in code or configuration.

### LLM-friendly language and tooling

Being extremely LLM-friendly is a first-class Arbor design requirement. It applies
to reading, generating, editing, and repairing programs. Success means producing
correct, reviewable changes with limited project context and reliable compiler
feedback. Human readability remains essential because developers review those
changes.

The following criteria guide further syntax and tooling decisions:

- Prefer consistent declarations, explicit boundaries, familiar constructs, and
  a small number of equivalent spellings. Optimize for predictable meaning rather
  than minimum character count or one model's tokenization.
- Keep syntax understandable without resolving types, running decorators, or
  loading application state. Type checking and effect verification follow parsing.
- Make local obligations visible: types, mutation rights, ownership, exceptions,
  and contracts. Export the same information for imported APIs in structured form.
- Expose effective signatures, decorator effects, and inheritance resolution so
  tools need not reconstruct them by reading an entire dependency tree.
- Give diagnostics stable identifiers, precise source spans, expected/actual
  information, related locations, and structured output. Offer edits only when
  their preconditions are known; do not guess a behavioral repair.
- Provide syntax/type/contract checking without requiring final code generation
  and linking. Reuse incremental compiler work for short edit/check cycles.
- Support symbol lookup, references, API discovery, documentation, and structured
  test results through shared compiler tooling and the LSP.
- Use canonical formatting and stable syntax trees to support small, localized
  edits. Parser recovery must remain useful while a construct is half written.
- Maintain concise, versioned language guidance with valid examples and examples
  of rejected code. Explicitly teach differences from familiar languages, such as
  Arbor's empty-iteration `for ... else` and its special exception categories.

Specific commands, diagnostic schemas, and export formats remain provisional.
LLM-friendliness must be evaluated empirically rather than inferred from keyword
choices: use generation, repair, and refactoring tasks checked by the compiler and
behavioral tests. Track successful completion, repair iterations, unintended
changes, and context requirements across representative models. Test these goals
alongside readability for humans rather than tuning syntax to one model release.

## 2. Repository and official tools

The `arbor/` directory is the umbrella repository root. Official tools live in
separate repositories included here as Git submodules. This document is maintained
in the umbrella repository as the shared design reference.

Illustrative layout; submodule paths and remote repositories are not yet fixed:

```text
arbor/
    DESIGN.md
    tools/
        compiler/          # Submodule
        formatter/         # Submodule
        linter/            # Submodule
        lsp/               # Submodule
        documentation/     # Submodule
        package-manager/   # Submodule: Cargo-like build/package/test interface
        toolchain-manager/ # Submodule: rustup-like installation and selection
    editors/
        tree-sitter/       # Submodule
        vscode/            # Submodule
        vim/               # Submodule: Vim/Neovim integration
        emacs/             # Submodule
```

Runtime, standard-library, and shared compiler-library repository boundaries
remain open. The layout above documents intent; it does not register submodules
or invent their remotes.

Official commands include:

```text
arbor build
arbor run
arbor test
arbor fmt
arbor lint
arbor doc
```

Language intelligence should be shared through compiler libraries and the LSP.
Editor integrations should avoid implementing independent semantic rules.

An ordinary Arbor project follows a strict production/test layout:

```text
manifest.yaml
src/
    main.arb
    lib.arb
tests/
    main_test.arb
```

Both entry files are shown as possible roles, not a requirement that every
project contain both. Module naming derives predictably from paths; exact import
and entry-point rules remain to be specified.

Production source cannot import tests, test-only APIs, or test dependencies,
including through intermediate modules. Test privileges and dependencies do not
become available merely because production source is compiled for a test run.

## 3. Compilation and toolchain versions

### Compiler front end and editor grammar

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

### Compilation priorities

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

### Standalone-call linting

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

### Independently patched tools

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

## 4. Source syntax and calls

- UTF-8 source and Unicode support are required.
- Examples use `fun`, braces, type annotations, and local type inference.
- Semicolons terminate statements; there is no automatic semicolon insertion.
- Trailing commas are allowed in comma-separated syntactic lists.
- Public APIs expose signatures and contracts explicitly.
- Every argument can be supplied positionally or by keyword.
- Argument expressions evaluate in source order.
- Duplicate bindings of the same parameter are errors.
- After binding, calls have one canonical argument representation.

```arbor
fun add(x: int, y: int) -> int {
    return x + y;
}

add(1, 2);
add(x=1, y=2);
add(1, y=2);
```

These calls bind the same parameter values, which matters for inspection, mocks,
tracing, and caching. Omission remains distinguishable from explicitly supplying
a value when a parameter uses the single global `sentinel` as its default.

Arguments are read-only unless marked `mut`. A reassignable binding and permission
to mutate the object it references are separate concepts. The working convention
is `let` for non-reassignable bindings and `var` for reassignable bindings; the
complete declaration and borrowing syntax remains provisional.

### Required statement terminators

Simple statements end in `;`, including bindings, assignments, calls, return,
throw, warn, break, continue, assertions, imports, and non-block declarations.
A missing terminator is a syntax error; a newline does not insert one.

```arbor
let total = calculate(
    first,
    second,
);

if total > 0 {
    write("Positive");
}
```

This is statement termination, not a requirement to put a semicolon on every
physical line. Function/class definitions and compound statements such as if,
for, while, loop, try, and with end with their block structure, without an extra
semicolon. Field declarations inside classes do end with semicolons. Decorator
annotations and function signature clauses are not separate statements.

When a braced expression is part of an assignment, the assignment still needs its
terminator: `let value = switch subject { ... };`. Switch arms are separated by
commas, including an allowed trailing comma after the final arm. Their result
expressions are not individually terminated by semicolons. Statements inside any
arm block follow the ordinary semicolon rule; exact arm-value syntax remains open.

The formatter puts separate statements on separate lines and places a statement's
semicolon after its final expression token, before an end-of-line comment. It does
not silently repair missing terminators by guessing boundaries from newlines.

### Trailing commas and Black-style formatting

The formatter follows Black's structural approach to trailing commas:

- Calls, parameter lists, tuples, sequences, sets, mappings, switch arms, and other
  comma-separated productions permit a final comma where the grammar allows it.
- A multiline comma-separated list is normally formatted with one item per line,
  a trailing comma, and a closing delimiter aligned with the construct's opening
  context. Nested expressions are formatted recursively.
- An existing optional trailing comma is a **magic trailing comma**: it requests
  the expanded layout even when the contents would fit on one line.
- A short list without a trailing comma may remain compact. Once an expanded list
  has a trailing comma, it remains expanded until that optional comma is removed.
- Commas required for meaning, particularly `(value,)` for a one-element tuple,
  are preserved and do not alone force expansion. `(value)` remains grouping.
- Formatting never changes a tuple, index, argument, or other expression's meaning
  merely to add or remove a comma. A trailing comma does not permit an extra empty
  element, argument, or mapping entry.

For example, `send(first, second,);` formats as:

```arbor
send(
    first,
    second,
);
```

Formatting is deterministic and idempotent. Comments and string contents retain
their meaning. Arbor adopts these formatting conventions within its own grammar;
default line width and remaining layout details are still to be specified.

## 5. Numbers and constant evaluation

### Numeric families

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

### Contextual numeric literals

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

### Constants

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

## 6. Modular arithmetic

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

### Representation and extraction

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

### Arithmetic and overflow guarantees

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

## 7. Strings and secret values

### Delimiters

```arbor
let ordinary = "hello";
let quotation = """She said "hello".""";
let multiline = """
first line
second line
""";
let day = date'2000-01-01';
let message = term'An unrecoverable $bold${red}error$reset occurred.';
```

- Double quotes delimit ordinary strings.
- Triple double quotes support multiline content and unescaped individual double
  quotes.
- Single quotes are reserved for prefixed typed literals; bare `'text'` is invalid.
- Unknown prefixes are compiler errors.
- Ordinary strings cannot contain literal line breaks.
- Triple-quoted strings preserve whitespace and line breaks exactly; indentation
  removal is an explicit operation.
- Ordinary and triple-quoted strings share escape rules, including quote,
  backslash, newline, and Unicode escapes.
- An unescaped triple double quote closes a triple-quoted string.
- The formatter must preserve literal contents exactly.

Mutable strings are supported. Encoding representation, indexing units, Unicode
normalization, grapheme operations, interpolation, and complete escape syntax
remain open. APIs must distinguish bytes, Unicode scalar values, and grapheme
clusters where the distinction affects behavior.

### Secret values

```arbor
let password = secret'hs73!:heirb£38';
let entered = Secret(read_password());
```

The literal constructs `Secret[string]`. A generic `Secret[T]` can protect other
values. There is no implicit conversion to the underlying type; explicit
`.reveal()` obtains an ordinary value without redaction protection. APIs may
accept secrets directly.

Required documentation wording:

> `Secret[T]` guarantees redaction only through standard-library logging. Those
> loggers render secrets as `********`, including secrets nested in structured
> log data. It provides no encryption, third-party logging guarantee, timed
> memory erasure, or erasure after use. Third-party libraries may expose or retain
> the value; malicious libraries remain a risk. Revealed values lose the redaction
> protection. Secret literals remain in source and may appear in compiled artifacts.

The mask has fixed length regardless of the secret. Supported transformations
that operate on underlying secret text preserve the wrapper. Default serialization
should reject secrets and require an explicit redaction or exposure decision.
Those API restrictions do not extend the logging guarantee to debuggers, arbitrary
formatters, third-party code, or memory inspection.

## 8. Dates, times, and timezones

Temporal values are first-class, with distinct meanings and types:

| Literal                                          | Type             | Meaning                             |
| ------------------------------------------------ | ---------------- | ----------------------------------- |
| `date'2000-04-18'`                               | `LocalDate`      | Calendar date                       |
| `time'09:30:00'`                                 | `LocalTime`      | Clock time                          |
| `local'2026-10-11T09:30:00'`                     | `LocalDateTime`  | Local date and time                 |
| `instant'2026-10-11T09:30:00Z'`                  | `Instant`        | Timeline point written in UTC       |
| `offset'2026-10-11T09:30:00+02:00'`              | `OffsetDateTime` | Date/time and fixed UTC offset      |
| `zoned'2026-10-11T09:30:00+02:00[Europe/Paris]'` | `ZonedDateTime`  | Date/time and named geographic zone |
| `tz'Europe/Paris'`                               | `TimeZone`       | Named timezone rules                |
| `duration'PT24H'`                                | `Duration`       | Elapsed time                        |
| `period'P1D'`                                    | `CalendarPeriod` | Calendar adjustment                 |

The bracketed zone is an extension to the ISO-style representation. Zoned literals
include both offset and zone; the compiler checks their agreement. The two offsets
at a daylight-saving overlap identify different occurrences of the local time.
Construction without an explicit offset requires a resolution policy or raises a
declared exception for ambiguous or nonexistent times.

Literal rules are deliberately narrow: Gregorian `YYYY-MM-DD`, `HH:MM:SS` with
optional fractional seconds, uppercase `T`/`Z`, and `±HH:MM` offsets. Invalid dates
and unsupported precision are compile-time errors. No omitted seconds or alternate
date formats in literals. Runtime parsing may offer additional explicit formats.

The initial proposal excludes leap-second literals and documents that timeline
arithmetic does not count leap seconds. Final range and precision limits remain
open. Timezone data is versioned and pinned for reproducible builds and evaluation.

Adding a calendar day can differ from adding 24 hours. Month arithmetic needs an
explicit invalid-date policy. Coordinate-to-timezone lookup uses separate versioned
geographic data; named zones do not require coordinate lookup. Exact lookup APIs
and their v0.1 scope remain open.

## 9. Containers

Four built-in container shapes are selected; names remain provisional:

| Shape             | Purpose                                       | Illustrative spelling |
| ----------------- | --------------------------------------------- | --------------------- |
| Tuple             | Fixed positions; homogeneous or heterogeneous | `(int, string)`       |
| Growable sequence | Homogeneous runtime-sized collection          | `list[int]`           |
| Set               | Unique elements                               | `set[int]`            |
| Mapping           | Unique keys with associated values            | `map[string, int]`    |

There is no separate fixed-size array type: tuples cover the fixed-size shape.
Tuples can have mutable slots or be deeply frozen; the earlier suggestion of
always-immutable tuple slots is superseded.

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

Working container rules carried forward from the design discussion:

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

### Sequences as mappings

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

## 10. Ownership, freezing, and pointers

### Ownership and borrowing

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

### Pointers

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

## 11. Types, generics, and variance

Arbor takes a pragmatic approach to static typing: nominal classes, structural
protocols, generics, complex and recursive types, and enums are required. The
working proposal also includes unions, intersections, tagged enums with payloads,
and an explicit dynamic boundary; their complete rules remain to be specified.

```arbor
type Json = null | bool | int | decimal | string | list[Json] | map[string, Json];
```

This is an illustrative recursive type. The actual JSON number representation
must account for large numbers and remains part of the numeric/data-format work.

### Enums, automatic values, and flags

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

### Unions versus correlated choices

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

### Variance and mutation capabilities

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

## 12. Null, ellipsis, sentinels, and void

### Strong constants

`true`, `false`, `sentinel`, `null`, and `void` are distinct built-in singletons.
They exist for the program lifetime, cannot be garbage collected, and can be
checked by identity. These guarantees describe language-visible identity;
implementations may use immediate representations rather than heap allocations
provided that identity behavior remains the same.

- `true` and `false` are the ordinary Boolean values, spelled in lowercase.
- `sentinel` means a function argument was not passed. There is exactly one
  sentinel across the language, not one per type or parameter.
- `null` is a present None-like value. Every type can explicitly admit it; it can
  be stored and propagated wherever permitted by the type contract.
- `void` represents absence and cannot be stored in arrays/sequences or mappings.
  Other restrictions on storing it as an ordinary payload still apply.

Being an immortal singleton does not make every constant legal in every context.
In particular, sentinel propagation and void storage remain prohibited.

### Special-value and result contracts

These have distinct roles:

- `null` is available for any base type only when explicitly permitted.
- Ellipsis is accepted by the compiler as an argument regardless of base type,
  requires no opt-in annotation, and is truthy.
- The global `sentinel` is admissible as an omission default regardless of a
  parameter's type, but cannot propagate as an ordinary value.
- `void` means no value/result, including an absent collection lookup result, and
  cannot be stored as an ordinary value.
- `NoReturn` means the function raises for every accepted input; it has no normal
  result, including no `void` result.

`null` is a present value equivalent in role to Python's `None`; `void` expresses
absence. A possibly missing lookup has result type `T | void`. If `T` permits
null, the cases remain distinct: a concrete value, present `null`, or absent
`void`. No public `Found`/`Missing` wrapper is required for lookup.

```arbor
let result = records.lookup(key);
if result is void {
    handle_missing();
} else {
    use_present(result);
}
```

The check syntax is provisional. The compiler tracks the no-value alternative and
requires it to be handled or propagated through a compatible result type before
using the result as a definite `T`. The test above removes only `void`, not any
nullable alternative in `T`. This is not a truthiness check: zero, false, an empty
string, and null do not mean a missing entry.

An exclusively `-> void` function returns control normally on successful
completion but has no bindable value. `let x = save()` is invalid when `save`
returns only void. `save()` is an ordinary call statement; any standalone-call
warning is linter-only and may be explicitly suppressed with `noqa`. Writing,
logging, and sleeping require no suppression. Suppression does not affect the
call's execution, as described in section 3.

A local result binding such as `result` above may track the absence alternative for
inspection, narrowing, and return propagation; this does not turn absence into a
storable collection or field value. Narrowing away `void` or supplying a storable
fallback is required before storage. Boxing or widening to a dynamic type must not
evade this rule. Runtime representation and the complete rules for passing no-value
results remain open; `void` is not automatically accepted wherever a value is
required.

The ability to inspect a `T | void` lookup result does not permit binding an
exclusively void call: the former has a possible value to handle, whereas the
latter never supplies one.

`NoReturn` instead specifies an exceptional outcome for every accepted input:

```arbor
fun fail(message: string) -> NoReturn
    throws InvalidArgument
{
    throw InvalidArgument(message);
}
```

There is no `NoReturn` value to store or inspect. An always-raising expression can
occupy a position expecting a value of any type because it never supplies a normal
value; its exception effects still apply. Normal return and fallthrough are
invalid. Nontermination alone does not satisfy this always-raising contract.
Exact compiler verification, including possible divergence during evaluation,
remains open; a general termination proof is not implied by the annotation.

```arbor
fun add(x: null mut int, y: int) -> int { ...; }
fun configure(limit: int = sentinel) {
    prepare_configuration();
    if limit is sentinel {
        ...;
    }
}
```

The first argument to `add` permits null and mutation; `y` permits neither. A
function accepting `mut` arguments cannot be pure. Sentinel handling may be
delayed until later in the function; there is no mandatory entry-time check.
The compiler tracks the potentially omitted parameter until the code checks or
resolves it before an operation requiring an actual typed value. The sentinel
cannot be forwarded to another function, returned, or stored as an ordinary value.
These restrictions belong to parameter handling, while the sentinel's identity
is global. This supersedes the original parameter-specific sentinel proposal.

Standalone `...` marks unfinished implementation and raises `NotImplementedError`
in development/test execution. That use is prohibited in production builds.
Passing, forwarding, and inspecting an ellipsis value is a separate capability.

**Open:** the final interaction between universally admissible ellipsis values,
operations requiring concrete data, and the production ban on unfinished-code
failures. A proposed rule was to raise `NotImplementedError` on concrete use, but
this needs reconciliation with the later production restriction. Ellipsis must
not silently become permission to ship unimplemented bodies.

## 13. Purity and runtime caching

Pure and ordinary functions are distinct. Pure functions cannot accept mutable
arguments, perform observable mutation or I/O, schedule work, read clocks or
randomness, or depend on mutable external state. They can call pure functions and
raise deterministic declared exceptions.

### Diagnostic logging exception

Pure functions may emit diagnostic logs. Logging has no influence on their
returned values or exception outcomes, and log occurrence is not guaranteed.
Cache hits can skip logging. Logger failures must not change the computation's
outcome. Guaranteed audit events do not belong in this exemption.

### Process-local, opportunistic caching

Purity permits caching without a time-based expiry. It does not require retaining
results forever or even caching every call. Entries live within a process, can be
evicted, and can be selected according to cost, reuse, and memory pressure.
Persistent cross-process memoization is not part of the selected model.

Purity and cache eligibility are separate:

- Read-only borrowed input can remain stable during a call yet change afterward.
- Cache identity cannot rely on the address of such an object.
- Eligible keys need stable frozen values, snapshots, or another verified
  value-based mechanism.
- Function implementation identity and relevant context affect cache validity.
- Settings and test patches require isolated cache contexts.
- Mutable results cannot be shared between callers as independently mutable
  objects; a sound policy needs fresh copies or frozen results.

Exact key construction, result handling, eviction policy, and cache-control APIs
remain open. Tests can disable caching and must not observe stale results from
different patches or settings.

## 14. Classes and inheritance

### Access and class forms

Members are public by default. Read and write access can be restricted separately,
including private methods and private setters. Documentation should state:

> Unnecessary privacy is poor practice. Restrict access when it protects a
> concrete invariant or boundary, and document that reason.

Tests receive elevated access to inspect internal coherence without bypassing
memory safety. Production code does not receive test privileges. Singletons and
frozen objects are supported.

Multiple inheritance is mandatory. Mixins may participate in inheritance, cannot
be instantiated on their own, and a class declaration inheriting only mixins
remains a mixin until it has a concrete class ancestor. Exact syntax and the
interaction with an implicit root class remain open.

### C3 and explicit method selection

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

### Substitutability

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

## 15. Decorators, accessors, and settings

### Decorators

Decorators support function and class transformations with compiler-visible
signatures, effects, mutation permissions, and exceptions. Typed compile-time
application is the working direction. A decorator cannot silently invalidate a
purity or exception contract.

Ordinary wrapping decorators apply from the nearest declaration outward.
Declaration markers such as accessors and fixture bindings have dedicated rules;
they are not arbitrary runtime descriptor composition.

`@classmethod` and `@staticmethod` are supported for ordinary methods.

### Getters and setters

Attribute access can invoke getters and assignment can invoke setters; there is no
separate property construct. Instance and class accessors use explicit compiler
designations:

```arbor
@getter
fun api_key() -> Secret[string] { ...; }

@classgetter
fun api_key() -> Secret[string] { ...; }
```

Corresponding setter designations are `@setter` and `@classsetter`. Exactly one
receiver/accessor designation is permitted. Stacking `@classmethod` and `@getter`
is an error in either order. Getter/setter pairs agree on receiver kind and value
type; read and write visibility may differ.

Class accessors receive the class used for access and follow C3. Access through
the class is the selected direction, avoiding ambiguous instance lookup. Other
decorators can wrap an accessor only with support for its signature and effects.

Separate `classproperty` and `staticproperty` decorators are unnecessary. Static
accessors are deferred; a class getter can ignore its receiver.

### Lazy values

The use case is `client.api_key`: fetch once, store privately, and reuse afterward.
Accessor syntax does not hide effects or declared exceptions. Ordinary cache
mutation requires mutable access.

A controlled initialize-once cell was proposed to support synchronized lazy
storage: one initializer at a time, successful values retained, failures retryable,
and recursive initialization detected. Class-level storage must specify whether
it is shared by the hierarchy or belongs to each concrete class.

**Open:** the cell's exact API and whether it can exist in an otherwise frozen
object. Externally sourced lazy initialization must not quietly undermine deep
freezing, hashability, or purity/caching. This requires an explicit internal
mutation contract rather than treating every getter as read-only or pure.

### Settings

`@settings` defines typed configuration with construction, validation, deep
freezing, and shared read-only access. Initialization is explicit, may perform I/O,
and may raise declared exceptions. No partially initialized instance is published.

The compiler uses this mechanism to parse its own manifest:

```arbor
@settings
class Manifest {
    package: Package;
    dependencies: Dependencies;
}
```

The loader parses the supported YAML subset, validates fields and types, applies
declared defaults, rejects unknown fields, and freezes the result. Diagnostics
include file, field path, and source position.

The singleton is per project/build context, not necessarily process-global. A
compiler may handle several projects or manifests concurrently. Tests install
separate frozen instances in isolated scopes. Cache contexts must distinguish
settings instances; process-wide mutable overrides are not the model.

Logging configuration is part of the manifest schema and has usable defaults.
An optional development-only `manifest.yaml.d/` mechanism is proposed in section
23; it is not committed to v0.1. The bootstrap implementation and separation of
schema decoding from contextual storage remain implementation questions.

## 16. Exceptions, validation, and contracts

Arbor prefers exceptions over return codes for exceptional failures, with explicit
contracts for ordinary escaping exceptions. Expected conditions have ordinary
control-flow representations. Iteration does not use a fake `StopIteration` error.

### Fundamental categories

| Type                  | Contract                                                                                   |
| --------------------- | ------------------------------------------------------------------------------------------ |
| `BaseException`       | Abstract root; cannot be thrown or caught directly                                         |
| `Exception`           | Cannot be thrown; catch-only umbrella for ordinary exceptions                              |
| `NotImplementedError` | Undeclared development/test failure; unfinished implementations are rejected in production |
| `RuntimeError`        | Unexpected runtime/platform invariant failure, potentially including memory corruption     |
| `KeyboardInterrupt`   | Generated for Ctrl-C; cannot be explicitly thrown                                          |
| `SystemExit`          | Immediate exit, default status 1                                                           |
| `ContractViolation`   | Undeclared programmer-contract failure, outside the ordinary catch-all                     |

Runtime errors, keyboard interruption, and system exit are outside `Exception`.
Catch-all application handling must not accidentally swallow them. Contract
violations can be caught explicitly, including by tests and application boundaries.
Detailed handling rules for other fundamental categories remain open.

### Throw and warn keywords

`throw` and `warn` are language keywords. Both operate on instantiated objects:

```arbor
throw AccountLockedError();
warn DeprecatedIn71Warning();
```

`throw` raises an exception and transfers control according to the exception
handling and declared-exception rules. `warn` submits a warning to diagnostic
reporting and execution continues normally. Warning classes are distinct from
exception classes; emitting a warning does not itself add an exception to a
function's declared exception contract.

The operand may also be an existing instance, so a diagnostic can be constructed
separately from its use:

```arbor
let notice = DeprecatedIn71Warning("foo_function");
warn notice;
```

Passing a class instead of an instance is a compilation error. Thrown exception
instances remain subject to the restrictions on fundamental categories above:

```arbor
throw InvalidArgument("Expected a positive value");
throw SystemExit();
throw SystemExit(status=0);
throw SystemExit; // Error: this is a type, not an instance.
```

Exception selectors in handlers are types; the instantiation rule applies to
throwing, not to selecting a handler.

### Typed diagnostic classes and construction

Errors and warnings retain reusable classes with typed contextual fields and
message templates. A diagnostic class defines its wording once, and instances
supply its contextual values. Standard logging can accept a diagnostic instance
and use the same message and structured fields; callers do not need to duplicate
message text, diagnostic identity, or default severity. Warning reporting uses
the standard diagnostic/logging facilities, including Secret redaction.

All concrete, user-constructible error and warning classes support bare
initialization. A bare instance must be a valid diagnostic with a renderable
message. Required contextual data therefore needs valid defaults or explicit
absence handling; the exact fallback-template mechanism remains open. This
construction guarantee does not remove restrictions on abstract or runtime-only
fundamental categories.

Diagnostic constructors accommodate standard `log` and `stacklevel` parameters,
using `sentinel` to distinguish omission from an explicit value. `log` controls
logging during construction. `stacklevel` selects the caller location attributed
to a report and does not alter the exception's traceback. The parameters have
consistent meanings across classes; compiler support and their default policies
remain to be specified. The usual sentinel rules still apply: omission must be
resolved during initialization, and sentinel cannot be stored in instance fields.

Supporting a common zero-argument constructor does not require identical
additional parameters across subclasses. Code constructing arbitrary diagnostic
classes with contextual arguments must require an appropriate factory signature.

Detailed reporting policy remains open, including repetition/filtering,
attribution to a later `warn` statement, and duplicate-report handling when an
instance was already logged during construction. Reporting policy must be
reconciled with checked exception contracts and the pure-function diagnostic
exemption. Exact template binding and diagnostic protocol spellings are also open.

### Development and runtime failures

Production builds reject standalone unfinished-body ellipses and explicit throws
of `NotImplementedError` in included production code, even behind a currently
false condition. Tests are excluded from that production-code check. Exact
dependency/build-artifact enforcement remains to be specified.

Actual memory corruption may make exception construction or cleanup unsafe and
require a fail-fast runtime path. Immediate `SystemExit` has no normal cleanup
guarantee. Ctrl-C cancellation and cleanup policy remain open.

### Declared operational exceptions

Ordinary exceptions escaping a function belong in its declared contract. The
compiler checks catching and propagation. Internal effect inference can assist
implementation, while public APIs remain explicit. Precise syntax, hierarchy
coverage, generic effects, and the implicit built-in failure set remain open.

Asking whether an operation is valid is preferred to using exceptions for routine
branching. For concurrent or OS operations, a preliminary check is not atomic;
APIs still need atomic operations and meaningful operational errors. The compiler
may prove or move redundant checks rather than mandating repeated checks in loops.

### Validation without signature pollution

```arbor
validate(x, positive);

pure fun square(x: int) -> int
    ensures(result >= 0)
    throws ArithmeticOverflow
{
    return x * x;
}
```

Syntax is provisional. Contracts express API obligations and promised results.
Squaring promises nonnegative, not strictly positive, because zero is valid.

- Preconditions constrain arguments; postconditions constrain successful results.
- The compiler proves contracts when possible and otherwise checks them at runtime.
- Unproved checks remain enabled in production.
- Failure raises `ContractViolation` without adding it to propagated `throws` lists.
- Callers need not add wrappers or statically prove every obligation.
- A caller precondition failure is distinguished from an implementation failing
  its postcondition.
- Predicates are pure; machine-readable contract expressions are retained in
  compiled metadata where possible.
- Redundant checks can be removed only when proven unnecessary. Opaque predicates
  may still have runtime costs.

Malformed external input is not automatically a programming mistake: a parser or
domain-validation API reports expected invalid input as an ordinary result or
declared exception. It cannot evade its operational contract by relabeling every
bad input a contract violation.

Assertions are test-only. Production validation uses contracts or `validate`.

## 17. Context managers and exception handling

Context managers provide structured resource acquisition and cleanup:

```arbor
with open("report.txt") as file {
    process(file);
}

async with connect(url) as connection {
    await connection.send(message);
}
```

Entry and exit effects contribute to function contracts. Cleanup runs on ordinary
scope exit, including return and exception unwinding. Immediate termination and
an unsafe runtime failure cannot promise cleanup. When cleanup also fails, both
failures must remain available; the exact chaining/aggregation rule is open.

### Flexible try clauses

```arbor
try {
    perform_operation();
} except PermissionError as error {
    recover(error);
} else {
    record_success();
} finally {
    release_resources();
}
```

Any combination of `except`, `else`, and `finally` is allowed; an exception handler
is not mandatory. Handlers are considered in source order. `else` runs only after
normal completion of the try body, not after an exception, return, break, or
continue. Exceptions from `else` are not caught by preceding handlers. `finally`
runs on ordinary exits and unwinding.

### Scoped suppression

```arbor
with ignore(PermissionError) {
    read_file(path);
}
```

A selected exception exits the block and resumes after it. Execution does not
resume at the failed statement. Multiple types may be selected. Unmatched errors
propagate; the compiler removes handled exceptions from the block's escaping set.

### Function-level handling

```arbor
@handle_exceptions(
    [PermissionError, FileNotFound],
    message="Unable to load configuration",
    level=warning,
    return_value=null,
)
fun load_config(path: Path) -> null Config
    throws PermissionError, FileNotFound, InvalidConfig
{
    ...;
}
```

The exposed callable in this example declares only `InvalidConfig`. A matched
failure is logged through standard-library logging and yields the fallback.

- Fallback values match the return type.
- Mutable fallbacks must be created per invocation rather than accidentally shared;
  the exact factory syntax is open.
- Other errors propagate.
- Secret redaction is preserved through the standard logger.
- Async handling covers task execution, not merely creation of its promise.
- Wrapper order is significant: handling outside retry can occur after exhaustion;
  handling inside retry can prevent retry from seeing the failure.

The selected name is `@handle_exceptions`, including for a single exception.
Ordinary exception selection does not include runtime/control events or contract
violations outside the `Exception` umbrella.

## 18. Tasks and async

Cheap concurrency is a core goal. Lightweight scheduled tasks avoid requiring an
OS thread per concurrent operation. Ownership moves and shared frozen data are
the safe basis for crossing task boundaries; concurrent mutable access needs an
explicit safe mechanism.

```arbor
async fun upload(file: File) -> Receipt
    throws NetworkError
{
    ...;
}

upload.fire(file);               // Start a detached task.
let receipt = upload.now(file);  // Start and wait.
let pending = upload(file);      // Start and return a promise.
let result = await pending;
```

Calling an async function starts it immediately. `.fire()`, `.now()`, and `await`
are allowed from synchronous functions. Waiting suspends the current lightweight
task rather than occupying its worker. Foreign synchronous entry and blocking
foreign calls need explicit runtime bridges.

- `.fire()` has no completion guarantee at shutdown and reports failures through
  a configured detached-task error handler.
- `.now()` and `await` expose execution exceptions.
- Promise creation may itself have immediate exceptions where relevant.
- Task groups provide ordinary structured concurrency, cancellation, and deadlines.
- Cancelling a waiter cancels that wait, not automatically the underlying task.
- Task cancellation is a separate explicit operation.
- Discarding a promise does not mean detaching it; `.fire()` expresses that intent.

### Repeated await and ownership

The same promise may be awaited repeatedly or concurrently. Work executes once
and its completion state remains available until released. This is not an
evictable pure-function cache and must never rerun the operation on a later await.

```arbor
let pending = upload(file);
let first = await pending;
let second = await pending;
```

Frozen results can be shared, copyable values can be returned by value, and
mutable/uniquely owned results cannot be handed out as independent owners. The
selected direction is read-only observation of the stored result plus an explicit
one-time ownership transfer:

```arbor
let view = await pending;
// End view's borrow before taking ownership.
let owned = pending.take();
```

`take()` is provisional syntax. It requires exclusive access, no outstanding
views, and no other promise handles, and waits if necessary. Repeated failure
observations retain the original failure outcome. Exception-object ownership and
exact result-return conventions remain to be specified.

A promise returned as another promise's result requires another await; there is
no implicit flattening. Runtime scheduling, cancellation delivery, cleanup,
blocking bridges, and detailed promise types remain implementation decisions.

## 19. Conditional expressions, indexing, and loops

### Ternary expressions

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

### Switch expressions

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

### Indexing

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

### Loop forms

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

### Break depth

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

### Empty-iteration else

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

## 20. Testing

Testing is an official language/toolchain capability, with tests under `tests/`.
The compiler enforces the production/test dependency boundary. Tests receive
elevated member access to inspect private state, but retain memory-safety rules.
Assertions are test-only.

### Typed, parameterized fixtures

Fixtures are factories with managed lifetimes and ordinary typed arguments:

```arbor
@fixture
fun users(are_superusers: bool = false, count: int = 3) -> list[User] {
    ...;
}

@fixture
fun bank_statements(count: int) -> list[BankStatement] {
    ...;
}

@test
@use(users, into=user_list, are_superusers=true)
@use(bank_statements, count=5)
fun latest_user(
    bank_statements: list[BankStatement],
    user_list: list[User],
) -> void {
    ...;
}
```

Fixture results are explicitly bound, not guessed from types or decorator order:

- Without `into`, the target parameter has the fixture's name.
- `into` identifies a checked parameter binding.
- Other arguments belong to the fixture; ordinary defaults apply.
- Missing, duplicate, unknown, or incompatible bindings are compilation errors.
- Each declaration acquires a separate instance unless sharing is explicit.
- Multiple parameterized instances of one fixture are allowed.
- Fixture dependencies determine setup order, and cycles are errors.
- Cleanup reverses acquisition order and handles partial setup failures.
- Async setup and teardown are supported.
- Default lifetime is the individual test/case.

`@use` declares runner setup without changing the function's ordinary typed
signature. Body-level acquisition also exists in the design, provisionally:

```arbor
let sample = fixture users(count=5);
```

Cleanup registration and the exact fixture teardown API remain to be specified.
Tests that complete normally return `void`. `NoReturn` promises that every
accepted invocation raises. Test invocations follow ordinary call semantics;
optimization cannot omit their required behavior merely because they return void.

### Parametrization and mocks

```arbor
@parametrize(
    (input, expected),
    [(0, 0), (2, 4), (-3, 9)],
)
@test
fun square_cases(input: int, expected: int) {
    assert square(input) == expected;
}
```

Each parameter combination is independently runnable, with its own fixture,
settings, patch, and cache scopes.

Mocks and monkey patches are scoped and can inspect call counts and arguments:

```arbor
let mocked = mock(service.send);
service.send(message="hello");
assert mocked.call_count == 1;
assert mocked.calls[0].args == {"message": "hello"};
```

Calls use canonical argument binding, so equivalent positional/keyword calls
compare consistently. Mutable argument recording needs an explicit snapshot
policy. Patches isolate caches transitively so cached callers cannot hide patched
implementations. Tests can disable caching and use deterministic clocks/scheduler
controls; exact APIs remain open.

### Parallelism and coverage

Parallel execution is the default. A test should behave the same alone, after
another test, or concurrently with unrelated tests. The runner isolates settings,
fixtures, patches, and caches. External-resource fixtures provision their own
database instance/schema, temporary directories, ports, or equivalent resources.

Tests establish the state they require. Their correctness must not depend on the
previous test's cleanup succeeding. Transactions alone are insufficient for tests
that commit, change schemas, or destroy databases. Explicit shared-resource locks
or serial execution remain visible escape hatches; arbitrary external effects
cannot be automatically isolated.

Coverage is built into compilation and test execution, with line and branch
coverage merged across workers and source mapping for generics and async code.
Production builds contain no instrumentation unless requested. How to isolate
compiled patches and instrument cached execution remains an implementation task.

## 21. Packages and dependency groups

### Dependency purpose

The five standard groups are `normal`, `test`, `dev`, `doc`, and `lint`:

```yaml
dependencies:
    normal:
        dates:
            package: "@chronos/datetime"
            version: "1.4.2"
    test:
        assertions:
            package: "@stdlib/assert"
            version: "2.0.1"
    dev:
        watcher:
            package: "@tools/watch"
            version: "1.1.0"
    doc:
        documentation:
            package: "@tools/docs"
            version: "3.0.0"
    lint:
        rules:
            package: "@quality/core"
            version: "1.2.0"
```

Manifest shape is provisional; the groups themselves are selected. Normal
dependencies are available to production source and relevant project tooling.
Other groups are activated for their own purpose and do not all become visible
automatically. Consumers of a published library bring in its normal dependencies,
not its test/dev/doc/lint environment.

### Identity, namespace, and alias

Published names require a scope: `@scope/package`. The scope is a registry-managed
ownership boundary. Full identity includes `(registry, scope, package, version)`.
Registries are explicit; no silent fallback to another registry is permitted.

Manifest dependency keys are local import aliases:

```arbor
import dates;
```

Scope and package components follow strict rules:

- Lowercase ASCII letters, digits, and hyphens only.
- Start with a letter and end with a letter or digit.
- No consecutive hyphens.
- At most 63 characters per component.
- No Unicode, underscores, dots, whitespace, or percent encoding.
- Invalid spellings are rejected instead of normalized.
- Official scopes are reserved by registry policy.

Import aliases follow Arbor identifier rules and cannot be keywords.

Different transitive dependencies may use different versions of the same package.
This is mandatory. Versions remain isolated, and nominal types from distinct
package versions are not silently interchangeable. Cross-version interoperability
requires an explicit mechanism.

## 22. Package versions and immutability

### Canonical version identity

Versions retain a three-part numeric base, with epochs, prereleases, post releases,
and local labels. The working grammar is:

```text
[positive-epoch!]MAJOR.MINOR.PATCH[-alpha.N|-beta.N|-rc.N][.postN][+local]
```

This extends SemVer; it is not strictly SemVer because post releases and epochs
have additional semantics.

```text
1.4.2
2026.9.6
26.3.0
1.2.3-alpha.5
1.2.3.post1
1.2.3+vendor.1
1.2.3.post1+vendor.1
1!1.0.0
```

Rules:

- Exactly three numeric base components are required.
- Numeric version components are unpadded.
- Epoch zero is implicit; explicit `0!` is invalid.
- Explicit epochs are positive, unpadded integers.
- Neither `s:` nor `c:` is accepted.
- No `v` prefix, `latest`, wildcard, or range syntax in v0.1 dependencies.
- Prerelease stages are `alpha`, `beta`, and `rc`, each with an unpadded number.
- `.postN` is allowed; `.pre` and `.dev` releases are rejected.
- Local versions use Python-style `+label`, not a literal `.local` suffix.
- Invalid spellings are rejected, not silently canonicalized.

The exact local-label alphabet/canonical spelling rules, component magnitude limits,
and remaining suffix edge cases need finalization before implementing a parser.

### Ordering and compatibility

Compare epochs numerically first, then the three numeric components. For the same
base, `alpha < beta < rc < final`, with numeric stage ordering. A post release
follows the corresponding release and post numbers compare numerically.

```text
1.2.3-rc.1 < 1.2.3 < 1.2.3.post1 < 1.2.4 < 1!0.1.0
```

Local labels distinguish exact artifact identities without changing release
precedence. Different labels are not silently substituted. Any display tie-break
order must not invent a compatibility or precedence claim.

Epoch changes establish ordering, not compatibility. Update tools require explicit
requests to cross epochs. Prereleases require explicit selection. Update commands
can discover candidates and write exact manifest changes for review.

### Descriptive versioning promises

Calendar conventions and vendor release schemes differ. The manifest can describe
them without changing version parsing or ordering:

```yaml
package:
    name: "@vendor/platform-bindings"
    version: "26.3.0"
    versioning: calver
    spec: "YY.M.MICRO"
    description: "YY identifies a platform release year beginning around September."
```

`versioning` and `spec` are publisher promises to users. Their meaning is not
enforced by the package manager. SemVer is the default interpretation; neither
field is required. There is no calendar-date validation for package versions, and
no automatic compatibility inference from a calendar label.

Changing descriptive schemes does not itself require an epoch change. Epochs are
available when restarting numbering must still order new releases after old ones.
This supersedes the earlier proposal of one enforced scheme per epoch. Published
metadata remains immutable, so a new manifest cannot rewrite an old release's
description. Version labels need not equal publication dates.

### Immutable publication and locking

Once published, a version permanently identifies the same artifact bytes and
dependency metadata:

- Different content cannot replace an existing version.
- Corrections require a new version.
- Deletion cannot make a version name reusable.
- Ownership transfers do not permit replacing previous releases.
- Yank status and advisory reasons may discourage selection without rewriting
  the immutable artifact.
- Security/legal removal may make content unavailable, but cannot permit reuse or
  silent substitution; unavailable required content produces an explicit failure.

A lockfile records exact transitive versions, registry identities, and
cryptographic artifact hashes. Builds verify hashes. The lockfile filename,
schema, local source dependencies, and publishing protocol remain open.

## 23. Standard library and data formats

Batteries are included: collections, numbers, modular arithmetic, Unicode strings,
time, concurrency, logging, configuration, and testing all have standard support.
Precise module names and the full v0.1 inventory remain to be planned.

JSON, TOML, and YAML are supported data formats. Full YAML is not required in
v0.1, but `manifest.yaml` must work through a documented supported subset.
Unsupported YAML constructs are rejected explicitly. The exact subset and
round-trip/serialization rules remain open.

The formatter, linter, documentation generator, and LSP are official tools, with
machine-readable diagnostics and API information as a design goal. Formatting
must not change string contents or program semantics.

### Output and diagnostics: write and log

The standard output API is named `write`, replacing the earlier `print` spelling.
`write("foo")` writes to stdout by default. It also supports explicit stderr and
file destinations. Diagnostic output uses `log`, which is available without
requiring users to create or configure a logger first. Both APIs should be readily
available through the normal standard environment.

Illustrative API spellings:

```arbor
write("foo");
write("Something went wrong", to=stderr);

with open("report.txt", mode="write") as report {
    write("Report contents", to=report);
}

log("Reached this point");
log.debug("Parsed statement", statement_id=statement.id);
log.error("Could not connect");
```

`log` also accepts typed diagnostic instances, using their message, structured
fields, and default severity as described in section 16. The `warn` keyword is the
language operation for emitting a warning instance.

Destination keyword spelling, writer protocols, newline behavior, encoding,
buffering, and return conventions are still to be specified. Explicit file handles
make file-opening and ownership separate from writing. Writes have ordinary I/O
effects and declared operational failures; they are not exempt from purity.

Logging configuration lives in `manifest.yaml` and is optional because defaults
are provided. Proposed defaults are stderr output, an info threshold, readable
text formatting, and `log(message)` as shorthand for an info-level event. These
defaults make the simplest diagnostic call visible without contaminating a
program's stdout. Exact levels, shorthand, formatting, and schema remain
provisional. An illustrative configuration is:

```yaml
logging:
    level: info
    destination: stderr
    format: text
```

The application controls logging configuration; dependencies use the provided
logging system without replacing the application's configuration on import.
Standard-library logging keeps the Secret redaction guarantee and the previously
defined pure-function diagnostic exemption. Renaming output to `write` does not
extend that logging guarantee to arbitrary output APIs. `write`, `log`, and
`sleep` remain automatically exempt from standalone-call linting.

### Terminal styled text

The selected syntax is a `term'...'` typed literal containing named dollar
controls. This supersedes the earlier `{styles:content}` annotations and their
brace-escaping proposal:

```arbor
write(term'this is an unrecoverable $bold${red}error$reset, do something');
```

The literal describes styled text. ANSI escapes are generated by the terminal
renderer rather than becoming the authored representation. The exact value type
name and programmatic style-construction API remain provisional.

#### Named controls and boundaries

| Control          | Meaning                                          |
| ---------------- | ------------------------------------------------ |
| `$bold`          | Enable bold in the current style                 |
| `$blue`, `$red`  | Change the current foreground color              |
| `$reset`         | Restore the base style for the rendered value    |
| `$push`          | Save the complete current style on a local stack |
| `$pop`           | Restore and remove the most recently saved style |
| `$company_error` | Apply a declared custom style preset             |

`$reset` is the canonical reset spelling; the earlier `$esc` spelling is not an
additional alias. Attribute changes preserve other current attributes: changing
foreground does not clear bold, for example. Reset changes the current attributes
but does not discard the saved-style stack.

An unbraced control consumes a complete identifier. `$blueword` names one control;
it is never guessed to mean `$blue` followed by `word`. A braced spelling explicitly
ends the name and works for both built-ins and custom presets:

```arbor
term'${blue}word';
term'An unrecoverable ${company_error}error$reset occurred.';
```

Braces outside a `${name}` control are ordinary text. Whitespace outside controls
is preserved exactly; spaces used around controls are not silently removed. A
malformed control, dangling unescaped dollar, or unknown name is a compilation
error. `${name}` denotes a style control, not variable interpolation.

#### Dollar escaping

A maximal consecutive run of N dollar signs, where N is at least two, emits N-1
literal dollar signs and performs no control expansion for that run. Emitted
characters are not reparsed. This is a run rule, not repeated pairwise escaping:

| Source text inside a term literal | Literal output    |
| --------------------------------- | ----------------- |
| `$$`                              | `$`               |
| `$$company_error`                 | `$company_error`  |
| `$$$company_error`                | `$$company_error` |
| `$$${blue}`                       | `$${blue}`        |

A later independently occurring single dollar can still introduce a control.

#### Style scopes and custom presets

Nested formatting uses explicit push/pop scopes:

```arbor
term'$bold Heading: $push${company_error}failure$pop continues in bold.$reset';
```

If `company_error` applies bold red, `failure` is bold red. The following text
returns to the saved bold style and foreground. Push/pop may nest. The compiler
checks for stack underflow and unmatched pushes within each literal; scopes do
not cross literal boundaries.

Custom names resolve to declared formatting presets, such as bold plus red for
`company_error`. Presets apply their specified attributes and retain other current
attributes. They cannot replace built-in control definitions or inject arbitrary
source/control text. Unknown presets are compilation errors. The declaration or
configuration API and the full attribute vocabulary remain to be specified.

At the end of rendering the whole value, the renderer restores its incoming style
so formatting does not leak into subsequent output. This restoration is separate
from compile-time validation of explicit push/pop balance.

Rendering uses terminal capabilities, supports auto/always/never styling, and
respects `NO_COLOR` in automatic mode. Redirected output and ordinary file output
default to plain text. Plain-text rendering omits controls while preserving content
and applying the same dollar-unescaping rule. Tests can inspect content and styles
without comparing raw ANSI sequences. Exact renderer option APIs remain open.

Future value interpolation must keep inserted ordinary text separate from authored
controls; inserted strings are not reparsed as styling. Value interpolation and
multiline syntax, disabling individual inherited attributes, additional color
formats, and the programmatic construction API remain open. Styling does not
broaden the standard-library logging guarantee for Secret values.

### Proposed development-only manifest fragments

`manifest.yaml.d/` is a proposed location for local development overrides, initially
limited to logging configuration. This capability may be deferred beyond v0.1;
the base manifest's logging configuration and usable defaults are independent of it.

```text
manifest.yaml
manifest.yaml.d/
    10-logging.yaml
```

Proposed rules:

- Only explicit development mode reads and applies these fragments. Production
  builds do not read, validate, embed, or apply them, even when the directory exists.
- Allowed override fields are explicitly restricted by schema, initially to
  logging. Other sections, such as dependencies or compiler settings, are rejected
  when processing development fragments.
- Apply validated overrides before freezing the effective settings object.
- Define deterministic precedence and expose which files supplied effective values.
  Base manifest followed by lexically ordered fragments is a candidate order.
- Applied configuration participates in relevant build fingerprints when it affects
  generated artifacts. Ignored production fragments do not affect those artifacts.

The final filename selection rules, merge semantics, dev/test mode relationship,
and initialization timing remain open. This directory is not a general mechanism
for arbitrary configuration overlays.

## 24. Open decisions and v0.1 boundaries

The design deliberately leaves the following work for the semantic specification
and implementation plan. An earlier suggestion is not a finalized rule merely
because an example used it.

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

Known v0.1 limits and priorities:

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
