# Performance Tests

[TOC]

## Test Location

All of our performance tests live in a versioned folder under
[//google/cloud/healthcare/test/](http://google3/google/cloud/healthcare/test/).
For example for v1alpha2 the tests are in
[//google/cloud/healthcare/test/v1alpha2/performance](http://google3/google/cloud/healthcare/test/v1alpha2/performance/).
They use the same underlying library to communicate with the ESF and backends as
the integration tests do. See the
[integration test documentation](integration_tests.md#test-suites) for more
details.

## Framework

Our performance tests use the go/perfgate framework to generate traffic patterns
and provide metric gathering and reporting.

## Running Tests

To run the performance tests use the
[`run_load_tests.sh`](http://google3/google/cloud/healthcare/test/v1alpha2/performance/run_load_test.sh)
script.

### Common Flags

*   `--modality=<e.g. dicom>`: Which set of backends you want to test. Only one
    modality may be selected at a time. These map to the test suites under the
    performance folder.
*   `--benchmark=<e.g. stow>`: Which specific test configuration you want to run
    for a given modality. These will differ based on the modality, for example
    DICOM has both `stow` and `qido` tests while HL7 has `readwrite` benchmarks.
*   `--env=(dev|staging)`: This determines which backends to use. `dev` will
    send the traffic to your personal [dev instance](api.md#dev-instances).
    Prefer using dev whenever possible. Typically traffic is only directed at
    staging as part of the release process or capacity planning.
*   `--duration=<seconds>`: How long the performance test should execute for.
*   `--max_qps=<qps>`: The maximum QPS total to send to the backends for the
    duration of the test.
*   `--min_qps=<qps>`: The minimum QPS total to send to the backends for the
    duration of the test. If only `max_qps` is specified, the performance
    testing framework will attempt to keep traffic steady at this level though
    it may slow down if the backends are overloaded. If both `max_qps` and
    `min_qps` are specified, the framework will start at `min_qps` and increase
    the QPS to `max_qps` over the duration of the test.
*   `--connection_type(grpc|http)`: The connection that test uses for connecting
    the APIs.

### Reading Output

At the end of a run you should see a run_chart_link in the test logs. This will
take you to the perfgate page and show you more detailed statistics for the run.

```
I0820 11:45:15.595589  170068 hl7v2_test.go:152] TestOutput:
run_chart_link: "https://perfgate.googleplex.com/run?run_key=6254797322190848"
run_key: "6254797322190848"
```

The charts will show data on latency and QPS (both target and actual), as well
as tables showing percentile data for every metric gathered.

### Longer Term Trends

To see data across multiple runs start from the
[cloud-healthcare-api perfgate page](https://perfgate.googleplex.com/project?name=cloud-healthcare-api).
Here you will see links for each of the benchmarks that we have configured. Once
you select a benchmark you will be able to see trends over time. You can select
different metrics to view for the graphs, in addition to filtering data by tags.
Tag filtering can be useful for isolating only certain runs, either by benchmark
type or by the environment it's running against.

NOTE: By default the links from the
[cloud-healthcare-api perfgate page](https://perfgate.googleplex.com/project?name=cloud-healthcare-api)
do not filter by tags, so they will plot a mix of dev and staging runs. To
monitor only staging over time, you can create a permalink with a tag filter
such as
[FHIR staging performance](https://perfgate.googleplex.com/benchmark?benchmark_key=4635859303792640&tag=env%3Dstaging).

Staging load test results can be found on
[Performance dashboard](http://g3doc/cloud/healthcare/g3doc/production/performance_dashboard.md)

## Creating a New Test Suite

Before creating a new test suite you should first familiarize yourself with
[how the perfgate framework works](http://g3doc/testing/performance/perfgate/g3doc/guide/how).

1.  Create a new directory for the test suite under
    [`google/cloud/healthcare/test/v1alpha2/performance/`](http://google3/google/cloud/healthcare/test/v1alpha2/performance).
1.  Branch
    [`benchmark.config.sample`](http://google3/google/cloud/healthcare/test/v1alpha2/performance/benchmark.config.sample)
    to your new directory as `benchmark.config`.
1.  Update the file, replacing `<MODALITY>` with the name of the modality that
    you're adding. Ignore `<BENCHMARK_KEY>` for now.
1.  Use the `perfgate` CLI tool to create the benchmark:

    ```shell
    $ /google/data/ro/teams/perfgate/perfgate create_benchmark path/to/your/benchmark/file/here
    ```

1.  Copy the benchmark key returned from the perfgate tool into your
    `benchmark.config`, replacing instances of `<BENCHMARK_KEY>`. Uncomment the
    `benchmark_key:` line at the same time.

1.  If you are branching based off of an existing modality make sure to change
    the benchmark key to the new one you just created.

1.  Add a sanity test ([example CL](http://cl/209422014)) to help prevent
    breakages as much as possible.

1.  Update
    [`run_load_test_lib.sh`](http://google3/google/cloud/healthcare/test/scripts/run_load_test_lib.sh)
    to include your new modality.

1.  Include your performance test in the staging release workflow by adding a
    new test to the tests list, make sure you increment the priority for your
    test to avoid parallel execution ([example CL](http://CL/210097133)):
    [`google/cloud/healthcare/test/guitar/template.ncl`](http://google3/google/cloud/healthcare/test/guitar/template.ncl)

1.  After your load test is run on staging, you should be able to see it on
    https://perfgate.googleplex.com/project?name=cloud-healthcare-api. Embed the
    performance diagram for your test suite on go/chc-perfgate-staging.

## Running load tests on Guitar

In addition to running load test from Rapid (see
[`Release instructions`](http://google3/cloud/healthcare/g3doc/process/release.md),
you can also run load tests on Guitar against staging or sandman environments.

*   To run on Guitar against staging:

NOTE install [guitar](http://go/guitar-cli) using `sudo apt install
google-guitar`.

```shell
guitar run \
    --cluster=healthcare-release-cluster \
    --workflow=//google/cloud/healthcare/test/guitar:healthcare_performance_staging
```

*   To run on Guitar release cluster against sandman with HTTP:

```shell
guitar run \
    --cluster=healthcare-release-cluster \
    --workflow=//google/cloud/healthcare/test/guitar:healthcare_performance_sandman
    --version=cl:${YOUR_CL_NUMBER?} \
```
