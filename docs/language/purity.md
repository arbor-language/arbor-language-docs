# Purity and caching

A pure function has a result or declared failure determined by its inputs. That
guarantee supports compile-time evaluation and carefully controlled runtime caching.

!!! note "Design documentation"
This chapter describes the language design. Examples are illustrative; implementation
availability and unresolved rules are explained in [Design status](../about/status.md).

Pure and ordinary functions are distinct. Pure functions cannot accept mutable
arguments, perform observable mutation or I/O, schedule work, read clocks or
randomness, or depend on mutable external state. They can call pure functions and
raise deterministic declared exceptions.

## Diagnostic logging exception

Pure functions may emit diagnostic logs. Logging has no influence on their
returned values or exception outcomes, and log occurrence is not guaranteed.
Cache hits can skip logging. Logger failures must not change the computation's
outcome. Guaranteed audit events do not belong in this exemption.

## Process-local, opportunistic caching

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

## Related reading

- [Numbers and constants](numbers.md)
- [Ownership, borrowing, and freezing](ownership.md)
- [Testing](../tooling/testing.md)
