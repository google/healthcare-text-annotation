This page: [go/hcls-errors](http://go/hcls-errors)

# Error handling

[TOC]

## Official guidelines

*   https://golang.org/ref/spec#Errors
*   https://golang.org/doc/effective_go.html#errors
*   https://blog.golang.org/errors-are-values

## Top-level error handling

For error handling at the top-level service code that handles the RPC requests,
use public error conversion functions in the
[puberr](https://cs.corp.google.com/piper///depot/google3/cloud/healthcare/util/public_errors.go)
package to annotate the underlying error with a public message. The internal
error details will still be encapsulated within the puberr but visible to
internal-only users.

```go
  puberr.ToPublicErrorf(err, status.PermissionDenied, "my public message")
```

<section class="zippy">

### Advanced Case: Specify The Public Error Details

Note: this is only for when you want to have your own values set in the public error code, msg,
  or detail. In most cases, puberr.ToPublicErrorf would be enough for you.

Following the
[One Platform guidance](http://go/api-errors#specifying-public-error-message),
we can specify the a public status inside an internal status. ESF extracts the
public error and return to clients.

More concretely, in golang:

1.  Create a `Status` instance `Foo` of the `statuspb.Status` type (in
    google3/google/rpc/status.proto). If needed, populate the `details` field
    with a list of protobuf.Any values. All contents of this instance will be
    visible to external request sender.

1.  Create a instance `Bar` of `Status` type in google3/util/task/go/status.go.
    Set `Foo` as the ErrorDetails extension in this status's message set.

1.  Return `Bar.Err()`

See google3/cloud/healthcare/aaa/errors.go;l=29 for an example.

</section>

## Lower-level error handling (ie. helpers, utils)

Use utility functions in the `puberr` package to handle errors from components
deeper in the system (e.g., `puberr.HandleAAAError`). These also take care of
incrementing error metrics consistently.

Defer conversion to public errors (using `puberr.ToPublicErrorf`) to the
top-level function that handles the RPC method (or a utility function used
directly from this function).

### Avoid premature detyping

Don't
[clobber error messages](/net/goa/g3doc/guides/code_review_comments.md#error-detyping)
from deeper in the system. Instead, annotate them or introduce a custom error
type before passing them through to the top.

<pre class="bad">
if err != nil {
  // This hides the details in err.
  return fmt.Errorf(status.NotFound, "image not found")
}
</pre>

<pre class="good">
if err != nil {
  return status.Annotatef(err, status.NotFound, "image not found")
}
</pre>

### Custom errors

If you need to use custom errors, such as for tracking an error's
[metrics](#metrics), define a canonical error or a message set extension on a
status error. Define an `IsMyError` function to check the error type.

### Don't use `IsUnderlyingError`

Don't propagate errors from underlying packages with a small `IsUnderlyingError`
wrapper method. These errors may have different meanings between the underlying
package and the calling package. For example, an `AlreadyExists` error from the
Spanner library may actually be an `Internal` error in our storage layer in the
case that some method creates some internal state that already exists.

<pre class="bad">
resource.go:
  return spanner.Insert()

errors.go:

  func IsAlreadyExists(err error) bool {
    return spanner.IsAlreadyExists(err)
  }
</pre>

<pre class="good">
if err := spanner.Insert(someInternalState); err != nil {
  return status.Annotatef(err, "inserting someInternalState %v", someInternalState)
}

if err := spanner.Insert(hl7Message); err != nil {
  if spanner.AlreadyExists() {
    return alreadyExistsError();
  }
  return status.Annotate(err, "insert hl7Message")
}
</pre>

### Metrics

*   Don't increment error metrics from within `TransactionDo`. It's easy to
    accidentally double-increment the metric in case of a transaction
    rollback/retry. Instead,

    -   to log an LRO succeeded, pass the metric logic using
        [`AppendWriteTxnCommittedCleanupFunc`](http://cs//symbol:AppendWriteTxnCommittedCleanupFunc)
        into `DatasetClient` (or `TransactionClient`), which guarantees only
        runs after the spanner transaction is committed. See cl/293676944 for an
        example.

    -   to log and LRO failed, return a custom error and increment the error
        metrics once the `TransactionDo` call fails.
