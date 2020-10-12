# Development Guide

## Contents <!-- omit in toc -->

- [Development Guide](#development-guide)
  - [Setting up a Local development environment](#setting-up-a-local-development-environment)
    - [Pre-Requisites](#pre-requisites)
    - [Forking from Git](#forking-from-git)
    - [Local configuration and setup](#local-configuration-and-setup)
    - [Setup Upstream for Sync](#setup-upstream-for-sync)
  - [Running Local Build](#running-local-build)
  - [Deploying using local Terraform provider](#deploying-using-local-terraform-provider)
  - [Acceptance Tests](#acceptance-tests)
  - [Pull Requests](#pull-requests)

---

## Setting up a Local development environment

This section covers the pre-requisites and the step by step process to setup a local environment for development of resource(s)/data-source(s) for Terraform provider for AzureRM.

### Pre-Requisites

- [Terraform version 0.12.x +](https://www.terraform.io/downloads.html)

- [Go version 1.14.x](https://golang.org/dl/)

    #### For Windows Systems

  - [Git Bash for Windows](https://git-scm.com/download/win)
  - Make for Windows.

    **NOTE** : To install Make, Install [Chocolatey](https://chocolatey.org/install)
and run the following command in PowerShell (Admin mode).

     ```PowerShell
     choco install make
     ```

### Forking from Git

Fork the [terraform-provider-azurerm](https://github.com/terraform-providers/terraform-provider-azurerm) repository.

**NOTE** : More guidance about Fork is available [here](https://docs.github.com/en/free-pro-team@latest/github/getting-started-with-github/fork-a-repo#fork-an-example-repository).

### Local configuration and setup

After installing Go, check the `$GOPATH` environmental variable and add `$GOPATH/bin` to `$PATH`

You can run the below script in PowerShell to update the path

```PowerShell
$GOPATH = $ENV:GOPATH; $ENV:PATH += ";$GOPATH\bin"
```

Then run the following command to create a folder to clone the repository

```PowerShell
mkdir -p $GOPATH/src/github.com/terraform-providers; cd $GOPATH/src/github.com/terraform-providers
```

Clone your forked repository to the terraform-providers folder

**NOTE** : Clone the repository that you have forked, not the terraform managed repository. More guidance about cloning a git repository is available [here](https://docs.github.com/en/free-pro-team@latest/github/creating-cloning-and-archiving-repositories/cloning-a-repository).

### Setup Upstream for Sync

Run the following command to setup upstream for your locally cloned repository

```bash
git remote add upstream https://github.com/terraform-providers/terraform-provider-azurerm.git
```

To verify the new upstream repository you've specified for your fork, run the following command

```bash
git remote -v
```

You should see the URL for your fork as origin, and the URL for the terraform repository as upstream

```bash
$ git remote -v
> origin    https://github.com/YOUR_USERNAME/YOUR_FORK.git (fetch)
> origin    https://github.com/YOUR_USERNAME/YOUR_FORK.git (push)
> upstream  https://github.com/terraform-providers/terraform-provider-azurerm.git (fetch)
> upstream  https://github.com/terraform-providers/terraform-provider-azurerm.git (push)
```

You can fetch changes from upstream using the following command

```bash
git fetch upstream
```

You can merge changes from upstream to your local branch using the following command

```bash
git merge upstream/master
```

**NOTE** : More guidance about Syncing a fork is available [here](https://docs.github.com/en/free-pro-team@latest/github/collaborating-with-issues-and-pull-requests/syncing-a-fork).

---

## Running Local Build

This section covers the setting up of a local build process to generate a binary output file

***NOTE** : As Make file uses `.sh` scripts, use a bash terminal to run the build. You can use git bash.*

If this is your first time running the build, run the following command to install the dependent tooling required to compile the provider.

```bash
make tools
```

Now you can build using the following command

```bash
make build
```

On successful build this will generate a binary output file in the `$GOPATH/bin` directory

***NOTE** : There are other useful make commands that are available in the official [GNUmakefile](https://github.com/terraform-providers/terraform-provider-azurerm/blob/master/GNUmakefile) and [README.md](https://github.com/terraform-providers/terraform-provider-azurerm/blob/master/README.md) files.*

---

## Deploying using local Terraform provider

You can use the locally developed provider to deploy resources in Azure by running the `terraform init` command with flags `-get-plugins=false` and `-plugin-dir=$GOPATH/bin`.

```Powershell
$GOPATH = $ENV:GOPATH;
terraform init -get-plugins=false -plugin-dir=$GOPATH/bin
```

---

## Acceptance Tests

Terraform resource Acceptance Tests provisions resources in Azure and validates the configuration of the resource using terraform state and deletes all the provisioned resources, acceptance tests can be applied in multiple steps and updates to the configuration is applied and tested. At the end of each acceptance  test.

***NOTE** : Acceptance Tests fail if any resources cannot be provisioned/de-provisioned or any configurations that are being validated are not available in the state file.*

Tests are available for a resource in the `azurerm/internal/services/SERVICE_NAME/tests` folder and follows the naming convention of `RESOURCE_NAME_test.go`

To run acceptance tests,

- Create a service principal in Azure and obtain Client ID and Client Secret

    ***NOTE** : Guidance on Creation of Service Principal in Azure is available [here](https://www.terraform.io/docs/providers/azurerm/guides/service_principal_client_secret.html) and [here](https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-create-service-principal-portal).*

- Set the following required environment variables with their corresponding values

    ```bash
    export ARM_CLIENT_ID=""
    export ARM_CLIENT_SECRET=""
    export ARM_SUBSCRIPTION_ID=""
    export ARM_TENANT_ID=""
    export ARM_TEST_LOCATION=""
    export ARM_TEST_LOCATION_ALT=""
    export ARM_TEST_LOCATION_ALT2=""
    ```

- Use the `go test` command with the flag `TF_ACC=1` to run the acceptance tests, use a regular expression to filter and run only the targeted acceptance tests in the folder path provided. Use the necessary `-timeout` as this can be a long running process in case of certain resources or when multiple acceptance tests are run in parallel.
  
  ```bash
  TF_ACC=1 go test -v ./azurerm/internal/services/SERVICE_NAME/tests -run ^REGULAR_EXPRESSION_WITH_TEST_NAME$ -timeout 6h
  ```

***NOTE** : Documentation for Acceptance Tests is available [here](https://www.terraform.io/docs/extend/testing/acceptance-tests/index.html).*

---

## Pull Requests

Once you commit your changes to your local branch, you can push your changed into your forked reposity and raise a pull request to merge your changes to the upstream reposioty.

Guidance on creating Pull Request from a forked repository is available [here](https://docs.github.com/en/free-pro-team@latest/github/collaborating-with-issues-and-pull-requests/creating-a-pull-request-from-a-fork).

You can track the status of the existing pull requests [here](https://github.com/terraform-providers/terraform-provider-azurerm/pulls)
