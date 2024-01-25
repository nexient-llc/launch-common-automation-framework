# launch-common-automation-framework

## Overview

This repository is used to seed component dependencies of another repository (e.g. a Terraform module) and is applied using the [repo](https://gerrit.googlesource.com/git-repo/) utility.  Launch has updated the `repo` utility to allow for substituting environment variables in the manifests to allow for more dynamic behavior. Our updated copy of the tool can be [found here](https://github.com/nexient-llc/git-repo).

## LCAF Components and Modules

- A **component** is a repository that covers platform-level concerns that apply across many repositories. These are all prefixed with `lcaf-component-` in their name and are typically only referenced through this entrypoint (the `launch-common-automation-framework` repository and its manifests).

- A **module** is an individual repository that represents a piece of software, a cloud resource defined by an IaC tool, or in some cases, an IaC helper used to provide functionality to other IaC modules (such as the [resource_name](https://github.com/nexient-llc/tf-module-resource_name) helper for Terraform). Modules utilize components to provide platform-level functionality that is shared across many modules.

## Authentication

This repository is publicly available and can be retrieved by the `repo` tool using anonymous access, however, we have run into scenarios where clients choose to host a private copy of this repository either on GitHub or in their SCM system of choice. In this scenario, you are responsible for configuring appropriate authentication prior to interacting with your fork of `launch-common-automation-framework`. 

Typically, this is handled by creating records in your `~/.netrc` file to provide a login and password for the host in question. Documentation showing how to configure `.netrc` can be [found here](https://everything.curl.dev/usingcurl/netrc).

## Manifests

Manifests define a composable structure of repositories and files that are synchronized into a repository when `repo init` is invoked by the `make configure` target. 

Our standard is to point the initial manifest target at a seed manifest, which then pulls in platform-level dependencies and sets up local files according to that manifest's type. 

For example, a [seed manifest for Terraform](manifests/terraform_modules/seed/manifest.xml) would pull in the [manifest necessary to set up a git connection](manifests/git-connection/manifest.xml), the [platform-level seed manifest](manifests/platform/seed/manifest.xml) (which in turn pulls in the [main platform manifest](manifests/platform/manifest/manifest.xml)), as well as the [main Terraform manifest](manifests/terraform_modules/manifest/manifest.xml) that contains configuration files and additional Make targets specific to Terraform.

In general, new modules for a predefined type of LCAF module like Terraform are generated from either a skeleton repository (that will already have the correct manifest location built into the included Makefile), or via `launch-cli` tooling, but if you need to create a new skeleton or a one-off repository that utilizes LCAF, you will need to configure a Makefile. A [sample Makefile](examples/Makefile) is included with this repository to provide a starting point for development. 

The relevant variables to changing synchronization/manifest behavior in the [sample Makefile](examples/Makefile), are shown below:

```
# Normally this will point to this repository, except in cases where you choose to self-host a fork.
REPO_MANIFESTS_URL ?= https://github.com/nexient-llc/launch-common-automation-framework.git
# Typically points at a git tag, but in special cases you can use this to point to a branch or a specific commit hash
REPO_MANIFESTS_REVISION ?= refs/tags/0.1.3
# This should always point at a seed manifest that then includes other manifests
REPO_MANIFESTS_PATH ?= manifests/terraform_modules/seed/manifest.xml

# The following settings tell the repo tool to pull in Launch version of itself:
REPO_URL ?= https://github.com/nexient-llc/git-repo.git
REPO_REV ?= main
```

However, for tweaking these settings on individual repositories, the recommended approach is to use the `.lcafenv` file, detailed in the following section.

## .lcafenv / Makefile Includes

In many cases, it is easier to place values that are commonly changed between repositories into a separate file to be included with your Makefile, to lessen the cognitive load associated with reviewing changes to a Makefile. An [example .lcafenv](examples/.lcafenv) is included with this repository, and our included [example Makefile](examples/Makefile) will reference this file to try to override its default settings.

Since the Makefile utilizes the `?=` operator, the values present in the Makefile are only effective if that variable is not already defined. Simply uncomment a value in `.lcafenv` to have it take precedence over the value in the Makefile.

## ASDF and .tool-versions

Our platform makes use of `asdf-vm` to further manage dependencies. Repositories that utilize LCAF are expected to have a `.tool-versions` file in their root to handle setting up required software both from a local development point of view as well as in the CI/CD aspect.

A [sample .tool-versions](examples/.tool-versions) file is included with this repository to demonstrate how a .tool-versions file used for a Terraform repository may look. Please note that when copying this file into your own project, you should carefully review the dependencies of your project and choose the latest compatible version of each.

For more information on `asdf-vm`, see [their documentation](https://asdf-vm.com/guide/introduction.html).

## Initializing your Repository with LCAF

You should execute `make configure` from the root of your repository and the `repo` tool will attempt to pull in files based on the manifest located at  `REPO_MANIFESTS_PATH` in the `REPO_MANIFESTS_REVISION` revision of the repository located at `REPO_MANIFESTS_URL`.

Once `make configure` has successfully completed, you will have additional `make` targets available for use. The exact targets will depend on your choice of manifest; see the individual LCAF modules' tasks/ directories for information on which commands are available.

## Notes

* The Makefile under examples/ is a good starting place for developing a new class of modules by creating a new skeleton, but most of the time, your initial Makefile will come from a [skeleton repository](https://github.com/nexient-llc/tf-module-skeleton) that includes instructions on how to bootstrap it into an LCAF module.

* For Bitbucket, you *must* define your Bitbucket credentials in your `~/.netrc` and that *must* use an HTTP access token.  Also, that token *must* have some expiration date, otherwise Bitbucket will lock your account after a few accesses.

* Additional platform documentation can be [found here](https://github.com/nexient-llc/common-platform-documentation).