# Testing

The official test runner combines typed fixtures, parameterized cases, scoped mocks,
parallel execution, and coverage. The compiler keeps test privileges out of production
code.

!!! note "Design documentation"
This chapter describes the language design. Examples are illustrative; implementation
availability and unresolved rules are explained in [Design status](../about/status.md).

## Keeping tests independent

Each test case acquires its required state and receives its own default fixture
lifetime. Setup must work when the case runs alone or alongside unrelated cases.
A database fixture therefore needs a resource-isolation strategy appropriate to
what the test does, rather than relying on a preceding test to leave it usable.

The examples below describe runner contracts. Fixture teardown, resource
provisioning, and scheduler-control APIs still need final spellings.

Testing is an official language/toolchain capability, with tests under `tests/`.
The compiler enforces the production/test dependency boundary. Tests receive
elevated member access to inspect private state, but retain memory-safety rules.
Assertions are test-only.

## Typed, parameterized fixtures

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

## Parametrization and mocks

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

## Parallelism and coverage

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

## Related reading

- [Projects and official tools](projects.md)
- [Purity and caching](../language/purity.md)
- [Packages and dependencies](packages.md)
