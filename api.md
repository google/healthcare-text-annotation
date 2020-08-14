# Cloud Healthcare API

go/chc-api

[TOC]

We followed the steps in go/api-deploy to get the API up and running in
production. The resulting configs and some helpful tips are documented below.
See go/api-deploy, go/api-codelab, and go/api-blueprint for more details.

## Environments

name     | service control                               | endpoint                                               | producer project                                                                                             | config file                                                           | GFE config
-------- | --------------------------------------------- | ------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------- | ----------
prod     | servicecontrol.googleapis.com                 | https://healthcare.googleapis.com                      | [cloud-healthcare-api](http://pantheon/iam-admin/iam/project?project=cloud-healthcare-api)                   | [prod.yaml](http://google3/google/cloud/healthcare/prod.yaml)         | [prod gfe config](http://google3/googledata/production/gslbpush/sources/cloud-healthcare/gfe.part.cfg)
staging  | staging-servicecontrol.sandbox.googleapis.com | https://staging-healthcare.sandbox.googleapis.com      | [cloud-healthcare-api-staging](http://pantheon/iam-admin/iam/project?project=cloud-healthcare-api-staging)   | [staging.yaml](http://google3/google/cloud/healthcare/staging.yaml)   | [staging gfe config](http://google3/googledata/production/gslbpush/sources/cloud-healthcare-staging/gfe.part.cfg)
autopush | staging-servicecontrol.sandbox.googleapis.com | https://autopush-healthcare.sandbox.googleapis.com     | [cloud-healthcare-api-autopush](http://pantheon/iam-admin/iam/project?project=cloud-healthcare-api-autopush) | [autopush.yaml](http://google3/google/cloud/healthcare/autopush.yaml) | endpoints in yaml file
sandman  | staging-servicecontrol.sandbox.googleapis.com | https://sandman-healthcare.sandbox.googleapis.com      | [cloud-healthcare-api-sandman](http://pantheon/iam-admin/iam/project?project=cloud-healthcare-api-sandman)   | [sandman.yaml](http://google3/google/cloud/healthcare/sandman.yaml)   | endpoints in yaml file
dev      | staging-servicecontrol.sandbox.googleapis.com | https://{{USERNAME}}-healthcare.sandbox.googleapis.com | [cloud-healthcare-api-dev](http://pantheon/iam-admin/iam/project?project=cloud-healthcare-api-dev)           | [dev.yaml](http://google3/google/cloud/healthcare/dev.yaml)           | endpoints in yaml file

