# Troubleshooting

[TOC]

## Access Token

If you have the following errors when trying to access the API, then it's very
likely that there is some issue with your access token.

### Missing Required Authentication Credential:

```shell
{
"error": {
  "code": 401,
  "message": "Request is missing required authentication credential. Expected OAuth 2 access token, login cookie or other valid authentication credential. See https://developers.google.com/identity/sign-in/web/devconsole-project.",
  "status": "UNAUTHENTICATED"
}
}
```

You either forget to supply an access token or your "access token" is not a real
access token. If your access token merely expires, the error will be different.

### Invalid Authentication Credentials:

```shell
{
  "error": {
    "code": 401,
    "message": "Request had invalid authentication credentials. Expected OAuth 2 access token, login cookie or other valid authentication credential. See https://developers.google.com/identity/sign-in/web/devconsole-project.",
    "status": "UNAUTHENTICATED",
    "details": [
      {
        "@type": "type.googleapis.com/google.rpc.DebugInfo",
        "detail": "Authentication error: 2"
      }
    ]
  }
}
```

The access token is expired. You need to get a refresh.

### Missing a Valid API Key:

```shell
{
  "error": {
    "code": 403,
    "message": "The request is missing a valid API key.",
    "status": "PERMISSION_DENIED"
  }
}
```

