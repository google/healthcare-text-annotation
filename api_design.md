# API Design

go/hcls-api-design

[TOC]

<!--*
# Document freshness: For more information, see go/fresh-source.
freshness: { owner: 'jiahuay', 'gradavis' reviewed: '2020-07-11' }
*-->

This page focuses on the design of a API within the HCLS team. For API review
process, see go/hcls-api-review instead.

Please read the [Get started](#get-started) references before deisgning an API.
If you need further help, contact hcls-api-discuss@ or book an
[API review office hour](go/hcls-api-review#office-hour)

## General API Design Resources {#resources}

### Get started {#get-started}

*   https://aip.dev/121: Resource-oriented design __(Absolute must read)__
*   https://aip.dev/122: Resource names
*   https://aip.dev/127: HTTP and gRPC Transcoding
*   AIP 131-135 Standard CRUD methods
*   https://aip.dev/136: Custom methods
*   https://aip.dev/126: Enumerations
*   go/hcls-breaking-changes: What are considered breaking changes
*   go/tf-api-review-template: Usability review template from TensorFlow team.
    Useful for new API designs.

### More refrences

*   HCLS
    *   API design guidelines go/hcls-api-design
    *   go/hcls-breaking-changes
    *   g/hcls-api-discuss for questions/discussions
*   Google-wide
    *   [API readability](http://go/api-readability)
    *   Public [aip.dev](https://aip.dev)
        *   AIP newsletter is available https://google.aip.dev/news.
        *   subscribe through g/api-announce
    *   Google private AIP go/aip-internal
    *   [API linter core rules](https://linter.aip.dev/rules/core/)
    *   g/api-discuss, YAQS
*   Other references
    *   [Legacy API Design Guide](https://cloud.google.com/apis/design/)
        (Predecessor of https://aip.dev)
    *   go/protodosdonts
    *   go/apidosdonts
    *   [One Platform Client Library Configuration](https://g3doc.corp.google.com/google/g3doc/oneplatform/client-config.md#client-library-configuration)
    *   Reference doc
        *   [One Platform docgen reference](https://g3doc.corp.google.com/google/g3doc/oneplatform/docgen_reference.md?cl=head#cross-references)
        *   [Google Developer Documentation Style Guide](https://developers.google.com/style/api-reference-comments)

## HCLS exceptions/conventions

TODO(b/159172518): create more comprehensive API style guide for HCLS

TODO(b/143101979): convert the HCLS API style guide into AIPs

A few of HCLS APIs are implementing external standards and introduce exceptions
against the standard API design practices in Google.

### Special method names

FHIR spec defines HTTP request path starting with '$': example:
[$lastn](https://cs.corp.google.com/piper///depot/google3/google/cloud/healthcare/v1alpha2/fhir/rest/fhir_service.proto?rcl=260201409&l=522).
The `rest_collection` and `rest_method_name` fields are set to generate
documentation correctly.

### Special request parameter names

*   Using underscore prefix for request field name.
    *   example:
        [_count](https://cs.corp.google.com/piper///depot/google3/google/cloud/healthcare/v1alpha2/fhir/rest/fhir_service.proto?rcl=260201409&l=1295).
    *   [reference](https://groups.google.com/a/google.com/g/envelope-framework-users/c/aZeLSKhsjDo/m/LyIYuNulEQAJ)

### FAQ

#### Why are some http mappings/visibility rules defined in the [protos](https://cs.corp.google.com/piper///depot/google3/google/cloud/healthcare/v1beta1/), while others are in the [yaml configs](https://cs.corp.google.com/piper///depot/google3/google/cloud/healthcare/)?

These are service configs(go/api-service-config). They can be defined either in
the yaml config files, or in the protos with
[Proto Annotations](http://go/api-service-config#proto-annotations). Common
proto annotations we use include `google.api.http`, `google.api.method_policy`,
`google.api.field_auditing)`, `google.api.field_visibility`, etc.

Notice that service config overrides proto definitions for the same API element,
see
[Service Config vs Proto](http://go/api-service-config#yaml-service-config-vs-proto)
for details.

#### Is there a way to override audit logging annotations?

A: Yes, audit logging annotations can be overridden in RPC security policy.
Check go/gin-annotation-ug#1-audit-directives and
go/autogin#configure-auditingconfig.

#### What's the (google.api.authz).permissions annotation used for?

A: It's used for
[public documentation generation](https://g3doc.corp.google.com/storage/zanzibar/iam/g3doc/public/documenting_permissions.md?cl=head).
The option is defined
[here](https://g3doc.corp.google.com/storage/zanzibar/iam/g3doc/public/documenting_permissions.md?cl=head).

#### Can I ignore Lint warnings for `active developer methods`?

Yes. These annotations will be added later, tracked in b/140626853.

#### Should an LRO that ran into errors set `result.Response` or `result.Status`?

If an LRO encountered any permanent errors that results in less than the
expected result, it should set the `result.Status` field with an appropriate
error code and message. Otherwise, set the `result.Response`. This is meant to
highlight to the customer that the LRO did not have the desired outcome, and
action is likely required.

One exception is our deid LROs, where errors may arrise (due to invalid dicom
files), and a mitigation is applied (the file is omitted). In this case we still
set the response field, because there is a mitigation in place (resource
omission is fine for deid, but not import/export), and because of the nature
deidentification ("failure" with partial success isn't useful for deid
customers).
