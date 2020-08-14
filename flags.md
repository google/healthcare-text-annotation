# Flags in the Cloud Healthcare API

go/chc-flags, go/chc-cleft

<!--*
# Document freshness: For more information, see go/fresh-source.
freshness: { owner: 'thelaw' reviewed: '2020-01-06' }
*-->

[TOC]

This document describes the lifecycle phases of a feature flag within the Cloud
Healthcare API.

IMPORTANT: When rolling out a new feature, which involves a Spanner schema
change, do NOT rely on the schema being in prod right away when code containing
the feature rolls out to production, as it makes the release/schema not
rollback-safe. Instead, allow for the schema to be in prod for at least 1
release cycle before enabling the feature that utilizes the schema change.

## Flag Locations

All of the feature flags live within the
[production/borg/cloud-healthcare/templates/](http://cs/google3/production/borg/cloud-healthcare/templates/).
There exists a primary borg file for each one of our environments:

*   [healthcare_dev.borg](http://cs/google3/production/borg/cloud-healthcare/templates/healthcare_dev.borg)
*   [healthcare_autopush.borg](http://cs/google3/production/borg/cloud-healthcare/templates/healthcare_autopush.borg)
*   [healthcare_staging.borg](http://cs/google3/production/borg/cloud-healthcare/templates/healthcare_staging.borg)
*   [healthcare_prod.borg](http://cs/google3/production/borg/cloud-healthcare/templates/healthcare_prod.borg)

Sandman (our integration testing environment) flags work slightly differently
and are segmented by binary, these are defined in the following location:

*   [production/borg/cloud-healthcare/sandman](http://cs/google3/production/borg/cloud-healthcare/sandman/)

Flag values should not be set anywhere else **unless absolutely necessary**,
feel free to consult with hcls-infra@google.com with any question.

## CLEFT

CLEFT is used for all non-ESF Cloud Healthcare binaries. This causes flag values
to be automatically updated depending on which CLs are included in a built
binary so manual flag progression is not needed in releases. CLEFT may also be
used for ESF Cloud Healthcare binaries.

## Lifecycle

### Defining a new feature flag

Follow these steps to define your feature flag. The following steps can all be
done in a single CL (example: cl/259817444).

1.  In your code, [define your flag](http://go/gocomments#TOC-Flags) in the
    main.go file for the affected binary. Replace `enableFeature` with your flag
    name.

    ```go
    var (
        enableFeature = flag.Bool("enable_feature", false, "Whether to enable some feature.")
    )
    ```

1.  Pass this flag value to the affected code location.

1.  Add two new CLEFT conditions to
    [cleft.borg](http://cs/google3/production/borg/cloud-healthcare/templates/cleft.borg).
    One is used to enable the flag in autopush, and the second will turn it on
    later in staging and production.

    ```gcl
    enable_feature_autopush = change_tmpl { ocl = <YOUR_CL_NUMBER> }
    enable_feature_staging_prod = change_tmpl {}
    ```

    Note: We intentionally omit `ocl` in the `_staging_prod` condition because
    we don't want it turned on there until sufficient validation has been done
    in autopush.

#### Testing CLEFT conditions

The
[`changes.flag_change('enable_feature', new, old)`](http://google3/production/borg/cloud-healthcare/templates/utils.borg?l=231&rcl=315540804)
borg config lambda tests if the cl corresponding to `'enable_feature'` is
included in a binary, and returns `new` if it is, or `old` otherwise.

```
Note: The first argument of the `flag_change` function should match the
prefix of the new CLEFT conditions you added (i.e. the bit before
`_autopush` and `_staging_prod`).
```

[Convenience lambdas](http://google3/production/borg/cloud-healthcare/templates/utils.borg?l=256&rcl=315540804)
simplify common use-cases:

```
changes.new_flag(f, new)       → changes.flag_change(f, new, null)
changes.new_true_flag(f, new)  → changes.flag_change(f, true, null)
```

#### Next step

The next step depends on how your flag works. If the value will be the same
regardless of region and environment then follow the
[simple steps](#new-flag-simple). Otherwise, if you need different values for
different regions/environments (e.g. specifying a GSLB target) follow the
[Advanced steps](#new-flag-advanced).

#### Simple Case {#new-flag-simple}

1.  In the main binary's borg file within the templates directory, add the
    following to the `args` section (there should be plenty of examples).

    ```gcl
    args {
      ...
      enable_feature = changes.new_flag('enable_feature', <VALUE>)
    }
    ```

1.  Move on to [Finishing Up](#new-flag-finishing-up).

#### Advanced Case {#new-flag-advanced}

WARNING: Before following these steps, please reach out to hcls-infra@ to make
sure that this is the appropriate solution for you. The vast majority of flag
changes should be handled via the simple case above.

1.  In the main binary's borg file within the templates directory, add the
    following to the `params` and `args` sections (there should be plenty of
    examples).

    ```gcl
    params {
      ...
      enable_feature = external
      ...
    }
    args {
      ...
      enable_feature = changes.new_flag('enable_feature', params.enable_feature)
    }
    ```

    Note: The first argument of the `flag_change` function should match the
    prefix of the new CLEFT conditions you added (i.e. the bit before
    `_autopush` and `_staging_prod`).

1.  In the
    [environment files](http://cs/search/?q=f://depot/google3/production/borg/cloud-healthcare/templates/healthcare_.*)
    and [sandman](http://google3/production/borg/cloud-healthcare/sandman/), set
    the appropriate value of your flag. It is safe to modify the staging and
    prod files in this CL because the actual flag change itself will be guarded
    by the CLEFT conditions introduced earlier.

1.  Move on to [Finishing Up](#new-flag-finishing-up).

#### Finishing Up {#new-flag-finishing-up}

Add your CLEFT changes to a single CL and ensure the UltraBorgDiff presubmit
check passes. This can be done by triggering a presubmit check from Critique or
running the following command:

```shell
ubclient -c <CL number>
```

Important: You shouldn't see any diffs in autopush/staging/prod in this
ultraborgdiff run because the changes will only take effect once your CL is
submitted and reaches those environments.

At release time Rapid will automatically run a tool which replaces the OCL value
with the submitted CL#. At that point CLEFT will use your new flag value iff
your submitted CL is present in the binary being used. e.g. cl/202225411

### Defining an integration test flag

Since your feature is not enabled in staging, your integration test will fail
when it is run against staging, at the time of release. To avoid this, when you
add an integration test for the new feature, add a test flag to guard it.

1.  Add a flag `enable_my_feature` in your integration test
1.  Add the flag to the list of feature flags dictionary in
    [feature_flags.bzl](http://cs/google/cloud/healthcare/test/guitar/feature_flags.bzl).
    Enable the flag for presubmit and autopush accordingly.

### Enabling in Staging and Prod {#staging-prod}

IMPORTANT: Only proceed with this step when your launch has been fully approved
and the Eng Review bug has been marked fixed by your reviewer (see
http://go/chc-launches#launch-doc-and-engineering-review).

The steps to follow are the following:

1.  Ensure your launch is fully approved.

1.  In
    [cleft.borg](http://google3/production/borg/cloud-healthcare/templates/cleft.borg),
    update your feature's `_staging_prod` condition and add `ocl =
    <NEW_CL_NUMBER>` with your current CL (*NOT* the original CL that added the
    feature).

1.  Flip the flag to enable the integration test testing this new feature in
    staging in
    [feature_flags.bzl](http://cs/google/cloud/healthcare/test/guitar/feature_flags.bzl).

1.  Link your engineering review bug from your CL description.

### Clean up the flag {#clean-up}

Once your launch has been in production for a while, you can remove the flag.

NOTE: It is perfectly fine to leave a flag in place if it protects a feature
that could cause production issues and would be useful if it can be turned off
quickly by the on-call team.

To clean up a flag, do the following two parts. (Example CLs: cl/210913021,
cl/213328406)

#### Part 1 (Example CL: cl/210913021)

1.  Prepend `deprecate_` to existing CLEFT entries you wish to deprecate:

    ```gcl
    // TODO(b/yyy): Remove when cl/zzz is rollback safe.
    deprecate_enable_feature_autopush = change_tmpl { ocl = <YOUR CL NUMBER> }
    deprecate_enable_feature_staging_prod = change_tmpl { ocl = <YOUR CL NUMBER> }
    ```

1.  In the binary borg file, change the CLEFT condition to set the flag to null
    if the condition is true.

    ```gcl
    args {
      ...
      enable_feature = changes.flag_change('deprecate_enable_feature', null, true)
    }
    ```

1.  Remove the flag from the `params` section in the binary borg file.

1.  Remove the flag values from the various environment files (including
    Sandman).

1.  Send the CL out for review and submit.

At release time, Rapid will automatically replace the OCL with the submitted
CL#. At that point CLEFT will use the `null` value iff your submitted CL is
present in the binary being used.

#### Part 2 (Example CL: cl/213328406)

IMPORTANT: Wait until the previous CL is fully rolled out to production and is
rollback safe (ie. two production releases have occurred).

1.  Remove the entry from cleft.borg and from the binary borg file.

1.  Send out the second CL out for review and submit.

### Enabling features for select API versions.

While rolling out features to individual API versions, usually the feature can
be restricted to the appropriate versions by adjusting the visibility of an
associated proto field. In this case, the feature flag can be enabled across all
API versions. However, in some cases no such field exists, so visibility cannot
be used. For example, changing default behaviour for existing functionality
(cl/271633991). In this case, the feature flag can help restrict to particular
versions, as follows:

#### Create flag with initial versions

1.  Define a "feature_enabled_versions" string list flag.

    ```go
    var (
        featureEnabledVersions = flag.StringList("feature_enabled_versions", nil, "Versions where some feature is enabled.")
    )
    ```

1.  Create the usual cleft condition, as above.

1.  Populate it with a list of the enabled API versions.

    ```gcl
    feature_enabled_versions = changes.new_flag('feature_enabled_versions', '<version1>,<version2>')
    ```

1.  In your binary, where you need to check whether to apply the feature, check
    if the feature is enabled for the API version of the request being
    processed. You can get the request's API version based on the
    [context](https://cs.corp.google.com/piper///depot/google3/cloud/healthcare/util/api_version.go?rcl=254408280&l=41).

1.  Progress the flag through its usual Lifecycle.

#### Adding additional versions

1.  Add a new CLEFT condition as before with a new cl number. Ensure there isn't
    a collision with your last flag (eg. append the new version).

    ```gcl
    feature_enabled_<NEW VERSION>_autopush = change_tmpl { ocl = <YOUR_NEW_CL_NUMBER> }
    feature_enabled_<NEW_VERSION>_staging_prod = change_tmpl {}
    ```

1.  Set the binary's borg file to use the new condition:

    ```gcl
    // If the old CLEFT flag has been deprecated or removed:
    feature_enabled_versions = changes.flag_change('feature_enabled_<NEW_VERSION>', <OLD + NEW VERSIONS>, <OLD VERSIONS>)
    // If the old CLEFT flag is still active:
    feature_enabled_versions = join(
          [changes.new_flag('feature_enabled_versions', <OLD VERSIONS>),
          changes.new_flag('feature_enabled_v1beta1', <NEW VERSION>)],
           ',')
    ```

## New binaries

When CLEFT is used for guarding a new binary turn up the advised approach would
be to first rollout the release that contains a CL used for guarding a job and
only after that turning up the jobs manually as CLEFT will check whether the mpm
is correctly labeled with environment/region value.

In case a faster turn up is required, the oncall person will need to:

*   launch a deploy binary workflow fist that will fail on 'borgcfg' stage as
    new service is missing

*   manually start the jobs for that region (cell)

*   retry binary deployment
