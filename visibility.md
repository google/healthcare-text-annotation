# Cloud Healthcare API visibility

go/chc-visibility

[TOC]

Note: Before configuring visibility, make sure the user has access to the
API/Endpoint. See [Granting API access](api.md#access) for details.

## OnePlatform API visibility overview

OnePlatform API's can have their visibility restricted at various levels. Each
API has a PRODUCER and a CONSUMER. The Producer is the project which owns the
API, and the Consumer is the project which is attempting to access the API.

The producer can grant individuals, groups, or whole domains ("the consumer")
permission to enable the API in any project by adding the consumer to the
serviceConsumer role on the producer api project (cloud-healthcare-api in our
case).

The producer may additionally mark certain features as requiring that a
per-project visibility label be associated with the consumer's project. This
allows the producer to make an API visible to many users while restricting
new/sensitive/early features to a small subset of consumers.

See OnePlatform [API Visibility](/google/g3doc/oneplatform/visibility.md) for
more details on OnePlatform API visibility.

## Visibility entities

Visibility labels can be set to the different entities, see table below for
examples.

Entity            | Visibility configuration (prod)                                                           | Notes
----------------- | ----------------------------------------------------------------------------------------- | -----
Service           | [visibility.yaml](http://google3/google/cloud/healthcare/visibility.yaml)                 |
API method        | [visibility.yaml](http://google3/google/cloud/healthcare/visibility.yaml)                 |
Proto messages    | [visibility.yaml](http://google3/google/cloud/healthcare/visibility.yaml)                 |
IAM permissions   | [healthcare.service](http://google3/configs/storage/zanzibar/prod/iam/healthcare.service) |
API Documentation | [BUILD](http://google3/google/cloud/healthcare/BUILD?rcl=181623542&l=317)                 |
API Discovery     | [BUILD](http://google3/google/cloud/healthcare/BUILD?rcl=181623542&l=301)                 |

Usually when adding a new API or an API method you need to configure the
visibility for your Service/API method and the IAM permissions. And after that
regenerate documentation (see [documentation](documentation.md) page for
instructions).

## Visiblities configuration {#configuration}

### API visibility

API visibility is configured in
[visibility.yaml](http://cs/google3/google/cloud/healthcare/visibility.yaml) and
used by autopush, staging, and prod instances.

Sample CL for updating API visibility: CL/182082419.

Note: When API visibility is updated, you should also check if IAM visibility
should be updated accordingly.

### IAM visibility

IAM visibility is configured in _healthcare.service_ files for
[dev](http://google3/configs/storage/zanzibar/dev/iam/healthcare.service),
[staging](http://google3/configs/storage/zanzibar/staging/iam/healthcare.service)
and
[prod](http://google3/configs/storage/zanzibar/staging/iam/healthcare.service)
IAM environments.

Sample CL for updating IAM visibility: CL/182591353.

## Cloud Healthcare visibility labels {#labels}

Check google3/google/cloud/healthcare/visibility.yaml for the list of visibility
labels used in the CHC service.

To make all entities mentioned above visible to any new consumer project you
need to assign a corresponding label to the project.

Please follow go/hcls-adding-trusted-testers to assign a label to the Consumer
project.

## Visibility config troubleshooting

### Missing visibility label for enum default value

Example error message during presubmit:

> ERROR: google/cloud/healthcare/v1alpha2/dictation/service.proto:1085:3: The
> default value of 'google.cloud.healthcare.v1alpha2.dictation.TrainingConsent'
> cannot be hidden. It is hidden because its required visibility
> '{GOOGLE_INTERNAL}' is not available. Current active visibility is
> '{CHC_DICTATION}'.

This error message is seen when a visibility label is set for a enum but not set
for its default values.

E.g. When CHC_FOO is set for `google.cloud.healthcare.v1alpha2.BarEnum` but not
for google.cloud.healthcare.v1alpha2.BarEnum.DEFAULT_VALUE, then
google.cloud.healthcare.v1alpha2.BarEnum.DEFAULT_VALUE applies default
visibility restriction, and remains unvisibile for CHC_FOO.

This configuration is invalid for projects with CHC_FOO label, where clients can
see the enum, but its default value is hidden.

Blaze/rabbit captures this invalidity while compiling the api_service BUILD
target, and renders error messages similar to above.

## Visibility diff tool

We have created a tool to generate visibility diffs to help verify that a CL
modifying protos or visibility rules has its intended effects. It can be run
against a pending CL, or against an already submitted CL.

To run the tool against a given CL, you can use the following command. The
output will contain all services, methods and types that have their visibility
affected by the CL.

Note: By default cldiff will read from the latest Critique snapshot of your
pending CL. If you want it to read the latest versions of files from the
underlying citc workspace, add `--use_citc_workspace`.

```shell
/google/bin/releases/cloud-healthcare/visibilitydiff/cldiff --cl=123456
```

The presubmit requires that you run the tool and add the CNS link in the
description of your CL with the tag `VISIBILITY_DIFF_URL=<url>` on any
visibility or public API proto changes.

> Note: Use the `--service_definition` flag to diff a service other than Cloud
> Healthcare, e.g. for Cloud Life Sciences:
>
> ```shell
>   /google/bin/releases/cloud-healthcare/visibilitydiff/cldiff --cl=123456 \
>        --service_definition=google/cloud/lifesciences/lifesciences_service.pb
> ```

### Oneofs (or: Why am I seeing oddly-named fields in my diff?)

Protobuf oneofs fields are generated with a strange name. They skip the name of
the `Message` they are contained in, so if the message `google.cloud.Foo`
contains a oneof field named `bar`, it will show up in the diff as
`google.cloud.bar`.

**This is not a bug in the visibility diffing tool.**

The OnePlatform visibility logic does pick up on oneofs following this naming
scheme, so please make sure that they appear under the correct visibility label
in your changes. We are not 100% clear on how oneofs are treated in every
possible situation so for now we are erring on the side of having a bit of extra
verbosity in our visibility rules.

### Base CL selection

In order to provide non-noisy diffs, the `cldiff` tool will attempt to determine
the correct base CL to compare to via the following method:

*   If the CL is submitted, compare against CL-1.
*   If the CL is pending:
    *   If there is no diffbase, use the base CL that the local workspace is
        synced to
    *   If there is a diffbase, use the diffbase CL's workspace

### Releasing visibilitydiff

Currently visibilitydiff is released manually to binfs. This will eventually be
automated. Releases can only include submitted CLs, and can only be triggered by
members of mdb/cloud-healthcare-releaser. The following command will push a new
version of the `cldiff` and `mpmdiff` tools built at `HEAD`. For more command
line options, see the
[binfs docs](http://g3doc/storage/binfs/g3doc/deploy_from_command_line).

```shell
cloud/healthcare/tools/visibilitydiff/deploy.sh
```
