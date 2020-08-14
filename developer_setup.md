# Getting Started on Healthcare APIs

go/chc-api-setup

[TOC]

<!--*
# Document freshness: For more information, see go/fresh-source.
freshness: { owner: 'jonesmi' reviewed: '2020-06-08' }
*-->

This quickstart guide is aimed at getting your dev environment set up and
working. Everything you need to finish this tutorial should be included here.

Start with go/health-cloud-noogler if you haven't already.

## Join Necessary Groups {.numbered}

First add yourself to
[cloud-healthcare-eng-team](https://ganpati.corp.google.com/#Group_Info?name=cloud-healthcare-eng-team)
to avoid permission issues with this guide.

## Bring Up HCLS Backend {.numbered}

The HCLS services internal backend is made up of Stubby servers, Spanner,
Blobstore, etc. They are hosted in Google Prod via borg jobs. Let's start up
your own dev instances of our borg job services. This will allow you to test
unsubmitted changes.

1.  Open a terminal and run `prodaccess` to get access to the google production
    network. You can use `prodcertstatus` to check if it's expired.

1.  Create a new citc workspace. (you can name it anything you want)

    ```shell
    g4d -f chc-api-setup
    ```

1.  Build & bring up your own personal dev instances of all HCLS backend
    services in Borg using this script. This will also blaze build all needed
    binaries and rollup any unsubmitted changes made in your citc workspace.

    ```shell
    google/cloud/healthcare/scripts/dev_reload_servers.sh --local_services=all
    ```

    NOTE: when completed, this script outputs Sigma links to the Borg jobs
    started.

## Configuring the API service {.numbered}

We use **One Platform** (OP) for providing external access to our APIs via
**Google Cloud Platform** (GCP), since it's the only approved way to build such
APIs. OP requires a service config, which we prepopulate for you in
[dev.yaml](http://cs/google3/google/cloud/healthcare/dev.yaml), that defines the
***Producer project*** in GCP that we own and manage, and the API endpoints that
will serve as that producer project. We use the same *cloud-healthcare-api-dev*
producer_project_id across all our personal dev endpoints.

NOTE: if you are an intern, please get your mentor or a teammate to help with
the inception push.

1.  Let's bring up your own ***{{USERNAME}}*** dev API endpoint in One Platform
    by pushing your service configuration to Inception. This only needs to be
    done once each time you set up or change your service configs
    (cs/google3/google/cloud/healthcare/dev.yaml).

    ```shell
    google/cloud/healthcare/scripts/dev_inception_push.sh
    ```

    If this is the first time a dev environment is pushed, ask
    [Secondary Oncall](http://go/hcls-oncall) to help you run this script:

    ```shell
    google/cloud/healthcare/scripts/dev_inception_push.sh --user=<ldap of the noogler>
    ```

    NOTE: [Inception](http://go/inception), which is known externally as
    [*Service Management API*](https://cloud.google.com/service-management), is
    a key component of One Platform. It manages the lifecycle and configuration
    of OP services, such as creating, updating, or deleting services. Inception
    is also responsible for pushing service configuration information to other
    infrastructure services, such as Chemist and QuotaServer, Cloud Launcher and
    others.

    TIP: Handling inception push failures due to quota limit errors. If
    inception pushes are consistently failing with errors similar to `WARNING:
    toplevel: Value LOW in limit AnnotationStoreOpsPerMinutePerProjectPerRegion
    can't be set below 100` (full sample
    [here](https://paste.googleplex.com/4737674571153408)), quota limits can be
    removed locally from
    [big_red_button_quota.yaml](https://source.corp.google.com/piper///depot/google3/google/cloud/healthcare/big_red_button_quota.yaml;rcl=320602091;l=457)
    ([sample cl](http://cl/321842186)). Afterwards, rerun the
    dev_inception_push.sh script ~5 times until the push succeeds.

1.  Check the status of your Inception push

    ```shell
    /google/data/ro/teams/oneplatform/launch_checker --service="{{USERNAME}}-healthcare.sandbox.googleapis.com"
    ```

    Your API endpoint should now be registered in OP as
    `{{USERNAME}}-healthcare.sandbox.googleapis.com`.

1.  Bring up your own dev [ESF](/google/g3doc/oneplatform/esf.md) borg job,
    which will host the OP API Frontend that proxies traffic from GFE to our
    internal stubby services.

    ```shell
    borgcfg production/borg/cloud-healthcare/dev/healthcare_dev.borg up healthcare.esf --vars=cell=ik
    ```

    NOTE: The borg cell `ik` is recommended for test and experimental.

1.  Check that your ***{{USERNAME}}*** dev ESF is working. You can use the
    stubby CLI to make requests directly to ESF endpoints using this simple
    interface and lets you list the operations available and inspect their
    parameters.

    ```shell
    export DEV_ENDPOINT="/abns/cloud-healthcare-eng/dev-us-central1.cloud-healthcare-eng-esf-${USER:0:8}:esf"
    stubby ls ${DEV_ENDPOINT?}
    ```

## Create your own Consumer project {.numbered}

Now it's time to actually set up an example client project in GCP that will
consume your *{{USERNAME}}* dev API endpoint. You only need to do this once, it
will be your personal project for testing.

1.  Prepare `gcloud`

    ```shell
    alias gcloud=/google/data/ro/teams/cloud-sdk/gcloud
    ```

1.  If you haven't logged in then (Do not use the service account option):

    ```shell
    gcloud auth login
    ```

1.  If you don't already have a test consumer project then create one:

    ```shell
    PROJECT={{USERNAME}}-test
    gcloud config set account {{USERNAME}}@google.com
    gcloud projects create --folder=55595711340 "${PROJECT:?}"
    ```

    TIP: It takes a couple minutes for your project to be ready

    NOTE: The last command might fail if you didn't previously accept the
    `LOAS-{{USERNAME}}` project _Terms of Service_ in the
    [Cloud Platform Console](https://pantheon.corp.google.com/home/dashboard?project=loas-{{USERNAME}}).

## Enable dev API in your Consumer project {.numbered}

```shell
gcloud services enable {{USERNAME}}-healthcare.sandbox.googleapis.com --project=${PROJECT:?}
```

You may need to enable billing for this step. You can enable billing through
[Google Cloud Dashboard](https://pantheon.corp.google.com/billing/linkedaccount).
Use `Billing Account for Cloud Healthcare`.

You can verify your API is enabled by checking the API library page for
[Cloud Healthcare API ({{USERNAME}})](https://pantheon.corp.google.com/apis/library/{{USERNAME}}-healthcare.sandbox.googleapis.com?project={{USERNAME}}-test&organizationId=433637338589)
for your `{{USERNAME}}-test` project in pantheon.

You can now monitor API request stats from your
[dashboard](https://pantheon.corp.google.com/apis/api/).

## Use your dev API endpoint {.numbered}

We can now call the API using your consumer project and verify that everything
is working end-to-end.

1.  Set gcloud default account and project.

    ```shell
    gcloud config set account {{USERNAME}}@google.com
    gcloud config set project ${PROJECT:?}
    ```

1.  Acquire new user credentials to use. It will prompt you to go to a URL to
    authenticate your google account.

    ```shell
    gcloud auth application-default login
    ```

1.  Get the access token.

    ```shell
    export TOKEN=`gcloud auth application-default print-access-token`
    echo $TOKEN
    ```

1.  Make a GET request to your API endpoint to verify everything works
    end-to-end

    ```shell
    curl https://{{USERNAME}}-healthcare.sandbox.googleapis.com/v1alpha2/projects/${PROJECT:?}/locations/us-central1/datasets?access_token=${TOKEN:?}
    ```

    Note: if you are getting a 500 Internal Server Error from this curl command,
    try running `export GOOGLE_CLOUD_PROJECT=${PROJECT:?}`

1.  Check out
    [chc-api-funcs.sh](https://cs.corp.google.com/piper///depot/google3/google/cloud/healthcare/scripts/chc-api-funcs.sh)
    which contains a lot of useful bash functions to call CHC APIs, and source
    it in your `~/.bashrc` file. You'll need to use an CitC client, but it can
    be any client, not necessarily the one you're developing in. For example,
    you could keep your chc-api-setup client alive and add this to your
    `~/.bashrc`:

    ```shell
    source /google/src/cloud/{{USERNAME}}/chc-api-client/google3/google/cloud/healthcare/scripts/chc-api-funcs.sh
    ```

    Other lines to consider adding to your `~/.bashrc`:

    ```shell
    alias gcloud=/google/data/ro/teams/cloud-sdk/gcloud
    export TOKEN=`gcloud auth application-default print-access-token`
    ```

    Note: For exporting datastores to GCS or BigQuery you need to grant Editor
    role to `cloud-healthcare-eng@system.gserviceaccount.com` in your consumer
    project. For more information see
    go/hcls-g3doc/api/service_accounts.md#service-accounts-in-dev-personal-instances

**Woohoo! You now have a working dev environment.**

API calls are authenticated using your own client project, routed from GFE to
your own dev API Frontend (ESF) through One Platform, and proxied to your own
healthcare stubby instance in borg.

![One Platform overview diagram](/google/g3doc/oneplatform/images/OnePlatform.svg){width="800"}

In this useful diagram from go/api-deploy, the API Backend in our case is our
own healthcare stubby instance running in borg.

## Where to go from here?

Maybe you'd like to **learn how to use all our API services**?

*   You can read up on our public or internal [API docs](documentation.md) and
    going through the developer guides. You can also try out some more
    [sample requests](sample_requests.md).

Maybe you'd like to **learn how to deploy healthcare endpoints** for other
environments?

*   Start with [api.md](api.md) for a reference on deploying to staging,
    autopush, and prod.
*   Also includes alternative dev environment setups, and other ways to obtain
    credentials.
