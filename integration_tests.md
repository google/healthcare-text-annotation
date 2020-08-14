This page: [go/chc-integ-tests](http://go/chc-integ-tests)

# Integration Tests

This documentation explains how our integration test environments are
structured, and what the building blocks are for new services and test cases.
Please refer to go/healthcare-integ-test-dd for the reasons behind the technical
decisions for our test infrastructure.

[TOC]

NOTE: Have a **flaky** or **broken** test? See go/hcls-buildcop.

## Test Suites

All our test suites live in
[//google/cloud/healthcare/test/v1alpha2/conformance](http://google3/google/cloud/healthcare/test/v1alpha2/conformance)
as required by the
[Cloud API Test Requirements and Guidelines](https://g3doc.corp.google.com/testing/cloud/g3doc/cloud_api_producer_testing_guidelines.md#requirements).
The plan is to have the same set of test cases running against different
environment setups and of different schedules. In such case, we only need to add
tests in one place. Over the development and deployment cycles, they will be
thoroughly validated.

All test cases talk to the Healthcare API via the same
[interface](http://google3/google/cloud/healthcare/test/v1alpha2/healthcare.go).
At first glance, this interface may look like a duplication of the backend
service proto definitions. But the real intention here is to make our test cases
agnostic to the underlying implementations of communication protocols. For
example, in some scenarios, the test cases access the API via GRPC while in
other cases, they access through HTTP endpoint. In the future, we may have
Stubby as well. The communication layer is initialized in the
[healthcare package](http://google3/google/cloud/healthcare/test/v1alpha2/BUILD)
according to the command-line flags. Test cases only need to concern themselves
with the request and response protobuf.

There are four lines of defense that can gate a bug from going to production,
namely, [Manual Local Tests](#manual_local_tests),
[Presubmit Integration Tests](#presubmit_integration_tests),
[Continuous Integration Tests](#continuous_integration_tests), and
[Deployment Tests](#deployment_tests). They are discussed one by one below.

## Manual Local Tests

### Robot account setup {#robot-account}

To run these tests, you need to have a robot account based on your LOAS, and the
robot account should be associated to your dev consumer project for testing on
dev environment. Your dev consumer project would be `{{USERNAME}}-test` if you
have followed the [developer setup](https://goto.google.com/chc-api-setup) and
created your own test project.

The robot account can use either your dev consumer project (likely
`{{USERNAME}}-test`) or shared `cloud-healthcare-test` project for testing. Once
the projects are setup, you can select the consumer project to use for testing
as described in [Run Tests from Blaze](#run-tests-from-blaze)

Note: `cloud-healthcare-test` is shared with other users in cloud healthcare
group, using it excessively on environments other than `dev` may hit the quota
limits and prevent your colleagues from using it with those environments.

1.  If you haven't already, create a personal robot account
    `{{USERNAME}}@system.gserviceaccount.com` with LOAS role as `{{USERNAME}}`
    at
    [robotmanager/](https://robotmanager.corp.google.com/manage_robot_accounts/createrobot).

1.  Set your PROJECT_ID, you can check the PROJECT_ID on
    [pantheon](https://pantheon.corp.google.com/home/dashboard?organizationId=433637338589&project={{USERNAME}}-test),
    but it should be {{USERNAME}}-test.

    ```shell
    export PROJECT_ID={{USERNAME}}-test
    ```

1.  Associate the service account with your dev project following
    go/hcls-g3doc/process/gcp_projects#service-accounts. Use the following
    variables for setup:

    ```shell
    MDB={{USERNAME}}
    PROJECT_NUMBER=$(gcloud projects describe ${PROJECT_ID?} | grep projectNumber | sed "s/.* '//;s/'//g")
    ```

1.  Add the service account as an owner of your dev consumer project and the
    shared `cloud-healthcare-test` project.

    ```shell
    gcloud projects add-iam-policy-binding ${PROJECT_ID?} --member serviceAccount:{{USERNAME}}@system.gserviceaccount.com --role roles/owner
    gcloud projects add-iam-policy-binding cloud-healthcare-test --member serviceAccount:{{USERNAME}}@system.gserviceaccount.com --role roles/owner
    ```

1.  Add the cloud-healthcare-eng robot service account as an owner of your dev
    consumer project.

    ```shell
    gcloud projects add-iam-policy-binding ${PROJECT_ID?} --member serviceAccount:cloud-healthcare-eng@system.gserviceaccount.com --role roles/owner
    ```

1.  Add {{USERNAME}}@prod.google.com as an Editor and Service Account Token
    Creator of your dev consumer project (likely `{{USERNAME}}-test`) and
    `cloud-healthcare-test` project.

    ```shell
    gcloud projects add-iam-policy-binding ${PROJECT_ID?} --member user:{{USERNAME}}@prod.google.com --role roles/editor
    gcloud projects add-iam-policy-binding ${PROJECT_ID?} --member user:{{USERNAME}}@prod.google.com --role roles/iam.serviceAccountTokenCreator
    gcloud projects add-iam-policy-binding cloud-healthcare-test --member user:{{USERNAME}}@prod.google.com --role roles/editor
    gcloud projects add-iam-policy-binding cloud-healthcare-test --member user:{{USERNAME}}@prod.google.com --role roles/iam.serviceAccountTokenCreator
    ```

If you notice that tests consistently fail with 403 Permission Denied, make sure
that your project has been granted access for the environment you are accessing,
[instructions here](http://go/hcls-adding-trusted-testers#step-3b-add-the-project).

### Test Your Own Service Config

If you want to test a service config (YAML file) change, you should create your
own version of Cloud Healthcare API backed by your selected backends.

```shell
google/cloud/healthcare/scripts/dev_inception_push.sh
```

Bring up your own backends, and then your ESF service. For example, for the
dataset service (for a full list of available services view the file in
[code search](https://source.corp.google.com/piper///depot/google3/google/cloud/healthcare/scripts/dev_reload_servers.sh)
):

```shell
google/cloud/healthcare/scripts/dev_reload_servers.sh --local_services=dataset-server
```

Now please refer to the [Run Tests from Blaze](#run_tests_from_blaze) section,
and launch integration tests locally against your own Cloud Healthcare API.

### Run Tests from Blaze

Our integration tests are triggered via `blaze test` and directed against
different environments. Take the DICOM integration tests for example, you can
run them against Autopush, Staging, and even your own Dev environments.

You can use test project `cloud-healthcare-test` for running integration test
against Autopush, Staging and Sandman environments.

For dev environment, you need to use your own project, e.g. {{USERNAME}}-test.
The following robot permissions setup is needed in your project:

1.  Add a new service account named `integration-test-user` with `Project Owner`
    permissions:

    ```shell
    gcloud iam service-accounts create integration-test-user \
      --display-name=integration-test-user --project=${PROJECT_ID?}
    gcloud projects add-iam-policy-binding ${PROJECT_ID?}  \
      --member=serviceAccount:integration-test-user@${PROJECT_ID?}.iam.gserviceaccount.com  \
      --role=roles/owner
    ```

1.  If you are running the FHIR API integration test with your own project,
    create a service account named `fhir-integration-test` with
    `healthcare.datasetAdmin` role for your project.

    ```shell
    gcloud iam service-accounts create fhir-integration-test --project ${PROJECT_ID?}
    gcloud projects add-iam-policy-binding ${PROJECT_ID?} \
     --member=serviceAccount:fhir-integration-test@${PROJECT_ID?}.iam.gserviceaccount.com \
     --role=roles/healthcare.datasetAdmin
    ```

    Alternatively you can setup your project using

    ```shell
    google/cloud/healthcare/test/scripts/setup-test-project.sh --project=<projectId> --env=<environment>
    ```

1.  Grant yourself `Service Account Token Creator` permissions if you don't have
    them already:

    ```shell
    gcloud projects add-iam-policy-binding ${PROJECT_ID?}  \
      --member=user:{{USERNAME}}@prod.google.com  \
      --role=roles/iam.serviceAccountTokenCreator
    ```

    Wait a minute for permissions to propagate.

Note: some tests have their own accounts, like `iamintegration_test` (which uses
many robot accounts) and `hl7integration_test`. If you want to run these tests
against your own project, you must grant permissions to these service accounts
in your project.

Launch the tests with the following command:

```shell
google/cloud/healthcare/test/v1alpha2/conformance/dev_run.sh
```

Note that if you want to run another version (for example, v1alpha2 or v1beta1),
replace v1alpha2 in the path above with the intended version name.

To run a specific test target, you can use the `target` flag which matches based
on part of the test name.

```shell
google/cloud/healthcare/test/v1alpha2/conformance/dev_run.sh --target=hl7v2
```

You can also run a subset of test methods using `test_filter` flag (which
accepts a regex):

```shell
google/cloud/healthcare/test/v1alpha2/conformance/dev_run.sh --target=dicomlro --test_filter=TestDicomStore_ImportError
```

Pass `runs_per_test` flag to run each test more than once:

```shell
google/cloud/healthcare/test/v1alpha2/conformance/dev_run.sh --target=fhirintegration_test --test_filter=TestCommon_InvalidHeaders --runs_per_test=10
```

To run with a specific consumer project, you can use the `project_id` flag. By
default `cloud-healthcare-test` will be used. E.g. to run with your dev consumer
project (likely `{{USERNAME}}-test`):

```shell
google/cloud/healthcare/test/v1alpha2/conformance/dev_run.sh --project_id={{USERNAME}}-test
```

To run a test with GRPC, you can use the `grpc` as connection type flag:

```shell
google/cloud/healthcare/test/v1alpha2/conformance/dev_run.sh --target=dataset --connection_type=grpc
```

To run a test connecting to the sandbox GFE via an HTTP connection, you can use
the `http` as connection type flag:

```shell
google/cloud/healthcare/test/v1alpha2/conformance/dev_run.sh --target=dataset --connection_type=http
```

It's also possible to specify which FHIR version to use (default value is
"Unspecified", which is STU3) using extra_test_args. For example, to run fhir
integration test using http and DSTU2:

```shell
google/cloud/healthcare/test/v1alpha2/conformance/dev_run.sh --target=fhirintegration_test --connection_type=http --extra_test_args=--fhir_version=dstu2
```

Tip: By default these tests are run against your personal dev instance. If you
need to run against autopush or staging you can add `--env=autopush` or
`--env=staging` respectively.

Warning: Please do not over-use `autopush` or `staging` as quota limits can
cause failures that impact others. Regular iterative testing while developing is
usually run against your own dev environment.

## Presubmit Integration Tests

When you send out your CL for review and before you hit submit, a
[Guitar presubmit integration workflow](https://cs.corp.google.com/piper///depot/google3/google/cloud/healthcare/test/guitar/BUILD?q=healthcare_integration_presubmit)
will be launched to validate the changes in your CL. It runs the same test cases
in
[//google/cloud/healthcare/test](https://cs.corp.google.com/piper///depot/google3/google/cloud/healthcare/test/?q=google/cloud/healthcare/test).

The Guitar workflow runs on our
[Guitar oneshot cluster](https://cs.corp.google.com/piper///depot/google3/google/cloud/healthcare/test/guitar/integration_oneshot.borg?dr).
It is pre-configured to execute on-demand integration tests. The result will be
reflected as a critique label "Guitar-healthcare_integration_presubmit". Upon
failure, the workflow result is set to ERROR, which blocks your CL submission.

### Skipping Tests

To skip presubmit integration tests, add tag
`SKIP_HEALTHCARE_GUITAR_APPROVED_BY_ONCALL_LDAP=<oncall LDAP, reason>` to your
CL.

*   Use the tag **only in urgent situations**, the usage needs to be explictily
    approved by either infra primary of secondary (preferred) oncall.

    *   Instead, check go/hcls-buildcop to ensure these failures are
        investigated and re-run your tests. This is a forcing function to get
        rid of flaky tests.

### Triggering on-demand

You can also manually trigger a Guitar on-demand integration workflow. You can
use it as a debug method since it will print out more detailed logging on your
own console.

NOTE install [guitar](http://go/guitar-cli) using `sudo apt install
google-guitar`.

```shell
# Add this to your ~/.bashrc to save yourself time in the future
alias chc_presubmit='guitar presubmit --cluster=healthcare-oneshot-cluster --workflow=//google/cloud/healthcare/test/guitar:healthcare_integration_presubmit'

chc_presubmit --c=${YOUR_CL_NUMBER?} [--email=reviewlog]
```

The tests are running against our
[Sandman environments](https://cs.corp.google.com/piper///depot/google3/production/borg/cloud-healthcare/sandman/healthcare.gcl),
which are brought up and torn down by Guitar on demand using a unique Borg
instance name every time. If you suspect that a failure is due to environment
issues, you can go to [Sigma/](https://sigma.corp.google.com) under user
cloud-healthcare-test to pull out the logs for inspection.

`--email=reviewlog` will apply/update the critique label on your CL.

`--version=citc:LOCAL` will send your most recent changes to the cluster.

`--test_targets_query` : a Blaze query expression to filter the test targets
actually run by Guitar.

If you only care about checking `//foo:bar_e2e_test`, do:
`--test_targets_query=//foo:bar_e2e_test` Fancier expressions work too. To run
every test target in your workflow that is also in the foo directory, use
`--test_targets_query=//foo/...` To run multiple specific test targets, use
`--test_targets_query='//foo:bar_test + //bar:baz_test + //baz:qux_test'`

#### To run the tests locally against a Rapid RC

See the
[release documentation](https://g3doc.corp.google.com/cloud/healthcare/g3doc/process/release.md#running-integration-tests-manually)
for this.

## Continuous Sandman inception push

We are using Guitar in [continuous build mode](http://go/guitar-cbuild) to run
inception push for SandMan environment. Guitar periodically applies the updates
in SandMan environment from HEAD. We use
[Guitar Oneshot Cluster](https://cs.corp.google.com/piper///depot/google3/production/borg/cloud-healthcare/guitar/healthcare_oneshot.borg)
set up for continuous integration. It executes a
[workflow](https://cs.corp.google.com/piper///depot/google3/google/cloud/healthcare/test/guitar/BUILD)
to bring up a sandman and runs inception push.

## Deployment Tests

We want to make sure the releases we deploy out are good. Therefore, we have
integration tests
[set up](https://cs.corp.google.com/piper///depot/google3/cloud/healthcare/release/integration_test.gcl)
in [Rapid](https://rapid.corp.google.com) to run against Autopush and Staging at
their deployments, and block their deployments upon test failure. Again, they
are running the same set of test cases. Integration test result for Autopush
will be sent to chc-wat+test@google.com when test status changes. Since we
deploy Autopush every two hours, we don't want to spam the mailing list unless
there is something different. But we do it for Staging at every deployment as
that happens less frequently and the stake is presumably higher that justifies
more diligent notifications.

## Integration Test Result Visualization

We use [Despeckle](https://despeckle.googleplex.com) to visualized our test
results. You can search by labels. The followings are saved searches:

*   [Presubmit](https://despeckle.googleplex.com/#spongeQuery=label:healthcare_integration_presubmit&env.mpm=&env.restarts=&env.codechange=&heuristics=NoRecentPass,AnyRecentPass,IgnorableStatusMarker,RhythmWorrier&maxResultCount=20&analysisSet=TEST_AND_TARGET&enableBrokenStatus=true&submit=true&failCountOnEnvChange=2&numRecentInvocations=3&stalenessThresholdInMins=120)

*   [Autopush](https://despeckle.googleplex.com/#spongeQuery=label:healthcare_integration_autopush&heuristics=NoRecentPass,AnyRecentPass,IgnorableStatusMarker,RhythmWorrier&maxResultCount=20&analysisSet=TEST_AND_TARGET&enableBrokenStatus=true&submit=true&failCountOnEnvChange=2&numRecentInvocations=3&stalenessThresholdInMins=120&env.mpm=&env.restarts=&env.codechange=)

*   [Staging](https://despeckle.googleplex.com/#spongeQuery=label:healthcare_integration_staging&heuristics=NoRecentPass,AnyRecentPass,IgnorableStatusMarker,RhythmWorrier&maxResultCount=20&analysisSet=TEST_AND_TARGET&enableBrokenStatus=true&submit=true&failCountOnEnvChange=2&numRecentInvocations=3&stalenessThresholdInMins=120&env.mpm=&env.restarts=&env.codechange=)

## Running Integration Tests With Different Projects (go/project-rental) {#project-rental}

Specify the `project_rental_pool` argument to have an integration test
automatically choose which project to run with. Once this argument is provided,
the test will ask go/project-rental to provide an unused project, and return it
when the test ends. For example, presubmit tests run with
`--project_rental_pool=healthcare_presubmit`.

For more info see our
g3doc/cloud/healthcare/g3doc/infrastructure/project_rental_service.md

## IAM Tests

Two differents IAM integration test setups have been put in place:

1.  Custom roles testing (**requirement for launching to Beta**).
1.  IAM integration tests written specifically for Cloud Healthcare
    (**requirement for launching to EAP**), uses [APISpec](#apispec-iam).

See the following sections for details on each.

### IAM Custom Role Testing

Custom roles testing via the go/iam-cli-test-doc testing framework (see
google3/cloud/testing/iam/json/surface/cloud_healthcare). Those tests are
monitored and run nightly by GCP EngProd. The results can be found in the
[all tests tab](http://testgrid/Partner%20Services%20Custom%20Project%20Roles#All%20Tests)
of go/iam-testgrid, and the list of permissions supporting custom roles can be
found in
[this dashboard](http://dashboards/_d1156785_94b1_4a2e_a6a2_4a060b6b060b?f=Service:in:healthcare&f=Support_Status:in:Supported,Pending%20testing,Not%20supported).

Permissions that are covered by custom role testing are marked with
`custom_roles_support_level: SUPPORTED` in the IAM service config.

When marking new permissions as supporting custom roles, we only need to test
them in prod before marking them as `custom_roles_support_level: SUPPORTED` in
all versions of `healthcare.service`.

## APISpec (go/apispec)

APISpec is a shared library used for writing integration tests for CHC that
requires the following pattern:

1.  Run each method of CHC API once for a particular environment/configuration.
1.  Perform assertions based on the error obtained or other results from
    executing them.

Think of it as a test registry, and its main objective is to avoid rewriting all
the cases for different integration tests, making it easier to add new ones and
enforce coverage for a particular test. It aims to cover:

*   IAM: pre-defined roles
*   Cloud Audit: ESF-generated logs
*   VPC-SC

Each of these tests have a slightly different execution requirement, but the
execution follows:

1.  Registry creates all the services, which only share a common dataset,
    nothing more.
1.  Test runner creates a test setup and clients that suit its use-case.
1.  For each service:
    1.  Setup is called (should allow every method spec to execute correctly).
    1.  Each MethodSpec is executed and validated/asserted.
    1.  TearDown is called,

[APISpec Service specs and method specs](http://google3/google/cloud/healthcare/test/apispec/specs.go)
defines basic types for the tests and are shared between all API versions. There
are [instructions and examples](#new-service) on how to add a new service.

### IAM integration test {#apispec-iam}

Due to the high number of requests (a lot of roles and methods combinations) and
the difference between environments, IAM is currently not enabled for presubmit,
but can be executed using the different environments, as other tests:

`google/cloud/healthcare/test/v1alpha2/conformance/dev_run.sh --target=iam
--env=autopush --extra_test_args=--service=dataset
--test_filter=TestSpecificRoles`

It will create a service instance for each role being tested and assert that a
particular role can successfully execute a method if it should or fail (with a
forbidden) if it shouldn't.

### Cloud Audit integration test {#apispec-cloud-audit}

Also not enabled for presubmit. Cloud audit runs every method twice - once with
Owner role (should always succeed) and once with None role (should always fail).
It will assert proper stackdriver logs are generated for both cases, by checking
the `Audit *AuditRequirements` field.

It performs a match (logic responsible for identifying that a wanted log was
generated) based on the method name and status (denied or not) and allows for
extra validations (e.g. contains particular fields) once the log was matched.

Due to stackdriver being a little bit flaky for dev, testing against
autopush/staging is usually the easier way of debugging it. Example with filters
for alpha:

`google/cloud/healthcare/test/v1alpha/conformance/dev_run.sh
--target=cloud_audit --env=autopush --extra_test_args=--service=fhir
-test_filter=TestRoleNone`

### Running APISpec tests locally

1.  Grant yourself `Service Account Token Creator` permissions if you don't have
    them already:

    ```shell
    gcloud projects add-iam-policy-binding cloud-healthcare-test  \
      --member=user:{{USERNAME}}@prod.google.com  \
      --role=roles/iam.serviceAccountTokenCreator
    ```

    Wait a minute for permissions to propagate.

1.  In your CITC client, run:

    ```shell
    blaze test google/cloud/healthcare/test/v1alpha2/conformance:iamintegration_V2_test \
     --test_output=streamed --notest_loasd \
     --test_arg=--healthcare_api_address=/abns/cloud-healthcare-autopush/autopush-us-central1.cloud-healthcare-esf:esf \
     --test_arg=--http_address=https://autopush-healthcare.sandbox.googleapis.com/
    ```

    Note: `dicomweb` tests are not supported by gRPC, so specify
    `--http_address`. See example command line above. The test can also be run
    in gRPC mode (with `--test_arg=--connection_type=grpc`), but it will skip
    that service resulting in incomplete coverage.

    Change the addresses to run against staging (or prod) instead of autopush,
    e.g.

    ```shell
    blaze test google/cloud/healthcare/test/v1alpha2/conformance:iamintegration_V2_test \
    --test_output=streamed --notest_loasd \
    --test_arg=--healthcare_api_address=/abns/cloud-healthcare-staging/staging-us-central1.cloud-healthcare-esf:esf \
    --test_arg=--http_address=https://staging-healthcare.sandbox.googleapis.com/
    ```

1.  In addition to the general integration test flags listed in earlier
    sections, the IAM tests have the `--services_to_test=a,b,c` flag to speed up
    execution and/or focus on problems related to a subset of services. Service
    names are
    [here](http://google3/google/cloud/healthcare/test/v1alpha2/conformance/iamintegration_v2_test.go?q="allServiceNames+%3D").

1.  Use `--test_filter=<TestNameForSpecificRole>` to subset which roles are run.
    See
    google3/google/cloud/healthcare/test/v1alpha2/conformance/iamintegration_v2_test.go
    for a list.

### Adding methods to APISpec tests {#new-method}

Example CL: cl/293660740

If your service already exists and contains an APISpec service spec
([alpha](http://google3/google/cloud/healthcare/test/v1alpha2/conformance/apispec/services/),
[beta](http://google3/google/cloud/healthcare/test/v1beta1/conformance/apispec/services/),
[v1](http://google3/google/cloud/healthcare/test/v1/conformance/apispec/services/)):

1.  If not yet there, add new methods to the
    [Healthcare Integration Test Service Interface](http://google3/google/cloud/healthcare/test/v1alpha2/healthcare.go?q="Service+interface").
    That will also requires adding implementations of new methods in the
    google3/google/cloud/healthcare/test/v1alpha2/grpc.go,
    google3/google/cloud/healthcare/test/v1alpha2/http.go test service
    implementations.

1.  If your method requires some pre-existing state, make sure the service spec
    SetUp method adds what it needs - don’t assume any order of execution for
    the MethodSpecs when doing so, Setup uses an admin client and MethodSpecs
    are independent. If you method adds anything outside the shared dataset
    (which gets deleted afterwards), consider adding proper clean up in the
    TearDown function.

1.  Add a new function that returns an MethodSpec, which includes a proper name,
    an Exec method (what will actually be executed). Other fields are used by
    the tests and should be fulfilled based on what the method does/requires
    (e.g. roles should contain all the predefined roles relevant to your service
    that should be able to execute it, it’s used by IAM testing).

1.  Make sure the new function is correctly returned under MethodSpecs().

Note: If your method is an LRO, it is recommended to avoid triggering/initiating
a real one, perform a request that will trigger an Invalid Argument instead.
This significantly improves the test perofmance. On the other hand,
[VPC-SC tests](http://go/chc-vpcsc-test) expect complete LRO executions.
Consider using
[AllowInvalidArgumentLROs](http://google3/google/cloud/healthcare/test/apispec/specs.go?l=89&cl=315563078)
in the method specs for dealing with this variation.

### Adding a new role to APISpec tests {#new-role}

1.  On a clean CITC client, follow instructions for
    [running APISpec tests locally](#running-apispec-tests-locally) and work out
    any setup/connection problems until the tests pass.

1.  Make sure your new roles have been pushed the new roll out to the `prod`
    environment in Cloud Resource Manager. Rollout status for CLs containing new
    [Healthcare roles](service_accounts.md#curated-iam-role) can be checked at
    go/iam-config-rollout-single/<cl_number>.

1.  Add the new role(s) to
    [roles.go](http://google3/google/cloud/healthcare/test/apispec/roles.go) by
    both adding a constant and returning it under roles.ServiceRoles() method.
    Also add the new role under every service.ServiceRoles() which such role
    should be tested (usually one modality and the dataset service). Add the new
    role to every MethodSpec that should be successfully executed by the role
    (omitting = forbidden).

1.  In your CITC client, run the following substituting `<new_role_name>`:

    ```shell
    ROLE=roles/healthcare.<new_role_name>
    PROJECT_ID=cloud-healthcare-test
    SERVICE_USER=`echo ${ROLE?} | tr '[:upper:]' '[:lower:]' | sed -Er 's/roles\///' | sed -E 's/\./-/g'`
    echo "Setup service account: ${SERVICE_USER?}"
    gcloud iam service-accounts create ${SERVICE_USER?} --display-name=${SERVICE_USER?} --project=${PROJECT_ID?}
    gcloud projects add-iam-policy-binding ${PROJECT_ID?} \
     --member=serviceAccount:${SERVICE_USER?}@${PROJECT_ID?}.iam.gserviceaccount.com \
     --role=${ROLE?}
    ```

1.  Wait 80 seconds to give time for the previous step's permissions to
    propagate.

1.  If you haven't run integration tests locally before, then grant yourself
    `Service Account Token Creator` permissions:

    ```shell
    gcloud projects add-iam-policy-binding cloud-healthcare-test  \
      --member=user:{{USERNAME}}@prod.google.com  \
      --role=roles/iam.serviceAccountTokenCreator
    ```

    Wait a minute for permissions to propagate.

1.  Now run your new test, substituting `<new_test_name>`:

    ```shell
    blaze test google/cloud/healthcare/test/v1alpha2/conformance:iamintegration_V2_test \
     --test_output=streamed --notest_loasd \
     --test_arg=--healthcare_api_address=/abns/cloud-healthcare-autopush/autopush-us-central1.cloud-healthcare-esf:esf \
     --test_arg=--http_address=https://autopush-healthcare.sandbox.googleapis.com/ \
     --test_filter=<new_test_name>
    ```

    Iterate on this step until your test passes. If you get strange setup
    errors, check that your role name in the test is exactly correct and that
    your new role is available in `staging`.

1.  If the new test passes locally, then Guitar tests should pass as well.
    Otherwise you will need to follow the
    [One Shot instructions](#presubmit-integration-tests).

1.  For running integration tests in CBuild, you also have to update
    [setup-test-project-iam-lib.sh](http://cs/google3/google/cloud/healthcare/test/scripts/setup-test-project-iam-lib.sh)
    and add the new role under "Add service accounts for iam_integration_test".

1.  [File a bug](https://b.corp.google.com/issues/new?component=622815&template=1385853)
    for the secondary oncall to run the setup script.

1.  Submit your CL only after the script run is complete.

### Adding a new service to APISpec tests {#new-service}

Example CLs: cl/295990425

1.  If you are adding a new service, ApiSpec uses the healthcare client and
    assumes your new service already has integration tests. If that’s not the
    case, adding the methods for your service under the healthcare client is
    necessary, as described by [adding methods to APISpec tests](#new-method).

1.  Properly [add the roles](#new-role) for your service. Notice that the test
    will use Guitar/CBuild, so make sure your roles are added to
    [setup-test-project-iam-lib.sh](http://cs/google3/google/cloud/healthcare/test/scripts/setup-test-project-iam-lib.sh)

1.  Implement
    [APISpec service spec interface](http://google3/google/cloud/healthcare/test/apispec/specs.go),
    being very conservative with what NewServiceSpec and Setup requires. Assume
    multiple instances of NewServiceSpec might be running at the same time, so
    avoid using global resources. There are plenty of examples already
    implemented.

1.  Try to use the shared dataset (under specs.Config) if possible, which saves
    you the trouble of cleaning it up and makes the test faster/less flaky.

1.  Add a TearDown method. Tests should fail if any error is returned, please
    don’t omit them. Teardown can assume that the initial dataset will be
    deleted later but should also take into consideration other resources
    created during MethodSpecs execution (e.g. if a policy was successfully
    created, it should be cleaned up later during teardown).

### Setup APISpec tests on a new project

1.  Enable the resource manager API (staging & prod).

1.  Enable all healthcare APIs you wish to test against.

1.  Enable the PubSub API and give "Pub/Sub Publisher" role to:

    *   cloud-healthcare-eng@system.gserviceaccount.com
    *   cloud-healthcare@system.gserviceaccount.com
    *   cloud-healthcare-test@prod.google.com
    *   cloud-healthcare-test@system.gserviceaccount.com

1.  Give GCS admin roles to the same accounts as listed in the previous step.

1.  Add a PubSub topic named `iam_test_topic`.

1.  Add a GCS bucket named `cloud-healthcare-integration-iam-test`.

1.  Create a Project Service Account named `admin-iam-test` and give it
    `Project > Owner` permissions.

1.  Grant yourself `Service Account Token Creator` permissions if you don't have
    them already:

    ```shell
    gcloud projects add-iam-policy-binding ${PROJECT_ID?}  \
      --member=user:{{USERNAME}}@prod.google.com  \
      --role=roles/iam.serviceAccountTokenCreator
    ```

    Wait a minute for permissions to propagate.

1.  Follow the steps for
    [Adding a new role to IAM tests](#adding-a-new-role-to-iam-tests) once per
    existing role defined in iamintegration_v2_test.go. It would be faster to
    batch the service account creation step for each role before moving on, then
    test all roles at once by removing the `--test_filter` flag.
