# API Library & GCP Marketplace

go/hcls-api-library

[TOC]

<!--*
# Document freshness: For more information, see go/fresh-source.
freshness: { owner: 'thelaw' reviewed: '2019-10-30' }
*-->

The
[Cloud Healthcare API library page](https://pantheon.corp.google.com/apis/library/healthcare.googleapis.com)
is what users see when they are browsing the API library or attempting to enable
the healthcare API. The API Library is powered by the GCP Marketplace
(go/op-marketplace) and is managed separately from our service config and
documentation pages.

## Updating the API Library Description

Important: Make sure that any changes to the API library page are run past
someone from the docs team (g/cloud-health-docs) first.

On occasion the information on the
[Cloud Healthcare API library page](https://pantheon.corp.google.com/apis/library/healthcare.googleapis.com)
needs to be updated. This can only be done by someone who is an owner of the
producer project (`cloud-healthcare-api` for production). Typically this should
be done via the [primary or secondary oncall](http://go/hcls-oncall).

In order to update the description or any other fields such as documentation
links, follow the *Updating your service page* steps in the
[marketplace documentation](http://go/op-marketplace) (the API library is
powered by the GCP Marketplace). You can skip the first step (deploy your
service config), as that is done automatically during our release process.
