# gRPC

go/chc-grpc

## gRPC Protos

### Public GitHub Repository

All GCP protos used by gRPC are published in the following GitHub repository:
https://github.com/googleapis/googleapis/tree/master/google

This is mirrored internally in the following location:
http://cs/google3/third_party/googleapis/

The guidance from the Cloud DPE team is to only publish the Beta and GA protos
and client libraries externally. The EAP and Alpha releases can be shared
privately with customers.

See
[these instructions](https://g3doc.corp.google.com/company/teams/veneer/user/library-publishing.md#cloud-apis-ga-or-beta-releases-only)
for more information.

### How to use gRPC externally?

TODO(b/128996096): Add a small guide here to using gRPC outside Google.

### How to use gRPC internally?

Use `grpc.Dial` to create a gRPC connection to the service. Pass the
`grpc.DialOption` for TLS and Credentials:

```go
var opts []grpc.DialOption

// TLS
opts = append(opts, grpc.WithTransportCredentials(credentials.NewClientTLSFromCert(nil, "")))

// Default Credentials.
opts = append(opts, grpc.WithTransportCredentials(credentials.NewClientTLSFromCert(nil, "")))
creds, err := oauth.NewApplicationDefault(ctx)
if err != nil {
  log.Exitf("oauth.NewApplicationDefault() failed: %v", err)
}
opts = append(opts, grpc.WithPerRPCCredentials(creds))

conn, cleanup := grpc.Dial(addr, opts...)
defer cleanup()
```

To use service account credentials replace the default credentials option with:

```go
// Service Account
creds, err := oauth.NewServiceAccountFromFile(serviceAccountKeyFile, scope)
if err != nil {
  log.Exitf("oauth.NewServiceAccountFromFile() failed: %v", err)
}
opts = append(opts, grpc.WithPerRPCCredentials(creds))
```

Once you have a gRPC connection you can create clients for the service you want
to call using `fgrpc.NewFooServiceClient`. Note that you need both the gRPC go
and go proto libraries.

For a working sample of using gRPC for calling Healthcare API see:
google3/cloud/healthcare/tools/sample/

The sample code only uses third_party packages and should be easy to modify to
use outside google3 and externally.

### Updating the Public Protos

(Based on the instructions found
[here](http://go/actools-user-guide#introduction))

#### One-time setup

1.  Check if you have Docker set up locally:

    ```shell
    which docker
    ```

1.  If not available, follow these
    [set up instructions](http://go/installdocker).

1.  Make sure you also do the
    [Sudoless Docker](http://go/installdocker#sudoless-docker) step as this is
    needed by the artman tool.

#### Run the artman tool

1.  Set up an alias for the artman tool:

    ```shell
    alias g3artman="/google/data/ro/teams/oneplatform/g3artman"
    ```

1.  Run the artman tool against the `api_publish` target in the
    [google/cloud/healthcare/BUILD](http://cs/google3/google/cloud/healthcare/BUILD)
    file:

    ```shell
    # Change the BUILD target as needed.
    g3artman bootstrap --output_dir third_party/googleapis \
        google/cloud/healthcare:healthcare-v1beta1_public_proto
    ```

1.  This will create a CL and update the protos in the
    [`third_party/googleapis/healthcare`](http://cs/google3/third_party/googleapis/healthcare)
    directory.

1.  If you want to make these protos public, send this CL for review to
    `chc-reviews`.

    If you intend to distribute them privately to a customer, just send them a
    copy of the generated proto file.

## GAPIC client libaries

The GAPIC client libraries (also known as veneer) intend to replace the
[REST-based apiary clients](http://go/chc-client-libraries). The GAPIC client
libraries are based on gRPC and follow a similar process to generating the
protos above.

TODO(b/128996096): Document GAPIC client process.
