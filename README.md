## Overview

This repo is a template for creating .NET repos. You can do this by clicking the **Use this template** button in the top right corner.

## git

Both `.gitattributes` and `.gitignore` are adapted for .NET development.

## VS Code

This repo contains `launch.json` and `tasks.json` used to configure debugging and other tasks. These files makes use of the [Command Variable](https://marketplace.visualstudio.com/items?itemName=rioj7.command-variable) extension to prompt user for build configuration (`Release` or `Debug`) and which test project to launch.  

## .NET

There is a `Directory.Build.props` file for common MSBuild properties to avoid copy-pasting across different C# projects. Depending on your setup, you may want to add some additional `Directory.Build.props` files in sub-folders.

It is recommended to use centralized package management as provided by `Directory.Packages.props`. If you choose not to do so, you can delete this file.

There is an `.editorconfig` that follows default style and naming rules for the most part.

`global.json` allows you to define which .NET SDK version is used when you run .NET CLI commands. This is useful so commands use the same SDK version whether run locally or in CI scenarios.

### Layout

It is recommended that you keep the solution file(s) in the root directory, non-test code in `./src`, test code in `./tests`.

All build outputs from all projects are saved in `./artifacts/bin`, separated by project. For more information, see [Artifacts output layout](https://learn.microsoft.com/en-us/dotnet/core/sdk/artifacts-output).

## CI/CD

There are several GitHub workflows for CI defined. Feel free to modify them as needed. Each workflow runs fairly quickly (in my experience).  You may want to combine some of the workflows, e.g. on pushing to a branch, you may want a single workflow that builds the code, perform SAST, run tests in that order rather than have separate workflows for each task.

Several of the workflows use the following two GitHub Actions:

1. [`fkirc/skip-duplicate-actions`](https://github.com/fkirc/skip-duplicate-actions/tree/v5/): This prevents duplicate runs of the same workflow. This can happen for example if a workflow is triggered by more than one of the following events: `pull_request`, `push` and `workflow_dispatch`.

2. [`dorny/paths-filter`](https://github.com/dorny/paths-filter)

### Workflows

#### Security + Linting

##### CodeQL Analysis 🔍

Automated code analysis to identify vulnerabilities and coding errors in code.

**File**: [codeql-analysis.yml](.github/workflows/codeql-analysis.yml)

**Notes**:

> ℹ️ See [here](https://docs.github.com/en/code-security/code-scanning/creating-an-advanced-setup-for-code-scanning/customizing-your-advanced-setup-for-code-scanning) for more details.
> ℹ️ There are [3 build modes](https://docs.github.com/en/code-security/code-scanning/creating-an-advanced-setup-for-code-scanning/codeql-code-scanning-for-compiled-languages#codeql-build-modes) available: `none`, `autobuild` or `manual`. In the workflow, build mode is set to `manual` but you may wish to change this as needed (see [this](https://docs.github.com/en/code-security/code-scanning/creating-an-advanced-setup-for-code-scanning/codeql-code-scanning-for-compiled-languages#building-c)).

##### Dependency Review 📦

Scans pull requests for dependency changes[^dep], highlighting security vulnerabilities and licensing issues.

[^dep]: Skips if PR is from dependabot.

**File**: [dependency-review.yml](.github/workflows/dependency-review.yml)

**Notes**:

>⚠️ Fails PR for high severity vulnerabilities.
>ℹ️ Adds summary to PRs.
>ℹ️ Uses dependency caching.

##### Semgrep 🛡️

Performs static analysis security testing (SAST) to detect security vulnerabilities and bugs.

**File**: [semgrep.yml](.github/workflows/semgrep.yml)

**Notes**:

> ❗Requires setting up Semgrep account and saving Semgrep token as `SEMGREP_APP_TOKEN` secret for repository.
> ℹ️ For more information, please refer to [online help](https://semgrep.dev/docs/deployment/add-semgrep-to-ci).

#### PRs

##### Label Merge Conflicts 💥

Automatically labels PRs that have merge conflicts with `💥 Conflicts` label.

**File**: [label_merge_conflicts.yml](.github/workflows/label_merge_conflicts.yml)

**Notes**:

> ℹ️ Retries up to 5 times with 5 second wait between retries

##### Pull Request Labeler 🏷️

Automatically applies labels to pull requests based on the files changed.

**File**: [labeler.yml](.github/workflows/labeler.yml)

**Notes**:

> ℹ️ Labels are defined in [this config file](.github\labeler.yml).

##### Update PR 🔄

Automatically updates PR branches when the **main** branch changes to prevent stale PRs.

**File**: [update_pr.yml](.github/workflows/update_pr.yml)

**Notes**:

> ❗Requires saving PAT as `ACTION_USER_TOKEN` secret for repository.
> ℹ️ Requires checks to pass
> ℹ️ Allows ongoing checks

#### Tests

##### Tests ✅

Runs tests, generates code coverage reports, and uploads code coverage results to [Codecov](https://www.codecov.io/).

**File**: [test.yml](.github/workflows/test.yml)

**Notes**:

> ❗Requires CodeCov account and token that is stored in a repository secret called `CODECOV_TOKEN`.
> ℹ️ Adds coverage summary to PR and workflow runs.

#### Dependency management

##### Version Sweeper ⬆️

Monthly check for outdated .NET versions and creates automated upgrade PRs.

**File**: [version-sweeper.yml](.github/workflows/version-sweeper.yml)

### Workflow Triggers

| Workflow              | Push | PR | Schedule | Manual |
|-----------------------|------|----|----------|--------|
| CodeQL Analysis       | ✅    | ✅  | -        | ✅      |
| Dependency Review     | ✅    | ✅  | -        | ✅      |
| Label Merge Conflicts | ✅    | -  | -        | -      |
| PR Labeler            | -    | ✅  | -        | -      |
| Semgrep               | ✅    | ✅  | Weekly   | ✅      |
| Tests                 | ✅    | ✅  | -        | -      |
| Update PR             | ✅    | -  | -        | -      |
| Version Sweeper       | -    | -  | Monthly  | ✅      |
