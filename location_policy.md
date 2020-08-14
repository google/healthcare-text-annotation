# Location Policy

go/hcls-location-policy

[TOC]

The location policy is enforced by the ESF when it calls Chemist for every
request.

See go/chemist-location-policy for more information.

It essentially prevents users from creating resources in regions they should not
be using. At this point in time (July 2020), the Cloud Healthcare API is not
deployed in any location-policy restricted region. However, the
`southamerica-east1` region is one we plan to launch in Q3 2020 and is a
restricted region.

**IMPORTANT:** This is a policy that is applied to ALL projects. This policy
should not be confused with the Location **Org** Policy which allows customers
to restrict the regions used by their projects.

## Proto Annotations

The location policy check is enforced on all fields within our API protos that
contain the
[`location_selector`](https://source.corp.google.com/search?q=location_selector%20lang:proto&sq=&ss=piper%2FGoogle%2FPiper:google3%2Fgoogle%2Fcloud%2Fhealthcare%2F)
annotation. The contents of this field must then always contain a **valid**
region name within its path.

For more information on the application of this annotation, see the
go/location-restriction-checks-sig.

## Testing

### Manual Testing

To manually verify an annotation, try sending a request where the field contains
an invalid region name (for example `europe-west5` is not allowed since it is a
private region).

Note that these tests should still fail with the Chemist location policy even if
the Cloud Healthcare API is not deployed in this region. That is because this
check is enforced at the ESF level before the request even hits our backends.

```shell
$ source google/cloud/healthcare/scripts/chc-api-funcs.sh
$ endpoint autopush
$ get-auth-token
# Fails with Chemist Location Policy (restricted region)
$ LOCATION=europe-west5
$ create-dataset
{
  "error": {
    "code": 403,
    "message": "Permission denied on 'locations/europe-west5' (or it may not exist).",
    "status": "PERMISSION_DENIED",
    "details": [
      {
        "@type": "type.googleapis.com/google.rpc.DebugInfo",
        "detail": "Location 'europe-west5' is prohibited by 'Region' policy for 'projects/rwainman-test' container in 'autopush-healthcare.sandbox.googleapis.com' service."
      }
    ]
  }
}
# Fails with Chemist Location Policy (invalid region since europe-west10 is not
# yet a valid region name)
LOCATION=europe-west10
$ create-dataset
{
  "error": {
    "code": 403,
    "message": "Permission denied on 'locations/europe-west10' (or it may not exist).",
    "status": "PERMISSION_DENIED",
    "details": [
      {
        "@type": "type.googleapis.com/google.rpc.DebugInfo",
        "detail": "Location 'europe-west10' is prohibited by 'Invalid Location' policy for 'projects/rwainman-test' container in 'autopush-healthcare.sandbox.googleapis.com' service."
      }
    ]
  }
}
# Not enforced by the Chemist Location policy since the region name is valid but
# not yet enabled by the Cloud Healthcare API.
LOCATION=europe-west3
{
  "error": {
    "code": 400,
    "message": "location ID invalid, want us-central1, got europe-west3",
    "status": "INVALID_ARGUMENT",
    "details": [
      {
        "@type": "type.googleapis.com/google.rpc.DebugInfo",
        "detail": "[ORIGINAL ERROR] generic::invalid_argument: location ID invalid, want us-central1, got europe-west3, cause (internal only): location ID invalid, want us-central1, got europe-west3 [google.rpc.error_details_ext] { message: \"location ID invalid, want us-central1, got europe-west3\" }"
      }
    ]
  }
}
```

You can tell that this is enforced by the Chemist Location Policy by looking at
the error message detail.

### Integration Tests

Integration tests are available for this in
google3/google/cloud/healthcare/test/v1alpha2/conformance/locationpolicyintegration_test.go.

For now, these tests verify the behavior for root-level resources.
