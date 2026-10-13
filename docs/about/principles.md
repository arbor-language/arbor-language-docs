# Principles

Arbor combines explicit boundaries with defaults that help prevent accidental mistakes.
These principles explain the choices throughout the language.

!!! note "Design documentation"
This chapter describes the language design. Examples are illustrative; implementation
availability and unresolved rules are explained in [Design status](status.md).

- **Language:** Arbor.
- **Source extension:** `.arb`.
- **Unified command:** `arbor`.
- **Project manifest:** `manifest.yaml`.
- **Visual identity:** a luminous tree whose roots become circuit traces.

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

## LLM-friendly language and tooling

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

## Related reading

- [Projects and official tools](../tooling/projects.md)
- [Design status and open decisions](status.md)
