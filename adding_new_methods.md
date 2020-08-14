# Adding new API methods

go/chc-new-methods

[TOC]

If you are new to Google Cloud APIs, familiarize yourself with the API Style
Guide at go/api-style and follow it when designing your API.

Steps to follow when updating our API.

## Define the API

*   Change or create the external service proto(s) (e.g. v1alpha2,
    grpc_service). (eg. CL/184047723). Get this change reviewed by someone
    locally on your team and then send to chc-reviews to have an API reviewer
    autoassigned.
*   New API methods, messages, and fields *must* have the method marked
    GOOGLE_INTERNAL in
    [visibility.yaml](http://google3/google/cloud/healthcare/visibility.yaml)
    (e.g.
    [CL/182082419](http://cl/#review/223564181/depot/google3/google/cloud/healthcare/visibility.yaml)).
    See the [visibility page](visibility.md) for more details on OnePlatform
    visibility and Healthcare visibility configuration.
*   See go/hcls-api-reviews#the-review-process for the review process.

## Add new method to service binary

Example CL/181353405

*   Change internal service proto
*   Add stub method implementation
*   Update corresponding shims (e.g. shimv1alpha2.go) and their unit tests.
*   If stand-alone GRPC shim exists (e.g. grpcshimv1alpha2.go), update the GRPC
    shim and unit tests. (e.g.
    [FHIR](http://cr/#review/211491020/depot/google3/cloud/healthcare/fhir/fhir_store/grpcshimv1alpha2.go))

## Set up ESF and borg config

NOTE: if your new method is matched by one of the rename_rules, and it will be
served by the corresponding service backend, you can skip this step.

Example CL/202233424 (modulo the proto changes)

*   See g3doc/google/g3doc/oneplatform/basics for reference.
*   Set up mapping rules between http and RPC requests. (This can be done in the
    service proto, or in healthcare.yaml)
*   Add security policy in healthcare.yaml. e.g. cl/204325678
*   Set up quota for new methods in
    [quota.yaml](http://cs/google3/google/cloud/healthcare/quota.yaml).
    **Important**: if this functionality is hidden behind a visibility label, it
    must be annotated with a visibility label (see examples for `CHC_TELEHEALTH`
    and `CHC_DICTATION` within the file).
*   Set up the service backend in healthcare.borg. Make corresponding change for
    sandman in healthcare.gcl.
*   Make sure your binary's borg file has a proper RPC security policy entry
    (flag `rpc_security_policy_initial`).
*   Update go/chc-overview, go/chc-service-overview, and vi/cloud-healthcare
    with any new dependencies being introduced.
*   Update go/hcls-method-dependencies with your new method and its
    dependencies.
*   Add a new or update the existing
    [OutboundACL config](http://cs/configs/production/outboundacl/cloud_healthcare/)
    with your new service and new dependencies if applicable. See
    go/hcls-outbound-acl and go/outboundacl:quick for details.

## Set up integration test

*   If a new service proto is added, add the dependency to
    [COMMON_DEPS](https://cs.corp.google.com/piper///depot/google3/google/cloud/healthcare/BUILD?rcl=204321715&l=10)
*   Add integrations for the new methods in
    google3/google/cloud/healthcare/test/v1alpha2/conformance (and all supported
    versions)

## IAM permissions {#iam}

1.  Add IAM permissions and roles to the Healthcare IAM configurations for IAM
    dev and staging, following [Modifying IAM config](#modify-iam-config). Make
    sure the IAM permissions and roles have the same visibility labels as the
    API.
1.  Add the IAM permission to IAM-prod. Leave the permission type fields
    (example value: DATA_READ, DATA_WRITE) for later.

#### Modifying IAM config: {#modify-iam-config}

1.  Update the `healthcare.service config` in your desired IAM environment

    *   iam-dev:
        [healthcare.service](http://google3/configs/storage/zanzibar/dev/iam/healthcare.service)
    *   iam-staging:
        [healthcare.service](http://google3/configs/storage/zanzibar/staging/iam/healthcare.service)
    *   iam-prod:
        [healthcare.service](http://google3/configs/storage/zanzibar/prod/iam/healthcare.service)

        Note that externally visible (i.e. non-label protected) prod IAM
        permissions and roles must be tested via our
        [custom roles testing setup](http://google3/cloud/testing/iam/json/surface/cloud_healthcare/dicom).
        Add a new test case if you add a new prod permission (or remove
        visibility labels).

1.  Run the zpp2zan tool to ensure that it is structured correctly and correct
    as necessary.

    ```shell
    g4d
    storage/zanzibar/iam/zpp2zan.sh
    ```

1.  Send the review to chc-reviews.

1.  After code review is approved by a team member, add `iam-codereview@` to the
    reviewers list.

1.  Once the CL is approved by IAM code reviewer, regenerate the .zan files
    after syncing to HEAD:

    ```shell
    g4d
    g4 revert .../*.zan
    g4 sync
    storage/zanzibar/iam/zpp2zan.sh

    ```

1.  Submit the CL.

## Implement the methods

*   Add used IAM permissions for the methods to
    [permissions.go](http://google3/cloud/healthcare/aaa/permissions.go) file.
*   Start to actually implement API methods. Do not submit the implementation
    until all used permissions are rolled out to iam-staging. Check
    go/iam-config-rollout-single for progress of the rollout.
*   Update integration tests accordingly.
*   [Add the methods to APISPec](./integration_tests.md#new-method), which will
    add IAM, Cloud Audit and VPC testing for the method.
*   If the used permissions has not been rolled out to iam-prod yet, keep both
    the integration tests and IAM tests disabled in CHC staging and prod using
    go/chc-cleft. Enable them when the rollout is complete.

Note: there can be a 0.5-2 week gap between the iam-staging rollout and iam-prod
rollout, depending on when you submit those two CLs. IAM config rollout schedule
can be found at go/iam-config-push-schedule

## Set proper audit logging

*   See go/cal-integ#audit-rules for guidance and examples.
*   Set (google.api.method_auditing).directive on methods
*   Set (google.api.field_auditing).directive on fields
*   Where a field needs to be conditionally audit-logged only for some
    operations use AUDIT_REQUEST_AND_RESPONSE or AUDIT_REQUEST in the method and
    AUDIT in the field.
*   Consider marking (datapol.semantic_type) with ST_HEALTHCARE_INFO or other
    tags to disable logging for sensitive or protected healthcare data.
*   Run the IAM integration tests against autopush or your dev instance to
    generate audit logs for the new permission. Your logs should be visible at
    http://pantheon/logs/viewer?mods=logs_tg_test.
*   Get a Cloud Audit Logging review -
    go/cloud-audit-logging-integration-howto - run the validation tool against
    the generated logs.
*   If you haven't, add the type field (example value: DATA_READ, DATA_WRITE) to
    your permissions in IAM config, following
    [Modifying IAM config](#modify-iam-config), in all IAM environments.
*   If your method generates a LRO (long running operation) add proper operation
    metadata
    ([documentation](https://g3doc.corp.google.com/google/cloud/audit/g3doc/site/integration/long-running-operation.md?#start-lro))
    at request time ([example](http://cl/294722975)).

## Add CUJ to CEP workflows {#prober}

Probers are needed to alert the oncall of any unexpected behavior or loss of
functionality in production at the method level. See go/chc-cep for detailed
introduction on adding new CEP workflows encoding CUJs involving the new API.

## SLO dashboard and SLO alerts

*   Add the method to your service's SLO dashboard file at
    google3/configs/slo/repository/cloud_healthcare/. More details about setting
    SLOs and instructions for testing the dashboard can be found at
    go/hcls-slo-process.

*   Include your method in the SLO alerts configuration at
    google3/configs/monitoring/cloud_healthcare/slo/lib.py. Use the same
    thresholds specified for the SLO dashboard.

## Remove or update visibility restrictions

Once you have a launch approved via go/chc-launch-process then you can modify
the [visibility.yaml](http://google3/google/cloud/healthcare/visibility.yaml)
file as follows:

*   To make visible to all projects for Beta/GA customers, simply remove any
    visibility restriction for the method(s) you are launching.
*   To make visible only to select projects with a more restricted set of
    customers then add CHC_ALPHA (or similarly restrictive label as
    appropriate).

## Regenerate documentation

API documentation is requested by the
[HCLS Legislator process](https://g3doc.corp.google.com/cloud/healthcare/g3doc/process/release.md#monday-2)
which automatically generates a bug in the
[API Docs bug component](https://b.corp.google.com/issues?q=componentid:780323%20status:open)
once a release is released to all regions. A writer on the hcls-docs@ team will
start the doc generation process within two days of the bug creation. For more
information, see
[Generating API Documentation](https://g3doc.corp.google.com/cloud/healthcare/g3doc/documentation/api_reference_documentation.md).