If you have a valid access token, then most likely the account (robot account in
most cases) you obtain the token from is not associated with a cloud project
that enables the API. Please refer to
[service account association](https://g3doc.corp.google.com/cloud/healthcare/g3doc/process/gcp_projects.md).

## Project Setup

### Unexpected Pass Through

If the setup is correct, unassociated robot accounts should not have access to
your API. If you see them pass, it is likely that you don't have the
[control environment](https://cs.corp.google.com/piper///depot/google3/google/api/control.proto?rcl=174125624&l=36)
configured properly. Unassociated robot accounts have zero quota, but if you
haven't configure quota check in the control environment, they can get pass.

### Google Service Control Is Not Enabled

If you see something like "Google Service Control is not enabled by the consumer
project" when you call the API, then you most likely don't have the Google
Service Control API enabled in the producer project of the your API. Note that
if you have
[control environment](https://cs.corp.google.com/piper///depot/google3/google/api/service.proto?l=232&cl=169988533)
set to staging, you will have to enable the corresponding Google Service Control
(Staging) in your producer project. The following commands can be useful.

```shell
# Check what APIs your project has enabled.
gcloud services list --enabled --project ${PRODUCER_PROJECT_ID?}
# Enable Google Service Control API.
gcloud service-management enable servicecontrol.googleapis.com --project ${PRODUCER_PROJECT_ID?}
# Enable Google Service Control API (Staging).
gcloud service-management enable staging-servicecontrol.sandbox.googleapis.com --project ${PRODUCER_PROJECT_ID?}
```

You may also need to add the borg MDB user as the Service Controller:

```shell
gcloud projects add-iam-policy-binding ${PRODUCER_PROJECT_ID?} \
      --member=user:<new_mdb_user>@prod.google.com \
      --role=roles/servicemanagement.serviceController
```

### "Method Not Visible To Labels", or "Method Not Found"

```shell
{
  "error": {
    "code": 404,
    "message": "Method not found.",
    "status": "NOT_FOUND",
    "details": [
      {
        "@type": "type.googleapis.com/google.rpc.DebugInfo",
        "detail": "Method ListDatasets not found for service autopush-healthcare.sandbox.googleapis.com. Method not visible to labels: {PUBLIC}"
      }
    ]
  }
}
```

You might also get a simple "Error 404: Method not found" message, for example
during the execution of integration tests:

```shell
Unable to create dataset due to error: googleapi: Error 404: Method not found., notFound
```

Your project is not allowlisted for [visibility](visibility.md) of the method.
Apply *GOOGLE_INTERNAL* visibility label to your project following
go/hcls-eng-tt-instructions.

### Permission 'servicemanagement.services.check' Denied

```shell
{
  "error": {
    "code": 500,
    "message": "Internal error encountered.",
    "status": "INTERNAL",
    "details": [
      {
        "@type": "type.googleapis.com/google.rpc.DebugInfo",
        "detail": "generic::permission_denied: Permission 'servicemanagement.services.check' denied for the consumer project [google.rpc.error_details_ext] { message: \"Permission \\'servicemanagement.services.check\\' denied for the consumer project\" details { type_url: \"type.googleapis.com/google.rpc.DebugInfo\" value: \"\\022\\350\\001Permission \\'servicemanagement.services.check\\' denied on service \\'sandman-healthcare.sandbox.googleapis.com\\' for user with ID 471926308727 and for policy \\'/00000046b50e2597/sandman-healthcare.sandbox.googleapis.com/0000000bd3998e0f\\'.\" } }"
      }
    ]
  }
}
```

Make sure you have enabled the corresponding "Google Service Control" API and
have granted Service Controller IAM permissions to the borg MDB user that is
providing the service (details in the
[Google Service Control Is Not Enabled](#google-service-control-is-not-enabled)
section). If the problem persists, it is possible that you have specified
[legacy.mdb](https://cs.corp.google.com/piper///depot/google3/google/api/legacy.proto?rcl=174955837&l=29)
setting in your YAML file. But it doesn't match with the LOAS role your backends
are running on, which is required to pass Service Control API permission check.

### Rejected by RpcSecurityPolicy

```shell
{
  "error": {
    "code": 403,
    "message": "The caller does not have permission",
    "status": "PERMISSION_DENIED",
    "details": [
      {
        "@type": "type.googleapis.com/google.rpc.DebugInfo",
        "detail": "[ORIGINAL ERROR] RPC::CREDS_POLICY_REJECTED_ERROR: RPC: Rejected by RpcSecurityPolicy: generic::unauthenticated: Rejected by creds_policy: Permission 'auth.creds.useNormalUserEUC' not granted to cloud-healthcare-test@prod.google.com, because it satisfies none of the 1 rules granting that permission. (Policy {type=creds, id=/creds-go/initial/6f60e837-5491-4648-b7b9-a9c9ca28307e}) (cloud-healthcare-test@prod.google.com, auth.creds.useNormalUserEUC)"
      }
    ]
  }
}
```

You are running backends on a mdb user that is not recognized by our
[authentication policies](https://cs.corp.google.com/piper///depot/google3/production/borg/cloud-healthcare/templates/rpcsp.borg?type=cs&l=16&cl=173102542).
You should run your backends using a mdb user that aligns with our per
environment policy. In the rare cases where you want to update the
[policy file](https://cs.corp.google.com/piper///depot/google3/production/borg/cloud-healthcare/templates/rpcsp.borg),
please send CLs to the team for review.

### Stale Service Config

After you inception_push a yaml file change, you have to reload the ESF borg job
with the latest yaml file build to actually get everything updated. Otherwise,
you won't see your yaml file change when you call the APIs.

### The API Doesn't Exist Or You Don't Have Permission

If an internal user cannot see the API even after being granted access into
go/hcls-trusted-testers, it is likely that they lack service consumer
permissions on the API's producer project. The error they receive may look
something like this:

    The API "staging-healthcare.sandbox.googleapis.com" doesn't exist or you don't have permission to access it.

To fix this they should be added to one of the MDB or email groups that has
service consumer permissions. Please reach out to the current oncall for help
figuring out which group you should be in. See the
[trusted testers page](../process/trusted_testers/adding_trusted_testers.md#adding_internal_trusted_testers)
for more details.

### Schema Mismatch or Spanner Conflicts

Although not very common, the schemas used for healthcare evolve with time and,
by default, your deployment will try to use your existing spanner database. That
being said, if you trying some change, it's possible that you will get
unexpected storage related errors from the endpoints. To mitigate that, try
running with spanner update flags:

```
./google/cloud/healthcare/scripts/dev_reload_servers.sh --update_db=true
or
./google/cloud/healthcare/scripts/dev_reload_servers.sh --force_reload_db=true
```

Be careful, as the second one wipes the existing data you might have. It's also
possible to see what's being written to spanner using the "span" client:

```
span query /span/nonprod/{{USERNAME}}:healthcare
```

Another tool that has proven useful to browse spanner is
[spanviewer](https://g3doc.corp.google.com/production/storage/chronicle/spanviewer/g3doc/index.md)

## Testing

### Use Your Own LOAS

If you want to run blaze test that involves auth to cloud healthcare API, you
need to enforce the test to use your own LOAS by

```shell
blaze test [target] --test_result=streamed --notest_loasd
```

`--test_result=streamed` makes sure your tests run locally - you won't get your
LOAS running on Forge.

`--notest_loasd` tells blaze test to use your machine LOAS; otherwise it will
use "insecure-shared-user".

### Errors on starting your own ESF instance

If you are getting _"Never responded on port 'esf' with monitor URL
localhost:14336/nullz (unknown error)"_ error message on attempt to start your
dev instance of ESF. This issue is caused by dependency on Discovery service, it
seems that ESF task cannot start without discovery registration, so make sure
you also start at least one backend job, e.g.
_'dataset-server-dev-{{USERNAME}}'_.

### Quota issue: rateLimitExceeded

The quota for the cloud-healthcare-test client project within each environment's
producer project is a custom setting. All integration tests as well as other
projects running in cloud-healthcare-test share the quota.

If you are getting "rateLimitExceeded" errors in integration tests, you should
assess if quota limit is an issue. Adjustments can be done via the Endpoints
page in pantheon. See
[Custom Project Settings](https://g3doc.corp.google.com/cloud/healthcare/g3doc/process/custom_project_settings.md).

For example, you can manage quota for staging
[here](https://pantheon.corp.google.com/endpoints/api/staging-healthcare.sandbox.googleapis.com/consumerquotas?api=google.cloud.healthcare.v1alpha2.dataset.DatasetService&project=cloud-healthcare-api-staging&folder&duration=PT1H&q=cloud-healthcare-test&consumerType=PROJECT).

## Stubby

### Set client side service name

If you are creating a stubby client to access the cloud healthcare API, without
renaming the service on client side, then you may run into errors:

    com.google.net.rpc3.client.RpcClientException: CLIENT_ERROR;google.cloud.healthcare.v1alpha2.fhir/FhirStoreService.TestIamPermissions;server does not have the client-specified method

This can be resolved by renmaing the service in your client, following
instructions
[here](https://g3doc.corp.google.com/cloud/healthcare/g3doc/customers/calling_from_borg.md#rename-service).
