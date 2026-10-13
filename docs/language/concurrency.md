# Tasks and async

Async calls start lightweight tasks immediately and return promises. Ownership transfer
and shared frozen values provide the basis for safe communication between tasks.

!!! note "Design documentation"
This chapter describes the language design. Examples are illustrative; implementation
availability and unresolved rules are explained in [Design status](../about/status.md).

## Choosing how to observe completion

| Expression          | Behavior                             |
| ------------------- | ------------------------------------ |
| `upload(file)`      | Starts work and returns a promise    |
| `await pending`     | Observes the promise's completion    |
| `upload.now(file)`  | Starts work and waits for completion |
| `upload.fire(file)` | Explicitly detaches work             |

Awaiting twice observes the same operation, including its original failure.
It does not create a retry. Retrying an operation requires starting another call.

## Starting and waiting

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

## Repeated await and ownership

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

## Related reading

- [Ownership, borrowing, and freezing](ownership.md)
- [Handling failures and resources](exception-handling.md)
- [Conditionals, indexing, and loops](control-flow.md)