These environments are also defined and available to be called by any Boq
services using the
[Boq service registry](http://google3/configs/social/service_registry/base.pi?l=3952&cl=244042760).

## Where to find things in production

There are several GSLB targets available for each environment. See
[GSLB page](https://g3doc.corp.google.com/cloud/healthcare/g3doc/production/gslb.md)
for details.

Borg configs are at google3/production/borg/cloud-healthcare. The borg jobs
serving each of the above environments can be easily found just using the
environment name in a job regex:

*   [prod](http://sigma/jobs/cloud-healthcare%2E*/prod)
*   [staging](http://sigma/cloud-healthcare-staging)
*   [autopush](http://sigma/cloud-healthcare-autopush)
*   [dev](http://sigma/jobs/cloud-healthcare-eng/{{USERNAME}})
*   [sandman](http://sigma/jobs/cloud-healthcare-test/.*healthcare.*) (used in
    presubmit/cbuild)

## Deployment instructions

Grant Service Controller IAM permissions on the producer project to the MDB
group serving the backend job (`cloud-healthcare` for prod, staging, and
autopush and `cloud-healthcare-eng` for dev).

Enable the Service Control API in the producer project (and Service Control
Staging API for non-prod deployments).

To deploy the autopush API, run:

```shell
/google/data/ro/teams/oneplatform/inception_tool push-service-nonprod --env=PROD  --build_target //google/cloud/healthcare:autopush-healthcare
```

API deployments usually take around 30 minutes (longer for the initial
deployment).

To check on the status of a deployment run the command of the following form
from near the end of the output of the previous command:

```shell
/google/data/ro/teams/cloud-sdk/gcloud endpoints operations describe 'operations/rollouts.autopush-healthcare.sandbox.googleapis.com:<some-id>'
```

To check on the health of a deployed API:

```shell
/google/data/ro/teams/oneplatform/launch_checker --service="autopush-healthcare.sandbox.googleapis.com"
```

You can also check the endpoints page in the producer project to verify that the
API has been deployed:

http://pantheon/endpoints/api/autopush-healthcare.sandbox.googleapis.com/overview?project=cloud-healthcare-api-autopush

## Granting API access {#access}

To grant access to the API, go to the
[endpoints page](http://pantheon/endpoints/api/autopush-healthcare.sandbox.googleapis.com/overview?project=cloud-healthcare-api-autopush)
in the producer project. Then,

-   If your user/group does not belong to any of the existing allowlisted
    groups, add a new allowlist group to the _Service Consumer_ role in the API
    endpoint's producer project. See
    [Controlling Who Can Enable Your API](http://cloud/endpoints/docs/openapi/control-api-callers)
    for steps.

Note that, although the API access is granted, the API might not be visible to
consumer projects yet. You need to also configure visibility for the API.

### Configuring API visibility

API visibility is managed with allowlists. An API is configured to be visible to
certain allowlists, and a project needs to be allowlisted for the API to be
visible.

By default, new API is visible to the Google internal Service Consumers (e.g.
the probers). In order to make your API visible in EAP (Alpha/...), follow
[visibility configuration](visibility.md#configuration).

To allowlist a project, follow go/health-cloud-project-allowlisting. For Cloud
Healthcare team internal projects, GOOGLE_INTERNAL label should be used.

## Testing {#testing}

IMPORTANT: Make sure to add yourself to
[cloud-healthcare-eng-team](http://mdb/cloud-healthcare-eng-team) to avoid
permission issues.

Tip: If you're getting errors here after allowlisting the project, check out the
[troubleshooting page](troubleshooting.md) for possible causes and solutions.

Prepare `gcloud` command:

```shell
alias gcloud=/google/data/ro/teams/cloud-sdk/gcloud
```

If you don't already have a test project then create one:

```shell
PROJECT={{USERNAME}}-test
gcloud config set account {{USERNAME}}@google.com
# Create a project in the 'experimental' GCP project folder.
gcloud projects create --folder=261046259366 "${PROJECT:?}"
```

Note: The last command might fail if you didn't previously accept the
`LOAS-{{USERNAME}}` project _Terms of Service_ in the
[Cloud Platform Console](http://pantheon/home/dashboard?project=loas-{{USERNAME}}).

It takes a couple minutes for your project to be ready.

Enable the API in your project, e.g. for Autopush:

```shell
gcloud services enable autopush-healthcare.sandbox.googleapis.com --project=${PROJECT:?}
```

And for dev (requires [creating your own API endpoint](#bringing-up-dev-esf)):

```shell
gcloud services enable {{USERNAME}}-healthcare.sandbox.googleapis.com --project=${PROJECT:?}
```

You can then monitor API request stats from your
[dashboard](http://pantheon/apis/api/).

Allowlist your project for the autopush API following
go/health-cloud-project-allowlisting.

To issue test requests from the command line you need an access token that meets
two requirements:

*   associated with a project in which the Cloud Healthcare API has been
    activated (this is enforced by using an OAuth 2.0 Client ID from this
    project)
*   auth scope of https://www.googleapis.com/auth/cloud-healthcare (needs to be
    set when requesting the access token)

There are several ways to obtain an access token for testing. The first is to
download the JSON client secret for the OAuth 2.0 Client ID on the test client
project. It's fine to reuse an existing OAuth 2.0 Client ID (e.g. "Testing OAuth
Client" in the mllp-adapter-test project). Click the Download JSON button to get
the secret.

http://pantheon/apis/credentials?project=mllp-adapter-test

Make sure you have oauth2l (OAuthTool) installed:

```shell
sudo apt-get install python-pip
sudo pip install google-oauth2l
```

Then run the following to obtain an access token:

```shell
oauth2l fetch --json <path/to/client_secret.json> https://www.googleapis.com/auth/cloud-healthcare
```

> NOTE: If you get an import error for opentype, run:
>
> ```shell
> sudo pip install --upgrade google-auth-oauthlib
> ```
>
> If it still throws the same error, try upgrading pip from PyPI, you may use:
>
> ```shell
> sudo easy_install -U pip
> ```
>
> Then you may have to use the absolute path to invoke the upgraded pip:
>
> ```shell
> sudo /usr/local/bin/pip install --upgrade google-auth-oauthlib
> ```

If this is your first time getting a token, it will prompt you to go to a URL to
authenticate your google account. After that you can run the following command
to put the token directly into a shell variable:

```shell
TOKEN=`oauth2l fetch --json <path/to/client_secret.json> https://www.googleapis.com/auth/cloud-healthcare` &&
echo $TOKEN
```

Then issue a test GET request:

```shell
curl https://autopush-healthcare.sandbox.googleapis.com/v1alpha2/projects/${PROJECT:?}/locations/us-central1/datasets?access_token=${TOKEN:?}
```

Or POST request:

```shell
curl -X POST --data '' https://autopush-healthcare.sandbox.googleapis.com/v1alpha2/projects/${PROJECT:?}/locations/us-central1/datasets?access_token=${TOKEN:?}\&dataset_id=test-dataset
```

Alternative ways to get credentials for testing:

*   Follow the steps at
    http://sites/apiary/home/launch-your-api/monitoring/setting-up-probers/storing-oauth-credentials-in-keystore
    but skip the parts about putting credential into keystore and stop at the
    step where you test with oauthcurl.
*   From a VM with the appropriate scope set up (e.g. "hospital-shell3" in the
    fake-hospital project
    http://pantheon/compute/instances?project=fake-hospital&organizationId=433637338589)
    you can access credentials inside a VM shell as follows:

```bash
export AUTH="Authorization: Bearer $(gcloud config config-helper --format='value(credential.access_token)')"
curl -X GET https://${USER}-healthcare.sandbox.googleapis.com/v1alpha2/projects/${PROJECT:?}/locations/us-central1/datasets -H "${AUTH:?}"
```

It is harder to debug errors with this approach since full internal debugging
information isn't included in responses.

### Dev Instances

To do testing of unsubmitted changes in dev environments you can bring up the
relevant jobs using borgcfg and then either bring up a dev ESF or create a
personal dev API target to act as a local ESF proxy.

To start the dev jobs run the following script:

```shell
google/cloud/healthcare/scripts/dev_reload_servers.sh --local_services=<LIST_OF_SERVICES>
```

`--local_services` specifies which backends should be built locally; it takes
either a comma separated list of services (see
[here](http://google3/google/cloud/healthcare/scripts/dev_reload_servers.sh) for
a full list) or 'all'. All other backends will use the latest MPM.

`--only_local` triggers only the build/deploy of the services specified by the
previous flag, which is very useful during development. Be aware that everything
should be deployed initially.

If the dev database is out of date, reload it by setting `--force_reload_db` to
true. Before reloading the server, disable drop protection on your database.

```shell
span setdropdatabaseprotection /span/nonprod/{{USERNAME}}:healthcare false
```

To bring up the dev jobs manually, build the target with `rabbit --verifiable`
rather than blaze (see go/verifiable-builds for more details) and then do a
`borgcfg up`:

```shell
rabbit --verifiable build cloud/healthcare/datasets:main && \
borgcfg production/borg/cloud-healthcare/dev/healthcare_dev.borg reload healthcare.dataset-server --vars=cell=ik --skip_confirmation
```

NOTE: Change the command accordingly if you want to build servers other than
dataset. (e.g. DICOM or FHIR server).

Then to test HTTP or gRPC requests you need to bring up an ESF proxy, which can
be done via bringing up a dev ESF.

#### Manually reloading jobs

If you only need to reload a job to make a small flag change, you can use the
following:

```shell
borgcfg production/borg/cloud-healthcare/dev/healthcare_dev.borg reload healthcare.meta-server --vars=cell=ik,mpm_pkg_label=latest,span_instance=/span/nonprod/{{USERNAME}}:healthcare --live_diff
```

You can replace `meta-server` with whatever job name you want.

#### Bringing Up Dev ESF

First you need to create your own API endpoint (this only needs to be done once
for each time that you change dev.yaml):

```shell
google/cloud/healthcare/scripts/dev_inception_push.sh
```

You can check on the status of the inception push using:

```shell
/google/data/ro/teams/oneplatform/service_checker --service="{{USERNAME}}-healthcare.sandbox.googleapis.com" --inception_env=PROD
```

**IMPORTANT: If this is your first time running an inception push for your
environment, you will need to ask on-call to run it for you. File a bug similar
to b/150290368 and assign it to the
[current secondary on-call](https://oncall.corp.google.com/cloud-healthcare)**.

Alternatively, you can run the command near the end of the output of the
inception_push command.

Once the API has been pushed follow the instructions [above](#testing) to enable
the new API in a client project and get an access token using OAuth.

Finally, you can bring up the ESF Borg job using a simple borgcfg up:

```shell
borgcfg production/borg/cloud-healthcare/dev/healthcare_dev.borg up healthcare.esf --vars=cell=ik
```

NOTE: If you want to bring up a **locally-built ESF** on Borg, modify the
`production/borg/cloud-healthcare/templates/esf.borg` file per these
[instructions](/tech/internal/env/framework/g3doc/production.md?cl=head#config-your-job-to-use-a-local-extension-binary).
Then use the `borgcfg` command above to bring up the job.

IMPORTANT: Make sure the cell the ESF is running in matches the cell specified
in the dev.yaml endpoints section.

Then you can make curl or oauthcurl requests directly to your endpoint:

*   Create a healthcare dataset _test-dataset_ (note the need to escape the `&`
    to prevent Bash from interpreting it):

```shell
curl -X POST --data \
'dataset.name="projects/${PROJECT:?}/locations/us-central1/datasets/test-dataset"' \
https://{{USERNAME}}-healthcare.sandbox.googleapis.com/v1alpha2/\
projects/${PROJECT:?}/locations/us-central1/datasets?\
access_token=${TOKEN:?}\&dataset_id=test-dataset
```

*   List stores in _test-dataset_:

```shell
curl https://{{USERNAME}}-healthcare.sandbox.googleapis.com/v1alpha2/projects/${PROJECT:?}/locations/us-central1/datasets/test-dataset?access_token=${TOKEN:?}
```

See [Sample requests](#sample-requests) section below for other requests.

## FHIR API Constraints

It's important to note that the FHIR API is only available over HTTP and not via
gRPC.

For any FHIR create/patch/update requests the spec requires that the
`Content-Type` header be set explicitly to
`application/fhir+json;charset=utf-8`. To do this with curl:

```shell
curl -H "Content-Type: application/fhir+json;charset=utf-8"
```

However, when patching FHIR resources, the `Content-Type` header should be set
to `application/json-patch+json`.

```shell
curl -H "Content-Type: application/json-patch+json"
```

## Sample requests

### Curl requests

List the healthcare datasets in a project:

```shell
curl https://autopush-healthcare.sandbox.googleapis.com/v1alpha2/projects/${PROJECT:?}/locations/us-central1/datasets?access_token=${TOKEN:?}
```

Create a healthcare dataset (note the need to escape the `&` to prevent Bash
from interpreting it):

```shell
curl -X POST --data 'dataset_id=test-dataset' https://autopush-healthcare.sandbox.googleapis.com/v1alpha2/projects/${PROJECT:?}/locations/us-central1/datasets?access_token=${TOKEN:?}
```

Update the healthcare dataset (note the need to escape the `&` to prevent Bash
from interpreting it):

```shell
curl -X PATCH --data '{"time_zone":"EST"}' https://autopush-healthcare.sandbox.googleapis.com/v1alpha2/projects/${PROJECT:?}/locations/us-central1/datasets/test-dataset?access_token=${TOKEN:?}\&update_mask=time_zone -H "Content-Type: application/json"
```

List the FHIR stores in a dataset:

```shell
curl https://autopush-healthcare.sandbox.googleapis.com/v1alpha2/projects/${PROJECT:?}/locations/us-central1/datasets/test-dataset/fhirStores?access_token=${TOKEN:?}
```

List the HL7v2 stores in a dataset:

```shell
curl https://autopush-healthcare.sandbox.googleapis.com/v1alpha2/projects/${PROJECT:?}/locations/us-central1/datasets/test-dataset/hl7V2Stores?access_token=${TOKEN:?}
```

Create an HL7v2 store:

```shell
curl -X POST --data 'hl7_v2_store_id=test-store' https://autopush-healthcare.sandbox.googleapis.com/v1alpha2/projects/${PROJECT:?}/locations/us-central1/datasets/test-dataset/hl7V2Stores?access_token=${TOKEN:?}
```

Publish an HL7v2 message:

```shell
echo "<msg_text>" | base64 -w 0 | cat <(echo -n "{\"message\": {\"data\" : \"") <(cat -) <(echo -n "\" }}") | curl -X POST -H "Content-Type: application/json" --data-binary @- https://autopush-healthcare.sandbox.googleapis.com/v1alpha2/projects/${PROJECT:?}/locations/us-central1/datasets/test-dataset/hl7V2Stores/test-store/messages?access_token=${TOKEN:?}
```

Or if you're looking for a pre-validated message: -H "Content-Type:
application/json"

```shell
curl -X POST -H "$AUTH"  https://autopush-healthcare.sandbox.googleapis.com/v1alpha2/projects/${PROJECT:?}/locations/us-central1/datasets/test-dataset/hl7V2Stores/test-store/messages?access_token=${TOKEN:?} -T cloud/healthcare/hl7/message_store/manual_tests/ingest_request.json
```

### Stubby requests

You can use the stubby CLI to make requests directly to the ESF endpoint. This
CLI offers a simpler interface and lets you list the operations available and
inspect their parameters.

For all our target aliases, see
[gslb.part.cfg](http://google3/googledata/production/gslbpush/sources/cloud-healthcare/gslb.part.cfg)

Example invocations (learning about an endpoint & its operations):

Common Endpoint and auth variable setting:

```shell
DEV_ENDPOINT="/abns/cloud-healthcare-eng/dev-us-central1.cloud-healthcare-eng-esf-${USER:0:8}:esf"
SANDMAN_ENDPOINT="/abns/cloud-healthcare-eng/esf-sandman:esf"
AUTOPUSH_ENDPOINT="/abns/cloud-healthcare-autopush/autopush-us-central1.cloud-healthcare-esf:esf"
STAGING_ENDPOINT="/abns/cloud-healthcare-staging/staging-us-central1.cloud-healthcare-esf:esf"
PROD_ENDPOINT="/abns/cloud-healthcare/prod-us-central1.cloud-healthcare-esf:esf"

ENDPOINT=$AUTOPUSH_ENDPOINT # Choose your endpoint

END_USER_CRED_FILE=/usr/local/google/tmp/end_user_cred.txt
/google/data/ro/projects/gaiamint/bin/get_mint --type=loas --text --endusercreds > $END_USER_CRED_FILE
```

Note: Access to `PROD_ENDPOINT` is locked down via a `auth.creds.useLOAS`
[allowlist configuration](http://google3/production/borg/cloud-healthcare/templates/rpcsp.borg?q=%27PROD%27)
to members of
[mdb:cloud-healthcare-prod-stubby-whitelist](http://mdb/cloud-healthcare-prod-stubby-whitelist).
If you are not allowlisted for prod access, you may see an error like this:
`CLIENT_ERROR RPC: Rejected by RpcSecurityPolicy: generic::unauthenticated:
Rejected by creds_policy (neither auth.creds.useLOAS nor
auth.creds.useNormalUserEUC granted): Permission 'auth.creds.useLOAS' not
granted to {{USERNAME}}@prod.google.com, because it satisfies none of the 1
rules granting that permission.`

List the services available

```shell
stubby ls ${ENDPOINT?}
```

List a service's operations

```shell
stubby ls ${ENDPOINT?} DatasetServiceV1Alpha2
```

See an operation's proto signature

```shell
stubby ls -l ${ENDPOINT?} DatasetServiceV1Alpha2.ListDatasets
```

See a specific type definition

```shell
stubby type ${ENDPOINT?} google.cloud.healthcare.v1alpha2.datasets.ListDatasetsRequest
```

Example invocation (list datasets):

```shell
stubby call ${ENDPOINT?} --rpc_creds_file ${END_USER_CRED_FILE?} DatasetServiceV1Alpha2.ListDatasets --proto2 'parent:"projects/'${PROJECT?}'/locations/us-central1"'
```

### gRPC requests

Similar to the Stubby CLI, you can use
[gRPC CLI](https://g3doc.corp.google.com/net/grpc/g3doc/grpc_cli.md) to send
gRPC calls to ESF endpoint.

#### Common Arguments

1.  GRPC_ENDPOINT - This should be the URL of the region or environment to send
    the request to. Examples: `dns:///us-central1-healthcare.googleapis.com:443`
    or `dns:///staging-healthcare.sandbox.googleapis.com:443`
1.  TOKEN - This is your access token.

Example for staging environment:

```
. google/cloud/healthcare/scripts/chc-api-funcs.sh
get-auth-token
endpoint staging
echo $TOKEN
echo $GRPC_ENDPOINT
```

Example for prod:

```
. google/cloud/healthcare/scripts/chc-api-funcs.sh
get-auth-token
endpoint prod
location europe-west2
echo $TOKEN
echo $GRPC_ENDPOINT
```

#### List DataSets

```
grpc_cli --noremotedb --proto_path=/google/src/head/depot/google3/ \
--channel_creds_type=ssl --call_creds="access_token=${TOKEN?}" \
--protofiles=google/cloud/healthcare/v1alpha2/dataset/service.proto \
call ${GRPC_ENDPOINT?} ListDatasets "parent: 'projects/cloud-healthcare-test/locations/us-central1'"
```

#### Get DataSet

```
grpc_cli --noremotedb --proto_path=/google/src/head/depot/google3/ \
--channel_creds_type=ssl --call_creds="access_token=${TOKEN?}" \
--protofiles=google/cloud/healthcare/v1alpha2/dataset/service.proto \
call ${GRPC_ENDPOINT?} GetDataset "name:'projects/cloud-healthcare-test/locations/us-central1/datasets/test-dataset'"
```

#### List FHIR Stores

```
grpc_cli --noremotedb --proto_path=/google/src/head/depot/google3/ \
--channel_creds_type=ssl --call_creds="access_token=${TOKEN?}" \
--protofiles=google/cloud/healthcare/v1alpha2/fhir/fhir_store_service.proto \
call ${GRPC_ENDPOINT?} ListFhirStores \
"parent:'projects/cloud-healthcare-test/locations/us-central1/datasets/test-dataset'"
```

#### Create a FHIR Store

```
grpc_cli --noremotedb --proto_path=/google/src/head/depot/google3/ \
--channel_creds_type=ssl --call_creds="access_token=${TOKEN?}" \
--protofiles=google/cloud/healthcare/v1alpha2/fhir/fhir_store_service.proto \
call ${GRPC_ENDPOINT?} CreateFhirStore \
"parent:'projects/cloud-healthcare-test/locations/us-central1/datasets/test-dataset' fhir_store_id:'test-store' fhir_store:{name:'projects/cloud-healthcare-test/datasets/locations/us-central1/test-dataset/fhirStores/test-store'}"
```

#### Get FHIR Store

```
grpc_cli --noremotedb --proto_path=/google/src/head/depot/google3/ \
--channel_creds_type=ssl --call_creds="access_token=${TOKEN?}" \
--protofiles=google/cloud/healthcare/v1alpha2/fhir/fhir_store_service.proto \
call ${GRPC_ENDPOINT?} GetFhirStore \
"name:'projects/cloud-healthcare-test/locations/us-central1/datasets/test-dataset/fhirStores/test-store'"
```

#### Get a FHIR Resource

```
grpc_cli --noremotedb --proto_path=/google/src/head/depot/google3/ \
--channel_creds_type=ssl --call_creds="access_token=${TOKEN?}" \
--protofiles=google/cloud/healthcare/v1alpha2/fhir/stu3/grpc/fhir_service.proto \
call ${GRPC_ENDPOINT?} GetResource \
"name:'projects/cloud-healthcare-test/locations/us-central1/datasets/test-dataset/fhirStores/test-store/resources'"
```

#### Create FHIR Resource

```
grpc_cli --noremotedb --proto_path=/google/src/head/depot/google3/ \
--channel_creds_type=ssl --call_creds="access_token=${TOKEN?}" \
--protofiles=google/cloud/healthcare/v1alpha2/fhir/stu3/grpc/fhir_service.proto \
call ${GRPC_ENDPOINT?} CreateResource \
"parent:'projects/cloud-healthcare-test/locations/us-central1/datasets/test-dataset/fhirStores/test-store' resource:{patient:{id:{value:'1'}}} type:'Patient'"
```
