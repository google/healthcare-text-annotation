# Discovery

go/chc-discovery

[TOC]

<!--*
# Document freshness: For more information, see go/fresh-source.
freshness: { owner: 'jiahuay' reviewed: '2019-04-08' }
*-->

From go/api-discovery:

> The notion of discovery is used in the Google API Platform for functionality
> related to representing and retrieving metadata for APIs. This data can be
> used in multiple ways by API consumers: it can be input to tools like API
> Explorer, it can be used for generation of client libraries, or it can be used
> for any other introspection based activity on the API.

The major use cases are:

-   Provide API Documentation
-   Client library auto-generation
-   APIs Explorer

See google3/google/api/discovery.proto for details on how to configure the
discovery rules.

## Discovery Docs for Cloud Healthcare

Our ESFs expose REST discovery documents at
https://healthcare.googleapis.com/$discovery/rest. Similarly a Swagger 2
discovery doc is available at
https://healthcare.googleapis.com/$discovery/swagger2.

See g3doc/google/g3doc/oneplatform/discovery.md#esf-discovery-url for a
description of additional query parameters that can be used.

Discovery for the Cloud Healthcare service is configured in discovery section of
google3/google/cloud/healthcare/healthcare.yaml and
google3/google/cloud/healthcare/prod.yaml.

## Discovery and Visibility

Public discovery is enabled for the production healthcare API overall.
Unlaunched versions of the API may still be set to private discovery via
settings in prod.yaml.

To view a discovery document on autopush and include everything with
GOOGLE_INTERNAL visibility use a URL like the following:
`https://autopush-healthcare.sandbox.googleapis.com/$discovery/rest?version=v1alpha2&labels=GOOGLE_INTERNAL&key=$API_KEY&token=$TOKEN`

Where:

*   `$API_KEY` can be created at
    https://pantheon.corp.google.com/apis/credentials. Any API key will do, it
    is just used for quota purposes.
*   `$TOKEN` can be generated from the command line using `gcloud auth
    print-access-token`.

NOTE: the `$discovery` part in the above URL has a literal '$' character, not
meant to be a variable.

For example, to get the production discovery doc for `CHC_ALPHA` visibility, you
would use
`https://healthcare.googleapis.com/$discovery/rest?version=v1alpha2&labels=CHC_ALPHA&key=$API_KEY&access_token=$TOKEN`

The ESF Dicovery documents are generated as part of the api_service BUILD rule
(e.g. for
[autopush](https://cs.corp.google.com/piper///depot/google3/google/cloud/healthcare/BUILD?rcl=213822946&l=189))
at compile time.

## Stand-alone Discovery Documents

There is also api_discovery BUILD rule for building and generating discovery
document statically in google3/google/cloud/healthcare/BUILD. go_apiary_library
then uses these discovery documents to generate client libraries.
