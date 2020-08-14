# Cloud Healthcare API Service Accounts

This doc: go/hcls-service-accounts

Cloud Healthcare is integrated with IAM according to [these
instructions](http://g3doc.corp.google.com/storage/zanzibar/iam/g3doc/public/faq.md#i-want-to-integrate-my-service-with-iam-do-you-have-any-guidance).

## Curated IAM Role

A curated IAM service agent role is used to grant the smallest possible set of
permissions to API service accounts. The `healthcare.serviceAgent` role is
defined in the service_roles.service file for each IAM environment:

*   google3/configs/storage/zanzibar/prod/iam/service_roles.service
*   google3/configs/storage/zanzibar/staging/iam/service_roles.service
*   google3/configs/storage/zanzibar/dev/iam/service_roles.service

## Service Account Projects

Healthcare service accounts will be created in service account projects named
after their environment. Namely gcp-sa-healthcare, gcp-sa-healthcare-staging,
gcp-sa-healthcare-autopush, gcp-sa-healthcare-dev and gcp-sa-healthcare-sandman.

NOTE: The service account projects differ from the API producer projects. This
ensures that the principle of least privilege can be maintained for service
accounts. See
go/tm-service-account#why-should-service-accounts-be-stored-in-a-separate-project
for details.

## Service Account Quota

There is a quota for how many service accounts can be created per Healthcare
service account project. The current quota is tracked in b/65120785.

The best practice is to request enough quota for 2-5 years of service account
activations. It is not possible to set an unlimited service account quota at
this time (this includes "essentially unlimited" requests like MAXINT).

The [IAM
FAQ](http://g3doc.corp.google.com/storage/zanzibar/iam/g3doc/public/faq.md#how-can-i-request-a-quota-increase-for-iam)
outlines the process for increasing service account quota. Note, that the quota
increase will need to be for relevant the gcp-sa-healthcare.* project.

The [IAM
FAQ](https://g3doc.corp.google.com/storage/zanzibar/iam/g3doc/public/faq.md#monitor-sa-quota)
also includes the following command which gets the count of service accounts in
a project and the current maximum number:

```shell
/google/data/ro/projects/gaiamint/bin/get_mint --type=loas --text --endusercreds\
  --scopes=24100,35600,84500 --out=/tmp/auth.txt
stubby --rpc_creds_file=/tmp/auth.txt --proto2 call blade:iam-fe-prod-esf \
IAM.GetProjectSettings 'name: "${PROJECT_ID?}"'
```

## Service Activation

The following activation hook in the google3/google/cloud/healthcare/prod.yaml
service configuration enables the creation of service accounts by
Inception/Tenant Manager on API activation:

```
usage:
  activation_hooks:
  - serviceusage.googleapis.com/service-account
```

Inception or Tenant Manager will create an API service account the Healthcare
API is enabled on a project. The service account will have a name like
`service-1234@gcp-sa-healthcare.iam.gserviceaccount.com`, where "1234" is the
project's id.

go/iam-serviceaccount-creation-with-inception details all of the steps required
to get service account creation working. The service account creation and
initial permission grant are configured at
google3/configs/production/cdpush/acl-zanzibar-cloud-prod/activation_grants/activation_grants.gcl?l=1659&rcl=237894616

## Service Accounts in Dev Personal Instances

For personal API instances there is no service account activation hook, so by
default we use the cloud-healthcare-eng robot account
(cloud-healthcare-eng@system.gserviceaccount.com) for all access.

Note: A consequence is that `cloud-healthcare-eng@system.gserviceaccount.com`
will require the `Editor` role in your consumer project if you want to export
stores to a GCS bucket or BigQuery table.

To test service account related permissions in DEV you can manually create your
own project service account and instruct your dev instance to use it.

1.  Create a service account in Pantheon named
    testsvc-${PROJECT_NUMBER?}@${PROJECT_ID?}.iam.gserviceaccount.com
2.  Give cloud-healthcare-eng@prod.google.com "Service Account Token Creator"
    role on the project. This allows the dev instance MDB group to impersonate
    the service account you just created.
3.  Modify the borg configs to use your service account:

cs/production/borg/cloud-healthcare/templates/healthcare_dev.borg

```
  robot_email_type = 'PROJECT'
  robot_email = "testsvc-%%d@<project_id>.iam.gserviceaccount.com",
```
