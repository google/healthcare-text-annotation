# VPC-SC Integration Automated Test

go/chc-vpcsc-test

<!--*
# Document freshness: For more information, see go/fresh-source.
freshness: { owner: 'vajih' reviewed: '2020-07-30' }
*-->

[TOC]

## Introduction

[VPC-SC](http://cloud/vpc-service-controls/) stands for VPC service control. It
is a way to mitigate data exfiltration from VPCs through GCP APIs. It allows
organization managers to create a security perimeter that consists of:

1.  which projects are protected
2.  which GCP APIs should respect the perimeter
3.  which requests are considered to be inside the perimeter.

[This background doc](http://doc/1FRg4q5kCbyviyXicdZqhogEo2Qxy5632tMmskV4f9bI#heading=h.jzboz77c19x6)
has more information on how data exfiltration can happen, and
[this questionnaire](http://doc/1Fe3auoV3mMX7Fx1Bavm94ThJykSA-1I_HSjeO7kisxQ)
describes specifically how the CHC API can be used to exfiltrate data. A chronic
automated testing help us detecting regressions during releases and monitoring
prod APIs.

### What should be tested?

VPC-SC protects GCP resources by guarding API requests. Each API request can
specify one to multiple GCP resources. It is required to demonstrate that the
requests to CHC APIs and resources outside of a VPC-SC service perimeter are
properly blocked. Thus, we have two types of tests showing failure in:

1.  Calling CHC API against a project of a perimeter from inside a perimeter,
1.  Calling a CHC API inside a perimeter from the same perimeter but the request
    has reference(s) to resource(s) outside of the perimeter.

To avoid spurious test results and make sure the VPC-SC service perimeters do
not block permissible requests, we also need tests to show calling a CHC API
inside a perimeter from the same perimeter is successful.

## How to test?

To verify VPC-SC protects resources, we send requests from within a service
perimeter and verify its response. We have set up a secure VPC-SC testing
environment where the tests must be run. The tests cannot be run on borg, as
they must be run on a GCE VM inside of the service perimeter. In more detail:

1.  We set up a security perimeter where CHC API, pubsub API, GCS API, BigQuery
    API are all restricted. Meaning that: for a request against these APIs,
    VPC-SC should decline violating requests that cross service perimeters.
    _E.g._, try to access an external project/resource from within the service
    perimeter.

1.  We run tests from a GCE VM within the perimeter. The test 1) (negative case)
    sends violating requests and expect VPC-SC errors, and 2) (positive case)
    sends compliant requests and expect no errors

For each resource in an API method, we should have one positive and at least one
negative case for it. The following sections demonstrate how CHC APIs can be
tested.

### Extending test coverage

The VPC-SC compliance tests are placed in
`/google/cloud/healthcare/test/${CHC_VERSION}/conformance/vpc/vpc_e2e` per CHC
API versions.

We test the aforementioned scenarios using three tests developed in Go:

1.  **TestVPCSCDeniedExternalProject** ensures that accessing outside of a
    perimeter is forbidden. For testing, we call CHC APIs on a project outside
    of the security perimeter. The call must be blocked with a Permission Denied
    error and relevant error message.
1.  **TestVPCSCDeniedExternalResources** shows data insides a perimeter cannot
    be exfiltrated to outside resources through CHC APIs. The test calls CHC
    APIs within a service perimeter but the source or destination resource are
    in another project, outside of the perimeter. Not all CHC APIs need to be
    tested in this way. For example, the APIs offering import or export services
    to GCS buckets or BQ datasets fall in this category.
1.  **TestVPCSCAllowedInternalProject** makes sure that a call to CHC APIs
    within a service perimeter is successful and not blocked by VPC-SC.

In order to test all CHC API surfaces, we use
[API-Spec framework](http://go/apispec) to specify the APIs and iterate them.
The framework ensures each APIs pre-condition, _e.g.,_ a dataset existence, is
already met. After the test, the framework ensures the testing environment is
properly cleaned up.

NOTE: The APIs under test must have been already specified in API-Spec before
adding VPC-SC tests for them. If not, please follow the instructions to add a
new
[service](https://g3doc.corp.google.com/cloud/healthcare/g3doc/api/integration_tests.md#new-service)
or
[method](https://g3doc.corp.google.com/cloud/healthcare/g3doc/api/integration_tests.md#new-method).

NOTE: It is important to successfully call APIs for
TestVPCSCAllowedInternalProject. Unlike IAM tests, the LRO job cannot be
bypassed by _Invalid Arguments_.

Once the API and Method specifications are added, the following adjustments are
needed:

1.  Allow the test service under the test by adding it to the
    [active service set](http://google3/google/cloud/healthcare/test/v1/conformance/vpc/e2e_test.go?l=46&cl=312285974).
1.  For method specs that have access to external resources, it is required to
    enable
    [`AdditionalResourceAccess`](http://google3/google/cloud/healthcare/test/apispec/specs.go?l=37&cl=312285974&rcl=314121725)
    flag.
1.  For testing methods accessing external resources, one can add multiple
    method specs. For example,`exportDicomToGCSMethodSpec` and
    `exportDicomToBQMethodSpec` are two variations of DICOM resource export to a
    GCS bucket or BigQuery dataset.
1.  If IAM tests need a special permission, please add it the
    [setup script](http://google3/google/cloud/healthcare/test/scripts/setup-vpc-project.sh?l=56&rcl=315730821)
    and ask EngProd team, vajih@, to apply the changes. See
    [cl/315730821](http://cl/315730821) as an example.

WARNING: Unlike other tests based on API-Spec, such as IAM tests, all call must
be successfully done. The LROs should not be bypassed.

### Test data

The testing data is available on two places: local disk and GCS buckets. The
testing files in
`[google/cloud/healthcare/test/apispec/testdata/](http://cs/google/cloud/healthcare/test/apispec/testdata/)`
is copied to VM's disk per each test trigger. See
[this](http://google3/google/cloud/healthcare/test/v1alpha2/conformance/apispec/services/dicomweb.go?l=75&cl=314463498)
example for reading testing data from local disk.

The GCE testing data should be first placed in
[gs://healthcare-no-vpc-data](https://console.cloud.google.com/storage/browser/healthcare-no-vpc-data).
Please free to request a short-term access to the bucket from EngProd team to
add new test data. The testing infrastructure copies the data to each perimeter.
Please add the filename to EnvironmentGate
[configuration](http://google3/google/cloud/healthcare/test/vpcsc/static.yaml?l=84-89&rcl=315348814).

### Infrastructure to setup a testing environment

The testing environment comprises GCP organizations, policies, and resources to
emulate customers' setup. We used [EnvironmentGate](http://go/environmentgate)
to setup and maintain the testing environment. The tool manages all network
setups, GCP projects, access policies for testing VPC-SC. Section
[Testing Infrastructure](#infra) provides more details.

In brief, the tests on Piper are ported to a GCE machine to run the test inside
a perimeter. Setting up a perimeter is time consuming and needs manual approval,
so we set them up once and reuse them per test. In order to run tests on a
specific environment, one should trigger Kokoro's job to perform all
aforementioned steps.

To trigger a test, select
[`prod:cloud_healthcare/vpcsc/gcp_ubuntu/continuous`](http://fusion/projectanalysis/summary/KOKORO/prod%3Acloud_healthcare%2Fvpcsc%2Fgcp_ubuntu%2Fcontinuous)
Kokoro job and hit in `+ TRIGGER BUILD`:

![](http://screen/jRPffSRcAtK.png)

Then, choose the CL contains latest changes of the tests and CHC environment,
_e.g.,_ Autopush, Staging, or Prod, and hit on `TRIGGER`.

![](http://screen/QYPkTwtywZ6.png)

### Testing at presubmit

Provided a CL contains changes related to VPC-SC tests, piper triggers the tests
to ensure the CL does not cause any regression. It tests all versions, _i.e.,_
Alpha, Beta, and GA, against Autopush. While developing VPC-SC tests, one can
leverage the presubmit tests to ensure the new tests complies with expected
results per each test.

TODO: Trigger from command line

### Run positive cases

On a dev machine, one can run **TestVPCSCAllowedInternalProject**. This ensures
the VPC-SC tests are well integrated with new CHC API's spec. Executed the test
using:

```sh
$ PROJECT_ID=?
$ GOOGLE_APPLICATION_CREDENTIALS=<PATH_TO...>
$ SERVICE=?
$ blaze test //google/cloud/healthcare/test/v1beta1/conformance/vpc:vpc_e2e -- \
    --internal_project=${PROJECT_ID?} \
    --healthcare_api_address=https://healthcare.googleapis.com
    --test_filter=TestVPCSCAllowedInternalProject/${SERVICE?}
```

The GCP project that is created for integration tests.

## Testing Infrastructure {#infra}

The testing infrastructure provides a testing environment to emulate customers'
use of CHC APIs inside VPC-SC perimeters. The environment is based on a distinct
test organization, `hcls-vpcsc-org.joonix.net` , already established for testing
VPC-SC organization policies.

### GCP Projects

There are three groups of GCP projects defined in the testing environment. One
is the EnvironmentGate master project, a.k.a. Workflow Master Project.
EnvironmentGate uses this project to set up the testing infrastructure and
manage other projects. We also use this project for hosting Kokoro's dedicated
pool. EnvironmentGate creates and manages a pool of Deployments under Test (DuT)
projects. These projects are inside service perimeters and run the tests. The
last type is a standalone project emulating a project outside of all service
perimeters. All these projects are in the testing organization.

### Resources

The testing resources are distributed on DuT projects as well as
`healthcare-no-vpc` project. The latter project is outside of existing service
perimeters for testing VPC-SC blocked calls.

![](https://screenshot.googleplex.com/uk9piZMSdhT.png)

Note that:

1.  All the changes in DuT projects are managed by healthcare-vpc-test project,
    WMP. We use [EnvironmentGate](http://go/environmentgate) and Deployment
    Manager templates to specify and manage the modifications on DuT and WMP
    projects.

1.  Updating a DuT, the `healthcare-no-vpc-data` bucket is synced with the
    counterpart buckets in each DuT. The files in these bucket are often used
    for testing import operations or per-population.

1.  The `$PROJECT_NAME-export` should be used for testing operation writing on a
    GCS bucket. The files on this bucket are wiped out after a day.

1.  APIs executed on a DuT's VM only have access to the resources on that
    project.

### Test execution flow

In order to run tests on a VM, we use Kokoro to orchestrate all the
interactions. Depicted in the following diagram, a Kokoro job interacts with
different Google platforms and GCP machines to fetch and build the tests and
store the logs.

![](https://screenshot.googleplex.com/ANkp4XOatDH.png)

1.  After being triggered, Kokoro builds a binary of tests stored on piper.
1.  Kokoro boots a VM from a dedicated pool available on `healthcare-vpc-test`
    project.
1.  Kokoro triggers EnvironmentGate to get a DuT project.
1.  Kokoro encapsulates the testing binary in a docker image and pushes it to
    the DuT project GCR repository.
1.  Kokoro SSH to the DuT's VM, `private-net-internal-instances` and run the
    docker image for testing.
1.  Kokoro fetches the logs from the DuT's VM in order to place them on Placer
    to be consumed by Sponge.

## Administration

### Test Execution Settings

1.  [Test Timeout](http://google3/google/cloud/healthcare/test/vpcsc/run_tests_remotely.sh?l=31&cl=312744484):
    The time a test can be executed. A test is defined per API
    environment/version.
1.  [Parallel tests](http://google3/google/cloud/healthcare/test/vpcsc/run_tests_remotely.sh?l=29&cl=312744484):
    Number of tests executed in parallel. A test is defined per API
    environment/version.
1.  Testing Environment: What release environments should the test be executed
    against. The configuration is a comma-separated list of environment names
    assigned to `CHC_API_ENVIRONMENTS` environment variable.

### Base Docker Image

In order to push the base docker image to `healthcare-vpc-test` GCR repo, run
the following script:

```sh
$ google/cloud/healthcare/test/vpcsc/docker/base_image.sh
```

### Updating DuTs

Run the following command to update all the DuT projects:

```sh
$ alias egm=/google/bin/releases/cloud-environment-manager/v1/egm
$ egm update --config=google/cloud/healthcare/test/vpcsc/environment.textproto --abandon
```

There are some configurations, such as assigning Roles and API enabling,
specified beyond EnvironmentGate specs. Please consider running the following
script per environment:

```sh
bash google/cloud/healthcare/test/scripts/setup-all-vpc-projects.sh
   --env=prod \
   --dut_prefix=healthcare--1585785581-
   --pool_size=16
```

Note that dut_prefix and pool_size might varry if the DuT pool is reconstructed.

### Kokoro Jobs

Job Name                                                                                                             | Environment | Versions        | Trigger
-------------------------------------------------------------------------------------------------------------------- | ----------- | --------------- | -------
[autopush](http://fusion/projectanalysis/summary/KOKORO/prod%3Acloud_healthcare%2Fvpcsc%2Fgcp_ubuntu%2Fautopush)     | Autopush    | GA\|Beta\|Alpha | Rapid
[staging](http://fusion/projectanalysis/summary/KOKORO/prod%3Acloud_healthcare%2Fvpcsc%2Fgcp_ubuntu%2Fstaging)       | Staging     | GA\|Beta\|Alpha | Rapid
[prod](http://fusion/projectanalysis/summary/KOKORO/prod%3Acloud_healthcare%2Fvpcsc%2Fgcp_ubuntu%2Fprod)             | Prod        | GA\|Beta\|Alpha | Rapid
[continuous](http://fusion/projectanalysis/summary/KOKORO/prod%3Acloud_healthcare%2Fvpcsc%2Fgcp_ubuntu%2Fcontinuous) | Prod        | GA\|Beta\|Alpha | Kokoro
[presubmit](http://fusion/projectanalysis/summary/KOKORO/prod%3Acloud_healthcare%2Fvpcsc%2Fgcp_ubuntu%2Fpresubmit)   | Autopush    | GA\|Beta\|Alpha | Piper
