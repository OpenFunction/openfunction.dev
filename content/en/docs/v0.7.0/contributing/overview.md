---
type: docs
title: "Overview"
linkTitle: "Overview"
weight: 7100
description:
---

This document provides the guidelines for how to contribute to the [OpenFunction](https://github.com/OpenFunction) through issues and pull-requests. Contributions can also come in additional ways such as engaging with the community in community calls, commenting on issues or pull requests and more.

See the [OpenFunction community repository](https://github.com/OpenFunction/community) for more information on community engagement and community membership.

## Issues

### Issue types

In most OpenFunction repositories there are usually 4 types of issues:

- Issue/Bug report: You've found a bug and want to report and track it.
- Issue/Feature request: You want to use a feature and it's not supported yet.
- Issue/Proposal: Used for items that propose a new idea or functionality. This allows feedback from others before code is written.

### Before submitting

Before you submit an issue, make sure you've checked the following:

1. Is it the right repository?
    - The OpenFunction project is distributed across multiple repositories. Check the list of [repositories](https://github.com/OpenFunction) if you aren't sure which repo is the correct one.
2. Check for existing issues
    - Before you create a new issue, please do a search in [open issues](https://github.com/OpenFunction/OpenFunction/issues) to see if the issue or feature request has already been filed.
    - If you find your issue already exists, make relevant comments and add your [reaction](https://github.com/blog/2119-add-reaction-to-pull-requests-issues-and-comments). Use a reaction:
        - 👍 up-vote
        - 👎 down-vote

## Pull Requests

All contributions come through pull requests. To submit a proposed change, follow this workflow:

1. Make sure there's an issue raised, which sets the expectations for the contribution you are about to make.
2. Fork the relevant repo and create a new branch
3. Create your change
    - Code changes require tests
4. Update relevant documentation for the change
5. Commit with [DCO sign-off]({{< ref "overview.md#developer-certificate-of-origin-signing-your-work" >}}) and open a PR
6. Wait for the CI process to finish and make sure all checks are green
7. You can expect a review within a few days

### Use work-in-progress PRs for early feedback

A good way to communicate before investing too much time is to create a "Work-in-progress" PR and share it with your reviewers. The standard way of doing this is to add a "[WIP]" prefix in your PR's title and assign the **do-not-merge** label. This will let people looking at your PR know that it is not ready yet.

## Developer Certificate of Origin: Signing your work
### Every commit needs to be signed

The Developer Certificate of Origin (DCO) is a lightweight way for contributors to certify that they wrote or otherwise have the right to submit the code they are contributing to the project. Here is the full text of the [DCO](https://developercertificate.org/)

Contributors sign-off that they adhere to these requirements by adding a `Signed-off-by` line to commit messages.

```
This is my commit message
Signed-off-by: Random J Developer <random@developer.example.org>
```

Git has a `-s` command line option to append this automatically to your commit message:
```
git commit -s -m 'This is my commit message'
```

Each Pull Request is checked whether or not commits in a Pull Request do contain a valid Signed-off-by line.

### I didn't sign my commit, now what?!

No worries - You can easily replay your changes, sign them and force push them!

```
git checkout <branch-name>
git commit --amend --no-edit --signoff
git push --force-with-lease <remote-name> <branch-name>
```

## Development Guide

[Here](https://github.com/OpenFunction/OpenFunction/tree/main/docs/development) you can find a development guide that will walk you through how to get started with building OpenFunction in your local environment.

## Code of Conduct

Please see the [OpenFunction community code of conduct](https://github.com/OpenFunction/OpenFunction/blob/main/code-of-conduct.md).