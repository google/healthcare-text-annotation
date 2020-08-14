# Client Libraries

go/chc-client-libraries

We currently support 4 key programming languages in our client libraries (this
is a requirement for launching to Beta). The 4 languages are Java, Go, Python
and NodeJS.

## Types of Client libraries

There are two common types of client libraries.

*   Google Cloud Client Libraries: based on gRPC. Managed by the go/actools team
    (formerly Veneer team). It was also known as the GAPIC client library. See
    http://go/chc-grpc#gapic-client-libaries for more detail.
*   Google API Client libraries: provide access to HTTP interface only.

go/api-clients and https://cloud.google.com/apis/docs/client-libraries-explained
has all the One Platform information about client libraries, and the details for
the different types.

NOTE: We only have the HTTP-based Google API Client libraries for now, all
content below are for this type of client library.

## Existing Public Client Libraries

*   Java -
    https://github.com/googleapis/google-api-java-client-services/tree/master/clients/google-api-services-healthcare
*   Go -
    https://github.com/googleapis/google-api-go-client/tree/master/healthcare
*   NodeJS -
    https://github.com/googleapis/google-api-nodejs-client/tree/master/src/apis/healthcare
*   Python - no library, uses the
    [discovery](https://healthcare.googleapis.com/$discovery/rest) document
    directly.

## Update Process

The Java and Go client libraries are updated on a daily cadence automatically.
NodeJS libraries are manually updated by beckwith@ on a regular basis.

Client libraries are generated from the public
[discovery document](https://healthcare.googleapis.com/$discovery/rest) that is
updated on a weekly basis as part of our release cycle. The generation code uses
the [public directory](https://www.googleapis.com/discovery/v1/apis) to find the
respective discovery document.

## Generating the libraries internally

### Java

See google3/experimental/users/rwainman/healthcare/generated/java as an example.

The `java_apiary_library` BUILD target generates the client library, call `blaze
build` against it directly to see the generated client library code in the
`blaze-genfiles` directory.

### Go

See google3/experimental/users/rwainman/healthcare/generated/go as an example.

The `go_apiary_library` BUILD target generates the client library, call `blaze
build` against it directly to see the generated client library code in the
`blaze-genfiles` directory.

### NodeJs

It is not possible to generate the client library internally. See the external
[GitHub repository](https://github.com/googleapis/google-api-nodejs-client/tree/master/src/apis/healthcare).

However, you can generate the library outside of Google3 (based on instructions
from beckwith@):

```shell
git clone git@github.com:googleapis/google-api-nodejs-client.git
cd google-api-node-js-client
npm install
npm run generate -- 'https://healthcare.googleapis.com/\$discovery/rest?version=v1beta1'
cd src/apis/healthcare
npm install
npm pack
```

This process should leave you with a tarball that can be npm install'd by
clients who want to directly use the API.

A few thoughts and tips:

*   You'll want to replace the url up there with a path to your file. I don't
    know if it will work with a local file, it may only work via http (sorry
    about that).
*   Pay special attention to the quotes and escapes in the path. You need the
    single quotes, and if you have a $ in the path, escape it.
*   The `npm run generate` command may take a while, mostly because our
    formatter and linter take forever. Make sure your laptop is bolted down to
    the desk.

### Python

See google3/experimental/users/rwainman/healthcare/generated/python as an
example.

No client library assets are actually generated since the Python library simply
uses the discovery document.

## Code Samples

In addition to publishing external client libraries, we also publish code
samples written by the internal Developer Programs Engineering team.

Code Sample Tracker: https://devrel.corp.google.com/samples?product=healthcare

The tracker above then links out to the external repositories containing the
samples.
