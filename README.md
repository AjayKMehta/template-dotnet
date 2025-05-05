## Overview

This repo is a template for creating .NET GitHub repos. You can do this by clicking the **Use this template** button in the top right corner.

## VS Code

This repo contains `launch.json` and `tasks.json` used to configure  debugging and various tasks. These files makes use of the [Command Variable](https://marketplace.visualstudio.com/items?itemName=rioj7.command-variable) extension to prompt for build configuration (`Release` or `Debug`) and test projects to launch.  

## CI/CD

There are several GitHub workflows for CI defined. Feel free to modify them as needed. Each workflow runs fairly quickly (in my experience).  You may want to combine some of the workflows, e.g. on pushing to a branch, you may want a single workflow that builds the code, perform SAST, run tests in that order rather than have separate workflows for each task.

### Workflows

#### Security + Linting

##### CodeQL Analysis 🔍

**Description**: Automated code analysis to identify vulnerabilities and coding errors in code.

**File**: [codeql-analysis.yml](.github/workflows/codeql-analysis.yml)

##### Dependency Review 📦

**Description**: Scans pull requests for dependency changes[^dep], highlighting security vulnerabilities and licensing issues.

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

**Description**: Automatically labels PRs that have merge conflicts with `💥 Conflicts` label.

**File**: [label_merge_conflicts.yml](.github/workflows/label_merge_conflicts.yml)

**Notes**:

> ℹ️ Retries up to 5 times with 5 second wait between retries

##### Pull Request Labeler 🏷️

**Description**: Automatically applies labels to pull requests based on the files changed.

**File**: [labeler.yml](.github/workflows/labeler.yml)

**Notes**:

> ℹ️ Labels are defined in [this config file](.github\labeler.yml).

##### Update PR 🔄

Automatically updates PR branches when the main branch changes to prevent stale PRs.

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
| CodeQL                | ✅    | ✅  | -        | ✅      |
| Dependency Review     | ✅    | ✅  | -        | ✅      |
| Label Merge Conflicts | ✅    | -  | -        | -      |
| PR Labeler            | -    | ✅  | -        | -      |
| Semgrep               | ✅    | ✅  | Weekly   | ✅      |
| Tests                 | ✅    | ✅  | -        | -      |
| Update PR             | ✅    | -  | -        | -      |
| Version Sweeper       | -    | -  | Monthly  | ✅      |
