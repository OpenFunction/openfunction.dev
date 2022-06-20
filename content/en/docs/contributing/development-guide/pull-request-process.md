---
title: "Pull Request Best Practices"
linkTitle: "Pull Request Best Practices"
weight: 7230
description:
---

This document describes the best practices of creating pull requests on the OpenFunction repository.

## Before Creating a Pull Request

You have to follow the [Development Workflow]({{< ref "../development-guide/development-workflow" >}}) to make code changes locally. Make sure you fully test your code locally.

## The Pull Request Best Practices

When you [create a pull request]({{< ref "../development-guide/development-workflow#step-5-create-a-pull-request" >}}), you can refer to the following best practices.

### Write release notes if needed

Release notes are required for any PR with user-visible changes, such as bug fixes, additional features, and output format changes. To add a release-note section to the PR description, refer to the following examples.

- For PRs with a release note, add the relevant content.

  ````
  ```release-note
  <your release note>
  ```
  ````

- For PRs that require additional action from users when using the new release, add the string `action required` (case insensitive) in the release note.

  ````
  ```release-note
  action required: <your release note>
  ```
  ````

- For PRs that don't need to be mentioned at release time, write `NONE` (case insensitive). You can also make a comment of `/release-note-none` after creating your PR.

  ````
  ```release-note
  NONE
  ```
  ````

{{% alert title="Note" color="success" %}}

If you don't add your release notes in the PR, the `do-not-merge/release-note-label-needed` label is automatically added to your PR after you create it.

{{% /alert %}}

### Test and merge a pull request

The OpenFunction repository uses labels by comments to run tests and merge pull requests.

{{% alert title="Note" color="success" %}}

For pull requests that are in progress and not ready for review, prefix the PR title with `WIP` or `[WIP]` and track remaining items through a checklist in the pull request description.

{{% /alert %}}

If you're **not** a member of the OpenFunction organization:

1. If a reviewer thinks the PR is good for test, the reviewer will make a comment of `/ok-to-test`.
2. The reviewer suggests changes.
3. Push changes to your PR branch according to comments.
4. Repeat the prior two steps as needed.
5. (Optional) When your PR passes review, the reviewer may prefer that you squash all your previous commits.
6. Assign the owner to add the `/approve` label to the PR.

If you are a member, or a member comments `/ok-to-test`, the PR will be considered to be trusted. Then the pre-submit tests will run:

1. Automatically run tests. See the current list of tests on the PR.
2. If tests fail, resolve issues by pushing edits to your PR branch.
3. If the failure is a flake, anyone on trusted PRs can comment `/retest` to rerun failed tests.

Once the tests, all failures are commented as flakes, or the reviewer adds the labels `lgtm` and `approved`, the PR enters the final merge queue. The merge queue is needed to make sure no incompatible changes have been introduced by other PRs since the tests were last run on your PR.
