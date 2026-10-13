# Versions and publication

A published version names immutable content. Exact pins, registry identities, and
artifact hashes make dependency selection and builds inspectable.

!!! note "Design documentation"
This chapter describes the language design. Examples are illustrative; implementation
availability and unresolved rules are explained in [Design status](../about/status.md).

## Canonical version identity

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

## Ordering and compatibility

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

## Descriptive versioning promises

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
Published
metadata remains immutable, so a new manifest cannot rewrite an old release's
description. Version labels need not equal publication dates.

## Immutable publication and locking

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

## Related reading

- [Packages and dependencies](packages.md)
- [Compilation and toolchains](compilation.md)
