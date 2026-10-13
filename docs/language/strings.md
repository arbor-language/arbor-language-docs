# Strings and secrets

Strings support Unicode text and explicit delimiters. Secret wrappers help keep
sensitive values out of standard-library logs, with a deliberately limited guarantee.

!!! note "Design documentation"
This chapter describes the language design. Examples are illustrative; implementation
availability and unresolved rules are explained in [Design status](../about/status.md).

## Delimiters

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

The `term` prefix creates styled text with named dollar controls. See
[Terminal styled text](../library/index.md#terminal-styled-text) for control names,
dollar escaping, style scopes, and rendering rules.

## Secret values

```arbor
let password = secret'hs73!:heirb£38';
let entered = Secret(read_password());
```

The literal constructs `Secret[string]`. A generic `Secret[T]` can protect other
values. There is no implicit conversion to the underlying type; explicit
`.reveal()` obtains an ordinary value without redaction protection. APIs may
accept secrets directly.

### Redaction guarantee

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

### Revealing a value

```arbor
let bare = 'example-password'; // Error: bare single-quoted strings are invalid.
let password = secret'example-password';
let ordinary = password.reveal();
```

The first line deliberately demonstrates a rejected spelling. In the remaining
lines, `password` retains its wrapper and `ordinary` is an ordinary string. The
revealed string no longer receives secret-specific redaction. Prefer an API that
accepts the wrapper directly when one is available.

## Related reading

- [Dates, times, and timezones](time.md)
- [Standard library and data formats](../library/index.md)
