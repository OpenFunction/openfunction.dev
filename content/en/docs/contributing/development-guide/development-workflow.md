---
title: "Development Workflow"
linkTitle: "Development Workflow"
weight: 7220
description: >	
    Learn how to follow the development workflow for OpenFunction.
---

This document describes the development workflow for OpenFunction.

## Development Workflow

### Prerequisites

You need to have a Kubernetes cluster and a Go development environment. If not ready, [install your Kubernetes cluster](https://kubesphere.io/blogs/install-kubernetes-using-kubekey/) and [set the Go environment up](https://go.dev/doc/code) first.

### Step 1: Fork the OpenFunction Repository

1. Visit [the OpenFunction repository](https://github.com/OpenFunction/OpenFunction).
2. Click **Fork** in the upper-right corner to fork the repository to your GitHub account.

### Step 2: Clone the forked repository

According to the [workspace instructions](https://go.dev/doc/code#Workspaces) of Go, perform the following procedure to clone the OpenFunction source code to your `GOPATH`.

1. Run the following commands to define a local working directory.

   ```shell
   export working_dir=$GOPATH/src/openfunction.io
   export user=<your GitHub account name>
   ```

2. Run the following commands to clone the forked repository.

   ```shell
   mkdir -p $working_dir
   cd $working_dir
   git clone https://github.com/<your GitHub account name>/OpenFunction.git
   cd $working_dir/OpenFunction
   git remote add upstream https://github.com/OpenFunction/OpenFunction.git
   ```

3. Run the following command to avoid pushing your updates to the upstream master branch.

   ```shell
   git remote set-url --push upstream no_push
   ```

4. (Optional) Run the following command to check your remote repositories.

   ```shell
   git remote -v
   ```

### Step 3: Add new features or fix issues

1. Run the following command on your local main branch to pull the latest code before making any changes.

   ```shell
   git pull --rebase upstream main
   ```

2. Run the following command to create a new branch, and then edit code  on the new branch. You can refer to [Effective Go](https://go.dev/doc/effective_go) while writing code.

   ```shell
   git checkout -b <new branch name>
   ```

3. Test and build. Currently, the make rules only contain simple checks such as vet, unit test, will add e2e tests soon.

4. Use [KubeBuilder](https://book.kubebuilder.io/introduction.html).

   - For Linux, you can download and execute this [KubeBuilder script](https://raw.githubusercontent.com/kubesphere/kubesphere/master/hack/install_kubebuilder.sh).
   - For macOS, you can install KubeBuilder by following [this guide](https://book.kubebuilder.io/quick-start.html).

5. Run the following commands to compile code and run tests.

   ```shell
   make all
   # Run every unit test.
   make test
   ```

   {{% alert title="Note" color="success" %}}

   You can run the `make help` command for additional information.

   {{% /alert %}}

### Step 4: Commit and push local changes

1. Run the following command to pull the lates code again before committing your local changes. If any conflicts exist after running the command, you have to resolve them.

   ```shell
   git pull --rebase upstream main
   ```

2. Run the following commands to commit your local changes.

   ```shell
   git add <changed file>
   git commit -s -m "<description>"
   ```

3. Run the following command to push the changes to your repository.

   ```shell
   git push -u origin <new branch name>
   ```

### Step 5: Create a pull request

1. Visit your forked OpenFunction repository.

2. Switch to the new branch and click **Compare & Pull Request**.

3. Write necessary content in the pull request description, and then click **Create Pull Request**.

   {{% alert title="Note" color="success" %}}

   For more information, see  [this GitHub document](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request).

   {{% /alert %}}

4. You need to assign a maintainer of the OpenFunction project to review your pull request.



