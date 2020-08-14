# Outbound ACLs

go/hcls-outbound-acl

Our binaries use [OutboundACLs](http://go/velvet-rope:quick-guide) to restrict
dependencies to a well known, curated set of services. New binaries should have
their own OutboundACL added under
[cs/configs/production/outboundacl/cloud_healthcare](http://cs/configs/production/outboundacl/cloud_healthcare).
For the Cloud Healthcare API, be sure to document new dependencies at
go/chc-service-overview, go/chc-overview, and vi/cloud-healthcare.

## Writing a new OutboundACL

Review the documentation at go/outboundacl:quick to learn how to write new
OutboundACLs. The following command was helpful when writing the initial
OutboundACLs for the Cloud Healthcare Service:

```shell
  /google/bin/releases/dependency-rpc/tools/outboundacls/aclsuggest  -borg_user_jobs="cloud-healthcare/${BORG_JOB_NAME}"  -service_name="cloud_healthcare"
```

## Adding a New Service to an Existing OutboundACL

Add the new service you're calling to the OutboundACL config of the binary that
will make the RPC call. For example, to allow calls to `MyService` add something
like the following to the binary's OutboundACL config under
[configs/production/outboundacl/cloud_healthcare/](http://cs/configs/production/outboundacl/cloud_healthcare/):

```
dependency myservice = common.service_dep {
  service_name = 'MyService'
}
```
