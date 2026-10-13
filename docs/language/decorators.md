# Decorators, accessors, and settings

Decorators preserve compiler-visible contracts. Accessors attach behavior to attribute
access, and settings provide validated, frozen configuration within a context.

!!! note "Design documentation"
This chapter describes the language design. Examples are illustrative; implementation
availability and unresolved rules are explained in [Design status](../about/status.md).

## Decorators

Decorators support function and class transformations with compiler-visible
signatures, effects, mutation permissions, and exceptions. Typed compile-time
application is the working direction. A decorator cannot silently invalidate a
purity or exception contract.

Ordinary wrapping decorators apply from the nearest declaration outward.
Declaration markers such as accessors and fixture bindings have dedicated rules;
they are not arbitrary runtime descriptor composition.

`@classmethod` and `@staticmethod` are supported for ordinary methods.

## Getters and setters

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

## Lazy values

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

## Settings

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
See [Output and diagnostics](../library/index.md#output-and-diagnostics-write-and-log)
for `write`, `log`, and the proposed defaults. The bootstrap implementation and
separation of schema decoding from contextual storage remain open.

## Proposed development-only manifest fragments

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

## Related reading

- [Classes and inheritance](classes.md)
- [Purity and caching](purity.md)
- [Testing](../tooling/testing.md)
