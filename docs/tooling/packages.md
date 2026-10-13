# Packages and dependencies

Dependencies have an explicit purpose, registry identity, and local import alias.
Published libraries expose their normal dependencies without bringing their development
environment to consumers.

!!! note "Design documentation"
This chapter describes the language design. Examples are illustrative; implementation
availability and unresolved rules are explained in [Design status](../about/status.md).

## Dependency visibility

| Group    | Intended use                                    |
| -------- | ----------------------------------------------- |
| `normal` | Production imports and relevant project tooling |
| `test`   | Test code and the test runner                   |
| `dev`    | Development tooling                             |
| `doc`    | Documentation tooling                           |
| `lint`   | Lint tooling                                    |

Compiling a project for tests does not grant production source access to the
`test` group. Published-library consumers acquire its normal dependency graph;
the other groups describe the publisher's working environment.

## Dependency purpose

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

## Identity, namespace, and alias

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

## Related reading

- [Projects and official tools](projects.md)
- [Versions and publication](versions.md)
