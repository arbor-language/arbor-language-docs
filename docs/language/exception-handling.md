# Handling failures and resources

Handlers recover from selected failures. Context managers and finally blocks release
resources during ordinary scope exit and exception unwinding.

!!! note "Design documentation"
This chapter describes the language design. Examples are illustrative; implementation
availability and unresolved rules are explained in [Design status](../about/status.md).

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

## Flexible try clauses

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

## Scoped suppression

```arbor
with ignore(PermissionError) {
    read_file(path);
}
```

A selected exception exits the block and resumes after it. Execution does not
resume at the failed statement. Multiple types may be selected. Unmatched errors
propagate; the compiler removes handled exceptions from the block's escaping set.

## Function-level handling

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

## Related reading

- [Exceptions and contracts](contracts.md)
- [Tasks and async](concurrency.md)
