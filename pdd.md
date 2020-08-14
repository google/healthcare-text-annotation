# Cloud Healthcare API Privacy Design Doc

go/chc-pdd

[TOC]

<!--*
# Document freshness: For more information, see go/fresh-source.
freshness: { owner: 'gavinbee' reviewed: '2019-06-11' }
*-->

## Cloud Healthcare API Privacy Design Doc

IMPORTANT: This is the canonical PDD used for the Cloud Healthcare API. All
incremental launches affecting the PDD should be appended as new versions to
this PDD.

**https://eldar.corp.google.com/assessments/607161053**

## About this Document

The Eldar entry above is the Master PDD for the
[Cloud Healthcare API](http://go/health-cloud-api).

This Eldar entry supersedes the
[Google Doc](https://docs.google.com/document/d/1Mc8-Uk9vsi-Ap2vzbpQzkSIG905hbAbnGleVqLkOUZc/edit#)
that was originally used as the Master PDD.

TIP: See the following sections for help on making changes to the Eldar entry
for your own launches.

### When and How to Update

Updates should normally be identified as part of launching changes working
through the [launch stage](http://go/hcls-lifecycle-launch) of the HCLS
Lifecycle. Be sure to check go/hcls-lifecycle-launch for additional guidance.

When it comes time to consider the privacy impact of your changes, you should
use the subsequent sections of this document as a reference and guide.

Start reviewing the
[Eldar entry](https://eldar.corp.google.com/assessments/607161053) at the
*Collection* section. As soon as you identify additions or changes needed as a
result of your launch, create a new version of the PDD in Eldar by following the
instructions below.

If you have no changes after reviewing then you are done! Be sure to make a note
that you completed this review in your
[launch doc](http://go/hcls-lifecycle-launch#write-launch-doc).

If you have changes then upgrade your launch to a Regular launch if it isn't
already AND complete the following steps:

1.  Create a new version in Eldar for your launch. To do this, click on the
    'Create New Version' button
    [in the tool](https://eldar.corp.google.com/assessments/607161053).

    ![Screenshot of the Create New Version button](https://screenshot.googleplex.com/zKit3QyGRF4.png)

1.  Choose a meaningful name for your version and be sure to reference the
    Ariane entry in your description.

1.  Review all sections and make changes/additions as needed as a result of your
    launch.

1.  When you are done, review the diffs you have generated to sanity check your
    changes. This is likely going to be the first thing your privacy reviewer
    will look at.

    ![Screenshot of the View Diff button](https://screenshot.googleplex.com/EtEj0cXcHBH.png)

1.  When you are satisfied, mark your PDD as Ready for Review.

    ![Screenshot of the Ready button](https://screenshot.googleplex.com/yJJaHiRjh0V.png)

1.  Link to the CL and Eldar PDD in your
    [launch doc](http://go/hcls-lifecycle-launch#write-launch-doc)

Once your PDD changes are reviewed and approved, the changes are automatically
merged into the canonical version if you have 'Enable Auto Commit' checked.

## Privacy Pointers

You may find the following links useful when making changes to the PDD or when
considering privacy topics in your design:

*   go/cloud-privacy-primer
*   go/cloud-privacy-resources
*   go/storage-deletion-latencies
*   go/udrdp

## PDD Guide

The following guide is helpful when filling out the PDD, see this for help and
details about what should go into each section.

go/eldar-pdd-guide

## Contact

Need more help? Have a specific privacy question? Don't hesitate to reach out to
hcls-privacy@google.com.
